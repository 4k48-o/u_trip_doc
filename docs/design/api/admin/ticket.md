# 票务管理 — `/api/v1/admin/tickets`

### 3.1 预约列表

**GET** `/api/v1/admin/tickets/reservations`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| visitDate | string | 否 | 日期 |
| timeSlot | string | 否 | 时段 |
| productId | string | 否 | SKU ID |
| channel | string | 否 | 渠道 |
| status | int | 否 | 预约状态 |
| keyword | string | 否 | 单号/姓名/证件号 |
| reservationSource | string | 否 | order / annual_card / agent |
| orderId | string | 否 | 关联订单 ID |
| createStart | string | 否 | 创建起始时间 |
| createEnd | string | 否 | 创建截止时间 |
| sortBy | string | 否 | create_desc/create_asc |
| pageNo / pageSize | — | — |

**响应records：** reservationNo, contactName, contactPhone(脱敏), contactIdType, productName, visitDate, timeSlot, quantity, channel, reservationSource, orderId, verifyStatus(0/1/2), verifyStatusText, statusText, createTime。

**GET** `/api/v1/admin/tickets/reservations/{id}` — 详情（含游客列表核销状态）

**PUT** `/api/v1/admin/tickets/reservations/{id}` — 修改 `{ "visitDate": "...", "timeSlot": "..." }`

**POST** `/api/v1/admin/tickets/reservations/{id}/cancel` — 取消

**GET** `/api/v1/admin/tickets/reservations/export` — 导出 Excel

### 3.2 季节/时段配置

**GET** `/api/v1/admin/tickets/seasons?productId=X&status=Y&pageNo=1` — 列表

**GET** `/api/v1/admin/tickets/seasons/{id}` — 详情

**POST** `/api/v1/admin/tickets/seasons`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| configCode | string | 是 | 配置编码 |
| configName | string | 是 | 如"旺季日场" |
| productId | string | 是 | SKU ID |
| sceneType | string | 是 | day / night |
| startDate | date | 是 | 生效日期 |
| endDate | date | 是 | 结束日期 |
| timeSlots | array | 是 | `[{"slot":"08:00-10:00","maxCapacity":500}]`（maxCapacity 对应 DB max_capacity） |
| advanceDays | int | 是 | 最大提前预约天数 |
| releaseRule | string | 是 | auto / manual |
| status | int | 是 | 0/1 |

**PUT** `/api/v1/admin/tickets/seasons/{id}` — 编辑（timeSlots 全量替换子表）

**DELETE** `/api/v1/admin/tickets/seasons/{id}` — 删除

### 3.3 窗口售票

**POST** `/api/v1/admin/tickets/window-sell`

> US-004 窗口售票员专用。支付成功后自动核销（视为已入园）。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| items | array | 是 | 票种列表 |
| items[].skuId | string | 是 | SKU ID |
| items[].visitDate | string | 是 | 游览日期 |
| items[].timeSlot | string | 是 | 时段 |
| items[].quantity | int | 是 | 数量 |
| items[].visitors | array | 是 | 游客信息（长度=quantity） |
| items[].visitors[].name | string | 是 | 真实姓名 |
| items[].visitors[].idType | string | 是 | 证件类型 |
| items[].visitors[].idNo | string | 是 | 证件号（HTTPS传输，后端AES加密存储） |
| items[].visitors[].nationality | string | 否 | 国籍 |
| items[].visitors[].gender | string | 否 | M/F/U |
| contactName | string | 否 | 联系人 |
| contactPhone | string | 否 | 联系人手机 |
| payMethod | string | 是 | wechat/alipay/unionpay/cash/bank_card/qrcode |
| cashAmount | string | 否 | 现金实收（payMethod=cash 必填） |
| insuranceFlag | boolean | 否 | 是否购买保险 |
| cashierId | string | 是 | 收银员 ID |
| sessionId | string | 是 | 班次 ID |

**响应：** `{ "code": 0, "data": { "orderId": "...", "orderNo": "...", "changeAmount": "10.00" } }`（cash 时含找零）

### 3.4 手动核销

**POST** `/api/v1/admin/tickets/verify-manual`

> 后台手动核销。优先级: visitorId > orderId+productId > idType+idNo。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| visitorId | string | 否 | 游客记录 ID（精确核销，优先） |
| orderId | string | 否 | 订单 ID（配合 productId） |
| productId | string | 否 | 票种 SKU ID |
| idType | string | 否 | 证件类型（兜底） |
| idNo | string | 否 | 证件号（HTTPS传输，后端AES加密后查询） |
| quantity | int | 否 | 数量（默认1） |
| gateLocation | string | 否 | 核销口 |
| reason | string | 是 | 手动核销原因 |
| approveBy | string | 是 | 审批人（不能自审） |
| offlineFlag | boolean | 否 | 离线核销标记 |

> verifyMethod 固定为 `manual`，后端自动写入。

**响应示例：**
```json
{
  "code": 0,
  "data": { "verifyResult": true, "visitorName": "张三", "productName": "成人全价票", "orderNo": "MTY06010001", "verifyTime": "2026-06-01 09:15:00" }
}
```

### 3.5 检票记录查询

**GET** `/api/v1/admin/tickets/verify-logs`

| 参数 | 必填 | 说明 |
|------|------|------|
| visitorId | 否 | 游客 ID |
| orderId | 否 | 订单 ID |
| deviceGateId | 否 | 闸机 ID |
| verifyMethod | 否 | qr_code/id_card/passport/face/manual |
| verifyResult | 否 | 1成功/0失败 |
| startTime | 否 | 核销起始 |
| endTime | 否 | 核销截止 |
| pageNo | 否 | — |

**响应records字段：** id, visitorName(脱敏), productName, orderNo, verifyMethod, verifyResultText, failReason, deviceGateId, gateLocation, verifyTime。

### 3.6 库存查看

**GET** `/api/v1/admin/tickets/stock?productId=X&visitDate=Y&timeSlot=Z&channel=W`

**响应records字段：** productId, productName, visitDate, timeSlot, channel, totalStock, usedStock, frozenStock, availableStock。

---

