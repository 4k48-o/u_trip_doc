# OTA 开放 API — `/api/v1/ota/`

> 面向携程、美团等 OTA 平台。使用 AK/SK + MD5 签名 + AES-128-CBC 加密。请求/响应格式遵循各 OTA 自身协议规范。

---

## 1. 鉴权说明

| 项目 | 说明 |
|------|------|
| 认证方式 | AK/SK 签名认证 |
| 签名算法 | `MD5(accountId + serviceName + requestTime + body + version + signKey).toLowerCase()` |
| 加密算法 | AES-128-CBC，PKCS5Padding，16字节 key + 16字节 iv |
| 请求格式 | JSON，body 字段为 AES 加密后的密文 |

**请求信封：**
```json
{
  "header": {
    "accountId": "supplier_account_id",
    "serviceName": "DatePriceModify",
    "requestTime": "2026-05-26 10:00:00",
    "version": "1.0",
    "sign": "8c0e41a6afe929a5c53898d670fe73c2"
  },
  "body": "<AES-encrypted-base64-string>"
}
```

**响应信封：**
```json
{
  "header": {
    "resultCode": "0000",
    "resultMessage": "操作成功"
  }
}
```

---

## 2. 商品同步（我方 → OTA）

### 2.1 同步商品

**POST** `/api/v1/ota/sync/product`

> 将我方商品信息推送至 OTA。对应携程 SyncProduct 接口。

**内部触发：** 商品新增/编辑保存时自动触发。也可管理端手动触发。

**同步内容映射：**

| 我方字段 | 携程字段 | 说明 |
|----------|----------|------|
| ota_product_mapping.ota_option_id | supplierProductId | 供应商商品 ID |
| product_language (title) | title | 多语言名称 |
| product_spu.category_code | category.code | 映射到 OTA 类目 |
| product_spu.main_image | gallery[0] | 主图（先上传至 OTA 获取 tripImageId） |
| product_spu.description | description | 商品描述 |
| product_spu.highlight | highlight | 亮点 |
| product_spu.inclusions | inclusions | 费用包含 |
| product_spu.exclusions | exclusions | 费用不含 |
| product_spu.how_to_use | howToUse | 使用方法 |
| product_spu.additional_info | additionalInfo | 附加信息 |
| product_spu.delivery_method | ticketInfo.deliveryMethods | DIGITAL / PRINT / VALID_ID |
| product_spu.redemption_type | redemptionInfo.redemptionType | 入园方式 |
| product_spu.booking_cutoff_time | bookingSettings | 预订截止 |
| product_spu.duration_* | duration | 时长 |
| scenic_spot.ota_poi_mapping | poi | POI 映射 |
| product_sku (price_type) | ticketType.code | Adult / Child / Senior |
| product_sku.age_limit_* | ticketType.restrictions | 年龄限制 |
| rule_refund.cancel_fee_json | cancellationPolicy.rateList | 退改费率 |
| product_option_slot | option / Time_Slot | 选项和时段 |

**请求示例（解密后 body）：**
```json
{
  "supplierProductId": "SPU001",
  "contractId": 12345,
  "primaryLanguage": "zh-CN",
  "status": "active",
  "category": [{ "code": "ATTRACTION_TICKET" }],
  "title": "Mùtiányù Great Wall Adult Ticket",
  "poi": [{
    "supplierPOI": {
      "supplierId": "SPOT_001",
      "mappingElements": {
        "name": "Mutianyu Great Wall",
        "addressDetail": "Mutianyu Village, Huairou District, Beijing",
        "latitude": 40.4319,
        "longitude": 116.5617
      }
    }
  }],
  "ticketInfo": { "deliveryMethods": "DIGITAL" },
  "redemptionInfo": { "redemptionType": "Direct_Entry" },
  "gallery": [{ "tripImageId": "10001" }],
  "description": "Mutianyu Great Wall is located in Huairou District...",
  "cancellationPolicy": {
    "type": "By_Visit_Date",
    "rateList": [
      { "dayBeforeVisitDate": 1, "time": "20:00", "unit": "PERCENTAGE", "value": 0 },
      { "dayBeforeVisitDate": 1, "time": "00:00", "unit": "PERCENTAGE", "value": 50 }
    ],
    "confirmationTime": 24
  },
  "ticketType": [
    { "code": "Adult", "restrictions": { "minAge": 19, "maxAge": 59 } },
    { "code": "Senior", "restrictions": { "minAge": 60 } }
  ],
  "bookingSettings": {
    "bookingType": { "dateType": "DATE_REQUIRED", "dateLimit": { "dateLimitType": "Single_date" } },
    "paymentConfirmationTime": 30
  },
  "option": [{ "optionCode": "Time_Slot" }]
}
```

---

### 2.2 同步套餐

**POST** `/api/v1/ota/sync/package`

> 将组合产品推送至 OTA。对应携程 SyncPackage 接口。

**映射：** product_combo → supplierProductId，product_combo_item → optionList → units。

---

### 2.3 上传图片

**POST** `/api/v1/ota/sync/image`

> 将商品图片上传至 OTA。对应携程 UploadImage 接口。

| 字段 | 必填 | 说明 |
|------|------|------|
| imageUrl | 是 | 我方 OSS 图片 URL |

**响应：** `{ "tripImageId": "10001" }`

> 图片首次上传后在 `ota_product_mapping` 中记录 tripImageId，后续同步复用。**商品主图/图片变更时，系统自动标记 Redis `ota:image:dirty:{spuId}:{otaCode}`，定时任务扫描后触发重新上传并更新 tripImageId。**

---

## 3. 价格库存同步（我方 → OTA）

### 3.1 批量推送价格

**POST** `/api/v1/ota/sync/price`

> 对应携程 DatePriceModify。定时任务每 5 分钟扫描有变更的价格日历并推送。

**请求示例（解密后 body）：**
```json
{
  "sequenceId": "2026-05-26-abc123-def456-...",
  "otaOptionId": 568898,
  "supplierOptionId": "SKU001",
  "dateType": "DATE_REQUIRED",
  "prices": [
    { "date": "2026-06-01", "salePrice": 40.00, "costPrice": 30.00 },
    { "date": "2026-06-02", "salePrice": 40.00, "costPrice": 30.00 }
  ]
}
```

**内部触发逻辑：**
```
product_price_calendar 写入
    → Redis SET "ota:price_dirty:{skuId}:{otaCode}" 
    → 定时任务每分钟扫 dirty flag
    → 查询 product_price_calendar 全量数据
    → 组装携程格式 → 发起请求 → ota_sync_log 记录
```

### 3.2 批量推送库存

**POST** `/api/v1/ota/sync/inventory`

> 对应携程 DateInventoryModify。

**请求示例（解密后 body）：**
```json
{
  "sequenceId": "2026-05-26-ghi789-...",
  "otaOptionId": 568898,
  "supplierOptionId": "SKU001",
  "dateType": "DATE_REQUIRED",
  "inventorys": [
    { "date": "2026-06-01", "quantity": 800 },
    { "date": "2026-06-02", "quantity": 750 }
  ]
}
```

> 库存 = product_price_calendar.stock - ticket_stock.used_stock。

---

## 4. 订单处理（OTA → 我方）

### 4.1 下单回调

**POST** `/api/v1/ota/callback/order`

> OTA 侧用户下单后，携程回调我方创建订单。

**内部处理流程：**
```
解析携程订单请求
    → 校验签名 + 解密
    → ota_product_mapping 查找 SKU（supplierOptionId → skuId）
    → 库存校验 + 规则校验
    → 创建 order_main（channel=ota, distribution_channel=ctrip）
    → 创建 order_sub + ticket_visitor
    → 库存扣减
    → 组装携程格式响应（我方订单号 + 确认类型）
    → 记录 ota_sync_log
```

**关键处理：**
- `confirmType`: 1=即时确认（库存充足+规则通过），2=手动确认（需人工审核的订单）
- `supplierConfirmType`: 1=即时确认（instant），2=手动确认（manual），在商品同步时上报
- 验证 `idempotentKey`（sequenceId 去重）

---

### 4.2 取消回调

**POST** `/api/v1/ota/callback/cancel`

> OTA 侧退票/取消订单时回调。

**内部处理流程：**
```
解析携程取消请求
    → 查找订单（channelOrderNo）
    → 校验订单状态（仅已支付/待支付可取消）
    → 根据 rule_refund 计算退款金额
    → 创建 order_refund
    → 回退库存
    → 回退 OTA 渠道库存
    → 组装携程格式响应（退款金额 + 状态）
    → 触发 MQ: order.refunded
```

---

### 4.3 预下单

**POST** `/api/v1/ota/callback/pre-order`

> 携程 CreatePreOrder。锁定库存 8 秒，等待最终确认或超时释放。

**内部处理流程：**
```
Redis SET "ota:pre_order:{sequenceId}" value=预订详情 EX=10
    → 库存暂时冻结（不是真扣减，是 frozen_stock += N）
    → 8秒内收到 PayPreOrder 则转为正式订单
    → 超时自动释放（frozen_stock -= N，删除 Redis key）
```

---

### 4.4 核销通知回传

**POST** `/api/v1/ota/callback/verify-notice`

> 我方检票核销后，回传 OTA 通知核销。

**内部触发：** `MQ: ticket.verified` → integration-service 消费 → 查找 ota_product_mapping 确认是否为 OTA 订单 → 定时批量回传。

---

### 4.5 OTA 下发凭证

**POST** `/api/v1/ota/callback/voucher`

> 携程下发电子票凭证至我方系统。

**内部处理：** 写入 `ota_voucher` 表 → 更新 `ticket_visitor` 关联凭证 → 通知游客。

---

## 5. 查询（OTA → 我方）

### 5.1 查询订单

**POST** `/api/v1/ota/query/order`

> 携程查询订单状态和信息。

**响应（解密后）：** 订单号、状态（待支付/已支付/已核销/已退款/已取消）、游客信息、金额。

### 5.2 查询余额

**POST** `/api/v1/ota/query/balance`

> 查询供应商账户余额（部分 OTA 支持对公账户充值模式）。

---

## 6. 内部管理接口

### 6.1 手动触发同步

**POST** `/api/v1/ota/internal/manual-sync`

| 字段 | 必填 | 说明 |
|------|------|------|
| mappingId | 是 | OTA 映射 ID |
| syncType | 是 | product / price / inventory |

### 6.2 OTA 通道健康检查

**GET** `/api/v1/ota/internal/health/{otaCode}`

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "otaCode": "ctrip",
    "status": "UP",
    "lastSyncTime": "2026-05-26 09:55:00",
    "pendingCount": 12,
    "errorRate24h": "0.3%",
    "avgResponseTime24h": 245
  }
}
```

---

## 7. OTA 入驻流程

```
1. 商务签约 → 获取 accountId + signKey + AES Key + AES IV
2. 配置 OTA 商品映射 (ota_product_mapping)
3. 配置 OTA POI 映射 (ota_poi_mapping)
4. 上传商品主图 → 获取 tripImageId
5. 同步商品 (SyncProduct)
6. 同步套餐 (SyncPackage) — 如有
7. 配置定时推送 (价格/库存)

--- 沙箱验收 ---
8. 沙箱环境商品同步验证 → 商品在 OTA 沙箱正确展示
9. 沙箱环境价格同步验证 → 价格库存数据一致
10. 沙箱环境下单验证 → OTA 下单 → 我方正确创建订单
11. 沙箱环境核销验证 → 我方核销 → OTA 收到通知
12. 沙箱退款验证 → OTA 退票 → 我方正确处理退款
--- 验收通过，配置切换正式环境 ---
13. 正式上线
```

## 8. OTA 回调重试策略

> OTA 回调（下单/取消/核销通知）为服务端间通信，可靠性要求高。

| 阶段 | 策略 |
|------|------|
| 首次回调 | OTA 发起 POST → 我方接收处理 |
| 处理失败 | 返回非 200 或超时 30s → OTA 自行重试（携程标准：1min/5min/15min，最多3次） |
| 我方异常 | 写入 `ota_sync_log` (status=0) → 定时任务每分钟扫描失败的同步记录 → 指数退避重试 1min/5min/15min → 3次失败后标记 `status=2`，触发人工告警 |
| 死信处理 | 3次重试后仍失败 → `alert_rule(alert_type=ota_sync_fail)` → 通知运营 + 记录 `ota_reconciliation` 对账差异 |
| 幂等保护 | 通过 `sequenceId` 去重，重复回调返回已有结果（order_main.channel_order_no 唯一索引兜底） |

---

## 8. 错误码

| OTA 错误码 | 我方处理 |
|-----------|----------|
| 0000 | 正常 -> 更新 ota_sync_log.status=1 |
| 0001 | 参数错误 -> 检查请求格式，记录告警 |
| 0002 | 认证失败 -> 检查 AK/SK 配置 |
| 0003 | 请求过于频繁 -> 降低推送频率 |
| 1001 | PLU 不存在/错误 -> 检查 ota_product_mapping |
| 2001 | 商品不存在/错误 -> 触发 product 重新同步 |
| 500 | 服务端错误 -> 记录重试，最多 3 次 |

> 详细错误码参见各 OTA 平台开发文档。
