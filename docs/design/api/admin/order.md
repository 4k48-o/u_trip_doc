# 订单管理 — `/api/v1/admin/orders`

### 4.1 全量订单检索

**GET** `/api/v1/admin/orders`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| orderNo | string | 否 | 订单号 |
| orderType | string | 否 | ticket/combo/rental/hotel/catering |
| contactName | string | 否 | 联系人 |
| contactPhone | string | 否 | 联系人手机 |
| idNo | string | 否 | 游客证件号 |
| travelDate | string | 否 | 出行日期 |
| status | int | 否 | 1待支付/2已支付/3已核销/4已退款/5已取消/6部分退款/7已关闭 |
| channel | string | 否 | 渠道 |
| distributionChannel | string | 否 | OTA 分销渠道 |
| userId | string | 否 | 用户 ID |
| merchantId | string | 否 | 商户 ID |
| agentId | string | 否 | 旅行社 ID |
| payStart | string | 否 | 支付开始时间 |
| payEnd | string | 否 | 支付结束时间 |
| insuranceFlag | int | 否 | 0/1 |
| createStart | string | 否 | 下单开始 |
| createEnd | string | 否 | 下单截止 |
| sortBy | string | 否 | amount_desc/amount_asc/create_desc/create_asc |
| pageNo / pageSize | — | — |

**响应records字段：** orderId, orderNo, orderType, orderTypeText, channelOrderNo, totalAmount, discountAmount, paidAmount, depositAmount, status, statusText, contactName, contactPhone(脱敏), travelDate, channel, distributionChannel, insuranceFlag, refundStatus, refundable, payTime, createTime。

### 4.2 订单详情

**GET** `/api/v1/admin/orders/{orderId}`

> 含主订单 + 子订单 + 游客信息 + 支付流水 + 退款记录 + 操作时间线。

**响应关键字段（新增 marked *）：**
```json
{
  "data": {
    "orderId": "...", "orderNo": "...", "orderType": "ticket",
    "channel": "self_miniapp", "distributionChannel": null,
    "totalAmount": "75.00", "discountAmount": "5.00", "paidAmount": "70.00",
    "depositAmount": null, "insuranceFlag": 0,
    "insuranceAmount": null, "insuranceProductId": null,
    "agentId": null, "agentName": null,    // *
    "expireTime": "2026-05-26 10:30:00",   // *
    "status": 2, "contactName": "张三", "contactPhone": "138****1234", "travelDate": "...",
    "subOrders": [{
      "subOrderId": "SUB001", "subOrderNo": "...",
      "productId": "SKU001", "productName": "成人全价票",
      "productType": "ticket",              // *
      "merchantId": null, "merchantName": "自营",
      "quantity": 1, "unitPrice": "40.00", "subAmount": "40.00",
      "verifyCount": 0, "status": 2, "refundStatus": null,  // * refundStatus
      "visitors": [{
        "visitorId": "VIS001", "name": "张三",
        "idType": "ID_CARD", "idNo": "110***********1234",
        "verifyStatus": 0, "verifyTime": null, "verifyGate": null,
        "verifyMethod": null,               // * qr_code/id_card/face/manual
        "voucher": { "voucherCode": "...", "voucherType": "barcode_128" }
      }]
    }],
    "payment": {
      "paymentNo": "PAY06010001", "payMethod": "wechat", "payAmount": "70.00",
      "transactionId": "4200001234567890", "payTime": "2026-05-26 10:15:00"
    },
    "refunds": [],
    "timeline": [...]
  }
}
```

### 4.3 订单状态管理

**POST** `/api/v1/admin/orders/{orderId}/cancel` — 手动取消 `{ "reason": "游客要求" }`

**POST** `/api/v1/admin/orders/{orderId}/close` — 手动关闭 `{ "reason": "纠纷订单" }`

### 4.4 子订单退款

**POST** `/api/v1/admin/orders/{orderId}/sub/{subOrderId}/refund`

> 指定子订单独立退款（组合订单场景）。

| 字段 | 必填 | 说明 |
|------|------|------|
| refundAmount | 否 | 退款金额（≤ 子单金额，默认全额） |
| reason | 是 | 退款原因 |

### 4.5 退款审批

**GET** `/api/v1/admin/orders/refunds?status=X&startDate=Y&endDate=Z&refundType=Z` — 退款列表

**GET** `/api/v1/admin/orders/refunds/{refundId}` — 退款详情（含申请信息/审核历史/退款流水）

**POST** `/api/v1/admin/orders/refunds/{refundId}/approve` — 审批通过

**POST** `/api/v1/admin/orders/refunds/{refundId}/reject` — 驳回 `{ "reason": "退票时限已过" }`

### 4.6 强制退款（超管）

**POST** `/api/v1/admin/orders/{orderId}/refund-force`

> 超管权限。不受退票规则限制。`refundMethod` 固定为 manual。

| 字段 | 必填 | 说明 |
|------|------|------|
| subOrderIds | 否 | 指定子订单（空=全部） |
| refundType | 是 | full / partial / force / deposit_return |
| refundAmount | 否 | 手动指定（≤ paid_amount，为空则全额） |
| approveBy | 是 | 审批人 |
| reason | 是 | 原因 |

### 4.7 批量核销

**POST** `/api/v1/admin/orders/batch-verify`

| 字段 | 必填 | 说明 |
|------|------|------|
| orderId | 是 | 主订单 ID |
| visitorIds | 是 | 游客 ID 列表 |
| gateLocation | 否 | 核销闸机位置 |

**响应：** `{ "code": 0, "data": { "successCount": 3, "failCount": 2, "failDetails": [{"visitorId":"VIS003","reason":"已核销"}] } }`（逐条处理，部分成功）

### 4.8 单游客全域订单聚合

**GET** `/api/v1/admin/orders/visitor/{visitorId}`

| 参数 | 必填 | 说明 |
|------|------|------|
| startDate | 否 | 开始日期 |
| endDate | 否 | 结束日期 |
| orderType | 否 | 订单类型筛选 |

### 4.9 订单导出

**GET** `/api/v1/admin/orders/export`

> 筛选参数同 4.1，导出 Excel。

### 4.10 支付对账视图

**GET** `/api/v1/admin/orders/payment-reconciliation`

| 参数 | 必填 | 说明 |
|------|------|------|
| payMethod | 否 | wechat/alipay/unionpay |
| startDate | 是 | 支付起始日期 |
| endDate | 是 | 支付截止日期 |

**响应：** `{ "records": [{ "payMethod": "wechat", "totalCount": 1280, "totalAmount": "89600.00", "totalRefund": "4500.00", "netAmount": "85100.00" }] }`

---

