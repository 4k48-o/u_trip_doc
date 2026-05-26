# 公共 API

> 鉴权、健康检查、文件上传等全局共享接口。

---

## 1. 认证接口

### 1.1 登录

**POST** `/api/v1/auth/login`

> 游客/管理员/商户/旅行社 统一登录入口，根据用户角色返回对应权限的 Token。

**请求参数（Body）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 用户名/手机号/邮箱 |
| password | string | 是 | 明文密码（HTTPS 传输） |
| captchaKey | string | 否 | 图形验证码 key（连续失败 3 次后必填） |
| captchaCode | string | 否 | 图形验证码值 |
| loginType | string | 是 | 登录类型：user（游客端）/ admin（管理端）/ merchant（商户端）/ agent（旅行社端） |

**请求示例：**
```json
{
  "username": "admin",
  "password": "******",
  "loginType": "admin"
}
```

**响应参数（data）：**

| 字段 | 类型 | 说明 |
|------|------|------|
| accessToken | string | JWT Access Token |
| refreshToken | string | JWT Refresh Token |
| expiresIn | int | Access Token 有效期（秒） |
| userInfo | object | 用户基本信息 |

**响应示例：**
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiJ9...",
    "expiresIn": 7200,
    "userInfo": {
      "userId": "U001",
      "username": "admin",
      "realname": "管理员",
      "avatar": "https://cdn.example.com/avatar/u001.jpg",
      "roles": ["ADMIN"],
      "permissions": ["admin:product:list", "admin:order:list"]
    }
  }
}
```

**错误码：**

| code | 说明 |
|------|------|
| AUTH_001 | 未登录或 Token 已过期 |
| AUTH_002 | 用户名或密码错误 |
| AUTH_006 | 账号已被禁用 |
| AUTH_005 | 操作过于频繁（需输入验证码） |

---

### 1.2 刷新 Token

**POST** `/api/v1/auth/refresh`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| refreshToken | string | 是 | 登录时获取的 Refresh Token |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
    "expiresIn": 7200
  }
}
```

---

### 1.3 退出登录

**POST** `/api/v1/auth/logout`

**Auth:** Bearer Token

**响应示例：**
```json
{ "code": 0, "msg": "success" }
```

> 服务端清除 Token（Redis 删除），客户端清除本地存储的 Token。

---

### 1.4 发送验证码

**POST** `/api/v1/auth/send-code`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| target | string | 是 | 手机号或邮箱 |
| scene | string | 是 | login / register / reset-password / bind |
| captchaKey | string | 否 | 图形验证码 key（连续发送失败必填） |
| captchaCode | string | 否 | 图形验证码值 |

**请求示例：**
```json
{
  "target": "<AES加密手机号>",
  "scene": "register"
}
```

**响应示例：**
```json
{
  "code": 0,
  "msg": "验证码已发送",
  "data": { "expireIn": 300 }
}
```

**限流：** 同目标每分钟 1 次，每小时 5 次。

---

### 1.5 注册（游客）

**POST** `/api/v1/auth/register`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 用户名，4-20 位字母数字 |
| password | string | 是 | 密码，8-32 位 |
| phone | string | 否 | 手机号（AES 加密） |
| email | string | 否 | 邮箱 |
| verifyCode | string | 是 | 短信/邮箱验证码 |

**响应示例：**
```json
{
  "code": 0,
  "msg": "注册成功",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiJ9...",
    "expiresIn": 7200
  }
}
```

**错误码：**

| code | 说明 |
|------|------|
| USR_002 | 用户名已被占用 |
| USR_003 | 邮箱已被注册 |
| USR_004 | 手机号已被注册 |
| VALID_002 | 验证码错误或已过期 |

---

### 1.6 重置密码

**POST** `/api/v1/auth/reset-password`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| target | string | 是 | 手机号或邮箱（已绑定） |
| newPassword | string | 是 | 新密码 |
| verifyCode | string | 是 | 短信/邮箱验证码 |

---

### 1.7 修改密码

**PUT** `/api/v1/auth/change-password`

**Auth:** Bearer Token

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| oldPassword | string | 是 | 旧密码 |
| newPassword | string | 是 | 新密码 |

---

### 1.8 获取当前用户信息

**GET** `/api/v1/auth/current-user`

**Auth:** Bearer Token

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "userId": "U001",
    "username": "admin",
    "realname": "管理员",
    "avatar": "https://cdn.example.com/avatar/u001.jpg",
    "phone": "138****1234",
    "email": "ad***@example.com",
    "roles": ["ADMIN"],
    "permissions": ["admin:*"],
    "orgCode": "MTY",
    "departName": "运营管理部"
  }
}
```

---

## 2. 公共服务

### 2.1 健康检查

**GET** `/api/v1/health`

> 无需鉴权。用于负载均衡器存活探测。

**响应示例：**
```json
{ "code": 0, "msg": "OK", "data": { "status": "UP" } }
```

### 2.2 就绪检查

**GET** `/api/v1/health/ready`

> 无需鉴权。检查数据库、Redis 连接状态。

**响应示例：**
```json
{
  "code": 0,
  "msg": "OK",
  "data": {
    "status": "UP",
    "checks": {
      "db": "UP",
      "redis": "UP",
      "mq": "UP",
      "oss": "UP"
    }
  }
}
```

---

### 2.3 获取字典项

**GET** `/api/v1/common/dict/{dictCode}`

> 无需鉴权。按字典编码获取字典项列表，用于前端下拉框、单选框等。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| dictCode | string | 是 | 字典编码（路径参数） |

**响应示例：**
```json
{
  "code": 0,
  "data": [
    { "itemText": "全价", "itemValue": "FULL", "itemColor": "" },
    { "itemText": "优惠", "itemValue": "DISCOUNT", "itemColor": "" },
    { "itemText": "免费", "itemValue": "FREE", "itemColor": "green" }
  ]
}
```

### 2.3a 获取全部字典

**GET** `/api/v1/common/dicts`

> 无需鉴权。返回全部字典编码→名称 Map，用于管理端表单初始化。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "price_type": [{ "itemText": "全价", "itemValue": "FULL" }, { "itemText": "优惠", "itemValue": "DISCOUNT" }],
    "category_code": [{ "itemText": "门票", "itemValue": "TICKET" }, { "itemText": "联票", "itemValue": "COMBO" }],
    "channel": [{ "itemText": "小程序", "itemValue": "self_miniapp" }, { "itemText": "PC", "itemValue": "self_pc" }]
  }
}
```

---

### 2.4 获取上传凭证

**POST** `/api/v1/common/upload-token`

**Auth:** Bearer Token

> 获取 OSS 直传凭证。前端拿到凭证后直接上传文件至 OSS，不经过服务端中转。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "accessKeyId": "STS.xxx",
    "accessKeySecret": "xxx",
    "securityToken": "xxx",
    "region": "oss-cn-beijing",
    "bucket": "project-uploads",
    "endpoint": "https://oss-cn-beijing.aliyuncs.com",
    "dir": "images/2026/05/26/",
    "expireTime": 1716714000
  }
}
```

---

### 2.5 文件上传回调确认

**POST** `/api/v1/common/upload-callback`

**Auth:** Bearer Token

> OSS 上传完成后，前端调用此接口确认上传并记录到 `oss_file` 表。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| fileName | string | 是 | 原始文件名 |
| fileUrl | string | 是 | OSS 文件 URL |
| fileSize | long | 是 | 文件大小（字节） |
| fileType | string | 是 | 文件类型，如 jpg / png / pdf |

**响应示例：**
```json
{
  "code": 0,
  "data": { "fileId": "F20260526001", "fileUrl": "https://cdn.example.com/images/2026/05/26/xxx.jpg" }
}
```

---

### 2.6 文件下载（带签名）

**GET** `/api/v1/common/file/{fileId}`

**Auth:** Bearer Token

> 返回带签名的临时下载 URL（有效期 10 分钟）。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "downloadUrl": "https://cdn.example.com/images/2026/05/26/xxx.jpg?sign=xxx&expires=xxx",
    "fileName": "门票截图.jpg",
    "fileSize": 204800,
    "expireIn": 600
  }
}
```

---

### 2.7 生成图形验证码

**GET** `/api/v1/common/captcha`

> 无需鉴权。返回验证码图片和 key。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "captchaKey": "uuid-captcha-key",
    "captchaImage": "data:image/png;base64,iVBORw0KGgo..."
  }
}
```

---

### 2.8 消息通知轮询

**GET** `/api/v1/common/notifications`

> **优先使用 WebSocket 实时推送，轮询为降级方案。** WebSocket 鉴权与心跳见 [接口规范](conventions.md#9-websocket-规范)。

**Auth:** Bearer Token

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 10 |
| readFlag | int | 否 | 0=未读，1=已读 |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "records": [{
      "id": "N001",
      "title": "支付成功",
      "content": "您的订单 MTY20260601001 已支付成功",
      "readFlag": 0,
      "createTime": "2026-05-26 10:00:00"
    }],
    "total": 5,
    "unreadCount": 2
  }
}
```

**PUT** `/api/v1/common/notifications/{id}/read` — 标记已读

**PUT** `/api/v1/common/notifications/read-all` — 全部标记已读
