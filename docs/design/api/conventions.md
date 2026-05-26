# API 接口规范

## 1. 通用约定

| 项目 | 约定 |
|------|------|
| 协议 | HTTPS（全站强制） |
| 格式 | JSON（Content-Type: application/json） |
| 编码 | UTF-8（URL 参数使用 encodeURIComponent） |
| 版本 | URL 路径版本 `/api/v1/` |
| 日期格式 | `yyyy-MM-dd`（纯日期，如 visit_date / settle_date / 业务日期字段）；`yyyy-MM-dd HH:mm:ss`（日期时间，如 create_time / pay_time / 业务时间戳字段） |
| 金额字段 | 字符串 `"120.00"`，避免浮点精度问题 |
| 布尔字段 | API 响应统一使用 JSON `true/false`（非 0/1）。数据库层存储 tinyint(1) 由 ORM/序列化层自动转换 |
| 时间戳 | 毫秒级 Unix 时间戳（long 类型），仅用于 `timestamp`（响应信封）和 WebSocket 消息时间 |

## 2. 鉴权方式

| API 组 | 路由前缀 | 鉴权方式 |
|--------|----------|----------|
| 游客端 | `/api/v1/tourist/` | 下单/个人中心需 JWT Token，查询类可选 |
| 管理端 | `/api/v1/admin/` | JWT Token + RBAC 角色权限（Shiro） |
| 商户端 | `/api/v1/merchant/` | JWT Token + 商户数据隔离 |
| 旅行社端 | `/api/v1/agent/` | JWT Token + 旅行社分组权限 |
| OTA 开放 | `/api/v1/ota/` | AK/SK 签名认证（MD5 + AES-128-CBC） |

**Token 传递：**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

**Token 刷新：**
- Access Token 有效期 2 小时
- Refresh Token 有效期 7 天
- 过期前 5 分钟客户端主动刷新

## 3. 请求头

| Header | 必填 | 说明 |
|--------|------|------|
| Content-Type | 是 | application/json |
| Authorization | 按需 | Bearer {token} |
| X-Request-Id | 否 | 客户端生成 UUID，用于全链路追踪 |
| Accept-Language | 否 | zh-CN / en / ja 等，影响错误消息语言 |
| X-Client-Type | 否 | miniapp / h5 / pc / window，用于埋点统计 |
| Accept-Language | 否 | 优先级: `?lang=` 查询参数 > `Accept-Language` 头 > 默认 `zh-CN`。用于 `statusText`、错误消息、展示文本等字段的国际化。支持: `zh-CN`、`en`、`ja`、`ko`、`ru`、`fr`、`es`、`ar`。不支持的语言 fallback 至 `zh-CN` |

## 4. 统一响应格式

```json
{
  "code": 0,
  "msg": "success",
  "data": {},
  "timestamp": 1716710400000,
  "requestId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| code | int | 0 = 成功，非 0 = 失败（详见错误码文档） |
| msg | string | 提示信息（生产环境不暴露系统错误细节） |
| data | object/array/null | 响应数据，分页时为 `{ records, total, pageNo, pageSize }` |
| timestamp | long | 服务端响应时间戳（毫秒） |
| requestId | string | 请求追踪 ID，与 X-Request-Id 一致或服务端生成 |

### 4.1 分页响应

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "records": [],
    "total": 150,
    "pageNo": 1,
    "pageSize": 20
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| records | array | 当前页数据 |
| total | int | 总记录数 |
| pageNo | int | 当前页码（从 1 开始） |
| pageSize | int | 每页条数（默认 20，最大 100） |

### 4.2 错误响应

```json
{
  "code": 1002,
  "msg": "用户名或密码错误",
  "data": {
    "errorCode": "AUTH_002",
    "fieldErrors": [
      { "field": "password", "message": "密码错误" }
    ]
  },
  "timestamp": 1716710400000,
  "requestId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

## 5. 敏感信息处理

> **传输加密说明：** "传输"列中的 AES-128-CBC 仅指 **OTA 开放 API** 的请求体签名加密（AK/SK + AES-128-CBC，密钥/IV 由 OTA 平台分配和管理）。游客端/管理端/商户端/旅行社端 API 的传输加密由 HTTPS（TLS 1.3）保证，不做应用层字段级 AES 加密。

| 数据 | 存储 | 传输 | 日志 | 响应 |
|------|------|------|------|------|
| 密码 | bcrypt 哈希 | HTTPS | 不记录 | 不返回 |
| 身份证号 | AES-256 加密 | HTTPS | 不记录 | 脱敏 `110***********1234` |
| 手机号 | AES-256 加密 | HTTPS | 脱敏（中国大陆: `138****1234`；国际: `+8190****5678`） | 脱敏 |
| 护照号 | AES-256 加密 | HTTPS | 不记录 | 脱敏 `E****1234` |
| Token | Redis 存储 | HTTPS Header | 不记录 | 仅登录时返回 |
| 人脸图片 | AES-256 加密 | HTTPS | 不记录 | 不返回原始图片，仅返回缩略图 URL |
| 支付密码 | bcrypt 哈希 | HTTPS | 不记录 | 不返回 |

## 6. 幂等性

| 场景 | 实现方式 |
|------|----------|
| 创建订单 | 客户端生成 `idempotentKey`，服务端 Redis 缓存去重（1 小时） |
| 支付 | 支付平台保证幂等（transaction_id 去重） |
| 退款 | `refund_no` 唯一索引，重复请求返回已有结果 |
| OTA 回调 | `sequenceId` 去重 |
| 租赁取设备 | `device_no` + 请求时间窗口，同一设备 30 秒内重复扫描返回已有租赁单 |
| 租赁归还 | `rental_no` 去重，已归还的租赁单重复请求返回已有结算结果 |

**幂等键传递：**
```
X-Idempotent-Key: uuid-v4
```

## 7. 限流策略

| 接口类型 | 限流规则 |
|----------|----------|
| 登录 | 同 IP 每分钟 10 次，同账号每分钟 5 次 |
| 发送验证码 | 同手机/邮箱每分钟 1 次，每小时 5 次 |
| 通用查询 | 同用户每秒 100 次 |
| 下单 | 同用户每秒 10 次 |
| OTA 回调 | 同 OTA 每秒 100 次 |
| OTA 查询（价格/库存） | 同 OTA 每秒 50 次，超过返回 429 |
| 租赁取/还设备 | 同用户每秒 5 次 |
| 文件上传 | 同 IP 每分钟 20 次 |

超限响应：
```json
{
  "code": 429,
  "msg": "操作过于频繁，请稍后重试",
  "data": { "retryAfter": 60 }
}
```

## 8. 版本管理

| 策略 | 说明 |
|------|------|
| URL 版本 | `/api/v1/`, `/api/v2/` |
| 兼容性 | 新增字段向后兼容，不删除已有字段 |
| 废弃标记 | 废弃接口返回 `Deprecation: true` 响应头 + `sunset` 日期 |
| 废弃周期 | 标记废弃后至少保留 6 个月 |
| 迁移策略 | v1 下线前 3 个月发公告 → 客户端升级至 v2 → v1 返回 `410 Gone` |

## 9. WebSocket 规范

> 用于实时推送场景（消息通知、大屏数据、AI 对话流式输出）。

| 项目 | 约定 |
|------|------|
| 协议 | WSS（WebSocket Secure） |
| 端点 | `wss://api.example.com/ws` |
| 鉴权 | 连接时 URL 参数传递 `?token=<jwt>`，服务端首条消息验证 |
| 心跳 | 客户端每 30 秒发送 `{"type":"ping"}`，服务端回复 `{"type":"pong"}`，120 秒无心跳断开 |
| 消息格式 | `{"type":"event_type","data":{},"timestamp":0}` |
| 自动重连 | 客户端断开后指数退避重连（1s/2s/4s/8s，最大 30s） |

**应用场景：**

| 场景 | event_type | data 内容 |
|------|-----------|-----------|
| 支付状态变更 | `payment_status` | `{"orderId":"...","status":2}` |
| 核销通知 | `verify_result` | `{"visitorId":"...","result":true}` |
| 大屏实时数据 | `dashboard_update` | 同 REST 响应 data |
| AI 对话流式 | `ai_stream` | `{"sessionId":"...","chunk":"一段文字"}` |
| 预警推送 | `alert_triggered` | `{"alertId":"...","alertName":"..."}` |
| 租赁逾期提醒 | `rental_overdue` | `{"rentalNo":"...","deviceName":"...","overdueFee":"..."}` |
| 租赁归还确认 | `rental_return_confirm` | `{"rentalNo":"...","rentFee":"...","depositRefund":"..."}` |
