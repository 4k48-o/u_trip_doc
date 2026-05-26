# 商户端 API — `/api/v1/merchant/`

> 面向入驻商户。需要商户 JWT Token，所有接口按商户数据隔离，商户只能操作自己的店铺、商品和订单。

---

## 1. 店铺管理

### 1.1 我的店铺信息

**GET** `/api/v1/merchant/shop`

> 返回当前商户的店铺基本信息。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "merchantId": "M001",
    "merchantCode": "MERCHANT_001",
    "merchantName": "绥中文创旗舰店",
    "category": "CULTURAL",
    "contactName": "李四",
    "contactPhone": "139****5678",
    "shopId": "SHOP001",
    "shopName": "长城文创馆",
    "shopLogo": "https://cdn.example.com/logo/m001.jpg",
    "shopDesc": "主营长城主题文创产品",
    "status": 1,
    "commissionRate": "0.1000",
    "totalRevenue": "45800.00"
  }
}
```

---

### 1.2 编辑店铺

**PUT** `/api/v1/merchant/shop`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| shopName | string | 否 | 店铺名称 |
| shopLogo | string | 否 | 店铺 Logo URL |
| shopDesc | string | 否 | 店铺描述 |

---

### 1.3 店铺页面装修

**PUT** `/api/v1/merchant/shop/page`

> 提交拖拽式页面编辑器的 JSON 配置。后台管理端预览后发布。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageConfig | longtext | 是 | 页面配置 JSON（含组件布局、样式、数据绑定） |

**请求示例（简略）：**
```json
{
  "pageConfig": "{\"layout\":\"header\",\"components\":[{\"type\":\"banner\",\"images\":[\"...\"]},{\"type\":\"product_list\",\"category\":\"CULTURAL\"}]}"
}
```

---

## 2. 商品管理

### 2.1 我的商品列表

**GET** `/api/v1/merchant/products`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| categoryCode | string | 否 | 业态 |
| keyword | string | 否 | 名称/编码 |
| status | int | 否 | 0下架/1上架/2待审核 |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**响应records字段：** spuId, spuCode, spuName, categoryCode, minPrice, status, statusText, auditStatus（0待审核/1通过/2驳回）, auditRemark, createTime。

---

### 2.2 上架商品

**POST** `/api/v1/merchant/products`

> 商品提交后进入平台审核流程。审核通过后才在游客端可见。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| spuCode | string | 是 | 商品编码 |
| spuName | string | 是 | 商品名称 |
| categoryCode | string | 是 | 业态分类 |
| description | string | 是 | 商品描述 |
| mainImage | string | 是 | 主图 URL |
| images | array | 否 | 图片列表 |
| notice | string | 否 | 购买须知 |
| refundPolicy | string | 否 | 退改规则 |
| deliveryMethod | string | 是 | DIGITAL / PRINT / VALID_ID |
| redemptionType | string | 是 | Direct_Entry / Need_Ticket_Exchange / Meet_at_Start_Point |
| durationValue | int | 否 | 时长 |
| durationUnit | string | 否 | Day / Hour / Minute |
| status | int | 是 | 0=保存草稿不提交审核，1=提交审核 |
| skus | array | 是 | SKU 列表（格式同管理端） |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "spuId": "SPU_M001",
    "spuCode": "CULTURAL_MTY_01",
    "auditStatus": 0,
    "auditStatusText": "待平台审核"
  }
}
```

---

### 2.3 编辑/下架商品

**PUT** `/api/v1/merchant/products/{spuId}` — 编辑（编辑后重新进入待审核状态）

**PUT** `/api/v1/merchant/products/{spuId}/stock` — 调整库存

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| skuId | string | 是 | SKU ID |
| adjustQuantity | int | 是 | +增/-减 |
| reason | string | 是 | 调整原因 |

**POST** `/api/v1/merchant/products/{spuId}/offline` — 下架

### 2.4 商品评价

**GET** `/api/v1/merchant/products/{spuId}/reviews`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**响应records字段：** reviewerName(脱敏), rating(1-5), content, images, replyContent(商户回复), createTime。

**POST** `/api/v1/merchant/products/reviews/{reviewId}/reply` — 商户回复评价 `{ "replyContent": "感谢您的评价" }`

---

## 3. 订单与核销

### 3.1 我的订单列表

**GET** `/api/v1/merchant/orders`

> 只显示商户自身商品的订单。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| orderNo | string | 否 | 订单号精确搜索 |
| status | int | 否 | 2已支付/3已核销/4已退款 |
| travelDate | string | 否 | 出行日期 |
| startDate | string | 否 | 下单开始 |
| endDate | string | 否 | 下单结束 |
| pageNo | int | 否 | — |

**响应records字段：** orderNo, productName(我的商品), quantity, paidAmount, status, statusText, contactPhone(脱敏), travelDate, createTime。

---

### 3.2 订单详情

**GET** `/api/v1/merchant/orders/{orderId}` — 只显示本商户的子订单 + 游客信息

---

### 3.3 扫码核销

**POST** `/api/v1/merchant/orders/verify`

> 商户使用 H5 或小程序核销终端，扫描游客出示的二维码/券码完成核销。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| voucherCode | string | 是 | 扫码获得的凭证编码/二维码数据 |
| deviceInfo | string | 否 | 核销终端信息（商户端自动采集） |

**响应示例（成功）：**
```json
{
  "code": 0,
  "data": {
    "verifyResult": true,
    "orderNo": "MTY06010001",
    "productName": "绥中长城成人票",
    "visitorName": "张*",
    "quantity": 1,
    "verifyTime": "2026-06-01 09:15:00"
  }
}
```

**错误码：**

| code | 说明 |
|------|------|
| BIZ_003 | 已核销，请勿重复操作 |
| BIZ_001 | 无效凭证 |
| VERIFY_001 | 非本商户商品，无权核销 |

---

### 3.4 核销记录

**GET** `/api/v1/merchant/orders/verify-log`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| startDate | string | 否 | 核销开始日期 |
| endDate | string | 否 | 核销结束日期 |

**响应records字段：** orderNo, productName, visitorName(脱敏), verifyTime, verifyMethod。

---

## 4. 结算

### 4.1 结算明细

**GET** `/api/v1/merchant/settlements`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| settleDate | string | 否 | 结算日期 |
| settleStatus | int | 否 | 0待结算/1已结算/2已打款 |
| pageNo | int | 否 | — |

**响应records字段：** settlementNo, orderNo, productName, totalAmount, commissionRate, commissionAmount, settleAmount, settleDate, settleStatusText。

---

### 4.2 对账单

**GET** `/api/v1/merchant/settlements/ledgers`

> 按账期查看对账单。支持确认。

**响应records字段：** ledgerNo, periodStart, periodEnd, totalOrders, totalAmount, totalCommission, totalSettle, status, statusText。

**PUT** `/api/v1/merchant/settlements/ledgers/{ledgerId}/confirm` — 确认对账单

---

### 4.3 销售看板

**GET** `/api/v1/merchant/settlements/stats`

> 商户首页经营数据概览。支持自定义日期范围。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| startDate | string | 否 | 开始日期，默认本月1日 |
| endDate | string | 否 | 结束日期，默认今天 |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "today": { "orderCount": 45, "revenue": "3600.00" },
    "yesterday": { "orderCount": 52, "revenue": "4200.00" },
    "thisMonth": { "orderCount": 1240, "revenue": "98500.00" },
    "settlementPending": "12500.00",
    "trend": [
      { "date": "2026-05-20", "orders": 48, "revenue": "3840.00" },
      { "date": "2026-05-21", "orders": 52, "revenue": "4200.00" }
    ]
  }
}
```
