# 管理端 API — `/api/v1/admin/`

> 面向景区运营管理者、财务人员、超级管理员。全部需要 JWT Token + RBAC 角色权限。

---

## 1. 景区与景点管理

### 1.1 景区列表/详情

**GET** `/api/v1/admin/scenic`

| 参数 | 必填 | 说明 |
|------|------|------|
| keyword | 否 | 名称模糊搜索 |
| status | 否 | 0关闭/1开放 |
| pageNo | 否 | — |
| pageSize | 否 | — |

**POST** `/api/v1/admin/scenic` — 新增景区

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| areaCode | string | 是 | 景区编码 |
| areaName | string | 是 | 景区名称 |
| areaNameEn | string | 否 | 英文名称（OTA 使用） |
| address | string | 否 | 地址 |
| longitude | decimal | 是 | 经度 |
| latitude | decimal | 是 | 纬度 |
| contactPhone | string | 否 | 联系电话 |
| description | string | 否 | 景区简介 |
| openingHours | string | 否 | `[{"season":"spring","time":"08:00-17:00"}]` |
| logo | string | 否 | Logo URL |
| status | int | 是 | 0/1 |

**PUT** `/api/v1/admin/scenic/{areaId}` — 编辑景区

**DELETE** `/api/v1/admin/scenic/{areaId}` — 逻辑删除

### 1.2 景点管理

**GET** `/api/v1/admin/scenic/{areaId}/spots` — 景区下景点列表

**POST** `/api/v1/admin/scenic/{areaId}/spots` — 新增景点

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| spotCode | string | 是 | 景点编码 |
| spotName | string | 是 | 景点名称 |
| spotNameEn | string | 否 | 英文名称 |
| spotType | string | 是 | scenic / cultural / facility / entrance |
| longitude | decimal | 是 | 经度 |
| latitude | decimal | 是 | 纬度 |
| googlePlaceId | string | 否 | Google Place ID（OTA 映射用） |
| description | string | 否 | 介绍 |
| sortNo | int | 否 | 排序 |
| status | int | 是 | 0/1 |

**PUT** `/api/v1/admin/scenic/{areaId}/spots/{spotId}` — 编辑

**DELETE** `/api/v1/admin/scenic/{areaId}/spots/{spotId}` — 逻辑删除

---

## 2. 商品管理

### 2.1 商品列表

**GET** `/api/v1/admin/products`

| 参数 | 必填 | 说明 |
|------|------|------|
| categoryCode | 否 | 业态筛选 |
| keyword | 否 | SPU 名称/编码 |
| status | 否 | 0下架/1上架 |
| pageNo / pageSize | 否 | — |

**响应records关键字段：** spuId, spuCode, spuName, categoryCode, categoryName, minPrice, maxPrice, status, statusText, createTime, skuCount。

### 2.2 新增商品

**POST** `/api/v1/admin/products`

> 一次性创建 SPU + SKU 嵌套结构。

**请求示例：**
```json
{
  "spuCode": "TICKET_ADULT",
  "spuName": "绥中长城成人票",
  "categoryCode": "TICKET",
  "scenicSpotId": "SPOT_001",
  "description": "绥中长城位于北京市怀柔区...",
  "mainImage": "https://cdn.example.com/images/p001.jpg",
  "images": ["url1", "url2"],
  "notice": "请提前预约时段...",
  "refundPolicy": "游览前1天20:00前免费取消...",
  "inclusions": ["成人门票1张"],
  "exclusions": ["缆车费用"],
  "highlight": ["世界文化遗产", "5A级景区"],
  "howToUse": ["凭二维码扫码入园"],
  "additionalInfo": "60岁以上老人免票",
  "deliveryMethod": "DIGITAL",
  "redemptionType": "Direct_Entry",
  "bookingCutoffTime": "{\"dayBeforeVisitDate\":0,\"time\":\"08:00\"}",
  "durationValue": 1,
  "durationUnit": "Day",
  "status": 1,
  "skus": [{
    "skuCode": "TICKET_ADULT_FULL",
    "skuName": "成人全价票",
    "specDesc": "{\"ageMin\": 19, \"ageMax\": 59}",
    "priceType": "FULL",
    "originalPrice": "45.00",
    "sellPrice": "40.00",
    "costPrice": "0.00",
    "needRealName": true,
    "maxBuyPerOrder": 5,
    "ageLimitMin": 19,
    "ageLimitMax": 59
  }, {
    "skuCode": "TICKET_ADULT_DISCOUNT",
    "skuName": "老人优惠票",
    "specDesc": "{\"ageRange\":\"60岁以上\"}",
    "priceType": "DISCOUNT",
    "originalPrice": "25.00",
    "sellPrice": "20.00",
    "costPrice": "0.00",
    "needRealName": true,
    "maxBuyPerOrder": 5,
    "ageLimitMin": 60,
    "ageLimitMax": null
  }]
}
```

### 2.3 编辑商品

**PUT** `/api/v1/admin/products/{spuId}` — 编辑 SPU 基本信息

**PUT** `/api/v1/admin/products/{spuId}/sku/{skuId}` — 编辑 SKU

### 2.4 组合产品管理

**POST** `/api/v1/admin/products/combo`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| comboCode | string | 是 | 组合编码 |
| comboName | string | 是 | 组合名称 |
| comboType | string | 是 | cross_category / cross_merchant / cross_scenic |
| totalPrice | string | 是 | 组合总价 |
| items | array | 是 | 明细列表 |
| items[].skuId | string | 是 | SKU ID |
| items[].quantity | int | 是 | 数量 |
| items[].sellPrice | string | 是 | 分项售价 |
| items[].settlePrice | string | 是 | 分项结算价（清分基准） |
| items[].required | boolean | 是 | 是否必选 |

---

### 2.5 价格日历批量设置

**PUT** `/api/v1/admin/products/{spuId}/price-calendar`

> 为指定 SKU 批量设置未来日期的价格和库存。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| skuId | string | 是 | SKU ID |
| entries | array | 是 | 日期条目 |
| entries[].date | string | 是 | yyyy-MM-dd |
| entries[].price | string | 是 | 当日价格 |
| entries[].stock | int | 是 | 当日库存 |

**请求示例：**
```json
{
  "skuId": "SKU001",
  "entries": [
    { "date": "2026-06-01", "price": "40.00", "stock": 1000 },
    { "date": "2026-06-02", "price": "40.00", "stock": 1000 }
  ]
}
```

### 2.6 渠道定价

**GET** `/api/v1/admin/products/{spuId}/channel-price?skuId=X` — 查询当前各渠道定价

**POST** `/api/v1/admin/products/{spuId}/channel-price`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| skuId | string | 是 | SKU ID |
| channel | string | 是 | self_miniapp / self_pc / self_kiosk / agent / ota / window |
| price | string | 是 | 渠道价格 |
| stockRatio | decimal | 否 | 库存分配比例（0-1） |

### 2.7 多语言内容

**POST** `/api/v1/admin/products/{spuId}/languages`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| languageCode | string | 是 | zh-CN / en / ja / ko / ru / fr / es / ar |
| title | string | 否 | 多语言名称 |
| description | string | 否 | 多语言描述 |
| notice | string | 否 | 多语言须知 |
| refundPolicy | string | 否 | 多语言退改规则 |
| inclusions | string | 否 | 多语言包含项 |
| exclusions | string | 否 | 多语言不含项 |
| highlight | string | 否 | 多语言亮点 |
| howToUse | string | 否 | 多语言使用方法 |

### 2.8 选项/时段管理

**PUT** `/api/v1/admin/products/{spuId}/options`

| 字段 | 类型 | 说明 |
|------|------|------|
| optionCode | string | 选项编码 |
| optionType | string | option / time_slot |
| values | array | 选项值列表 |
| values[].valueCode | string | 值编码 |
| values[].valueName | string | 值名称 |
| values[].sortNo | int | 排序 |

### 2.9 手动调整库存

**PUT** `/api/v1/admin/products/{spuId}/stock`

> 紧急情况下手动调整已售/可用库存。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| skuId | string | 是 | SKU ID |
| date | string | 是 | 日期 |
| timeSlot | string | 是 | 时段 |
| channel | string | 是 | 渠道 |
| newTotalStock | int | 否 | 新总库存 |
| adjustUsedStock | int | 否 | 调整已售（+增/-减，含变更原因） |
| adjustReason | string | 是 | 调整原因 |

---

### 2.10 商品批量导入导出

**POST** `/api/v1/admin/products/batch-import` — Excel 批量导入（multipart/form-data）

> Excel 模板列：`序号 | SPU编码 | SPU名称 | 业态分类 | 说明 | SKU编码 | SKU名称 | 价格类型(FULL/DISCOUNT/FREE) | 原价 | 售价 | 成本价 | 最大限购 | 年龄下限 | 年龄上限 | 是否需要实名(0/1)`

**GET** `/api/v1/admin/products/export` — 导出全量商品 Excel

---

## 3. 票务管理

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
| pageNo / pageSize | — | — |

**响应records：** reservationNo, contactName, contactPhone(脱敏), productName, visitDate, timeSlot, quantity, channel, statusText, createTime。

### 3.2 季节/时段配置

**GET** `/api/v1/admin/tickets/seasons`

**POST** `/api/v1/admin/tickets/seasons`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| configCode | string | 是 | 配置编码 |
| configName | string | 是 | 如"旺季日场" |
| productId | string | 是 | SKU ID |
| sceneType | string | 是 | day / night |
| startDate | date | 是 | 生效日期 |
| endDate | date | 是 | 结束日期 |
| timeSlots | array | 是 | `[{"slot":"08:00-10:00","max":500}]` |
| advanceDays | int | 是 | 最大提前预约天数 |
| releaseRule | string | 是 | auto / manual |
| status | int | 是 | 0/1 |

### 3.3 窗口售票（新开交易）

**POST** `/api/v1/admin/tickets/window-sell`

> US-004 窗口售票员专用。支持一体化售票界面（选票种/填游客/收款）。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| items | array | 是 | 同游客端下单格式 |
| contactName | string | 否 | 联系人（现场无联系人的可不传） |
| contactPhone | string | 否 | 联系人手机 |
| payMethod | string | 是 | wechat/alipay/unionpay/cash/bank_card/qrcode |
| cashierId | string | 是 | 收银员 sys_user.id |
| sessionId | string | 是 | 当前班次 ID |

**响应：** `{ "orderId": "...", "orderNo": "...", "verifyTime": "..." }`

### 3.3a 手动核销

**POST** `/api/v1/admin/tickets/verify-manual`

> 后台手动核销（游客遗失手机/证件特殊情况）。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| idType | string | 是 | 证件类型 |
| idNo | string | 是 | 证件号 |
| gateLocation | string | 是 | 核销口 |
| reason | string | 是 | 手动核销原因 |
| approveBy | string | 是 | 审批人用户名（手动核销需另一管理员审批，不能自审） |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "verifyResult": true,
    "visitorName": "张三",
    "productName": "成人全价票",
    "orderNo": "MTY06010001",
    "verifyTime": "2026-06-01 09:15:00"
  }
}
```

---

## 4. 订单管理

### 4.1 全量订单检索

**GET** `/api/v1/admin/orders`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| orderNo | string | 否 | 订单号精确搜索 |
| contactName | string | 否 | 联系人姓名 |
| contactPhone | string | 否 | 联系人手机 |
| idNo | string | 否 | 游客证件号 |
| travelDate | string | 否 | 出行日期 |
| status | int | 否 | 订单状态 |
| channel | string | 否 | 渠道 |
| distributionChannel | string | 否 | OTA 分销渠道 |
| createStart | string | 否 | 下单开始时间 |
| createEnd | string | 否 | 下单结束时间 |
| pageNo / pageSize | — | — |

**响应records 字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| orderId | string | 订单 ID |
| orderNo | string | 订单号 |
| channelOrderNo | string | 第三方单号（OTA/旅行社） |
| totalAmount | string | 总金额 |
| paidAmount | string | 实付 |
| status | int | — |
| statusText | string | — |
| contactName | string | 联系人 |
| contactPhone | string | 脱敏手机号 |
| travelDate | date | 出行日期 |
| channel | string | 渠道 |
| refundable | int | 是否可退 |
| createTime | datetime | — |

### 4.2 订单详情

**GET** `/api/v1/admin/orders/{orderId}`

> 含主订单 + 子订单 + 游客信息 + 支付流水 + 退款记录 + 操作时间线。

**响应关键字段：**

```json
{
  "data": {
    "orderId": "...",
    "orderNo": "...",
    "channelOrderNo": "...",
    "orderType": "ticket",
    "channel": "self_miniapp",
    "distributionChannel": null,
    "totalAmount": "75.00",
    "discountAmount": "5.00",
    "paidAmount": "70.00",
    "status": 2,
    "contactName": "张三",
    "contactPhone": "138****1234",
    "travelDate": "2026-06-01",
    "insuranceFlag": 0,
    "subOrders": [{
      "subOrderId": "SUB001",
      "subOrderNo": "SUB_MTY06010001_01",
      "productId": "SKU001",
      "productName": "成人全价票",
      "merchantId": null,
      "merchantName": "自营",
      "quantity": 1,
      "unitPrice": "40.00",
      "subAmount": "40.00",
      "verifyCount": 0,
      "status": 2,
      "visitors": [{
        "visitorId": "VIS001",
        "name": "张三",
        "idType": "ID_CARD",
        "idNo": "110***********1234",
        "verifyStatus": 0,
        "verifyTime": null,
        "verifyGate": null,
        "voucher": {
          "voucherCode": "6901234567890",
          "voucherType": "barcode_128"
        }
      }]
    }],
    "payment": {
      "paymentNo": "PAY06010001",
      "payMethod": "wechat",
      "payAmount": "70.00",
      "transactionId": "4200001234567890",
      "payTime": "2026-05-26 10:15:00"
    },
    "refunds": [],
    "timeline": [
      { "time": "2026-05-26 10:00:01", "action": "提交订单", "detail": "下单渠道: 小程序" },
      { "time": "2026-05-26 10:15:00", "action": "支付成功", "detail": "微信支付 ¥70.00" }
    ]
  }
}
```

### 4.3 强制退款

**POST** `/api/v1/admin/orders/{orderId}/refund-force`

> 超管权限。不受退票规则限制。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| subOrderIds | array | 否 | 指定子订单，空=全部 |
| refundType | string | 是 | full / partial |
| refundAmount | string | 否 | 手动指定退款金额 |
| reason | string | 是 | 强制退款原因 |

### 4.4 批量核销

**POST** `/api/v1/admin/orders/batch-verify`

> 旅行社团体一次性核销。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| orderId | string | 是 | 主订单 ID |
| visitorIds | array | 是 | 要核销的游客 ID 列表 |

### 4.5 单游客全域订单聚合

**GET** `/api/v1/admin/orders/visitor/{visitorId}`

> 查一个游客的所有历史订单（门票+组合+酒店+餐饮+文创+活动…），用于客服查询。

**响应：** 按订单类型分组的订单列表，含状态、金额、时间。

### 4.6 订单导出

**GET** `/api/v1/admin/orders/export`

> 按当前查询条件导出 Excel。查询参数同 4.1。

**响应：** `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`

---

## 5. 清分结算

### 5.1 清分明细

**GET** `/api/v1/admin/settlements`

| 参数 | 必填 | 说明 |
|------|------|------|
| merchantId | 否 | 商户筛选 |
| settleDate | 否 | 结算日期 |
| settleStatus | 否 | 0待结算/1已结算/2已打款 |
| pageNo | 否 | — |

### 5.2 对账单

**GET** `/api/v1/admin/settlements/ledgers` — 对账单列表

**POST** `/api/v1/admin/settlements/ledgers/generate` — 生成对账单

| 字段 | 必填 | 说明 |
|------|------|------|
| merchantId | 是 | 商户 ID |
| periodStart | 是 | 账期开始 |
| periodEnd | 是 | 账期结束 |

**PUT** `/api/v1/admin/settlements/ledgers/{ledgerId}/confirm` — 确认对账单

### 5.3 用友 NC 凭证

**GET** `/api/v1/admin/settlements/nc-vouchers` — 列表

**POST** `/api/v1/admin/settlements/nc-vouchers/sync` — 手动触发同步

---

## 6. 发票管理

### 6.1 开票申请审核

**GET** `/api/v1/admin/invoices/apply` — 待审核列表

**POST** `/api/v1/admin/invoices/apply/{applyId}/audit`

| 字段 | 必填 | 说明 |
|------|------|------|
| auditResult | 是 | 1通过 / 2驳回 |
| auditRemark | 否 | 驳回原因 |

### 6.2 开具/冲红

**POST** `/api/v1/admin/invoices/issue`

| 字段 | 必填 | 说明 |
|------|------|------|
| applyId | 是 | 申领 ID |

**POST** `/api/v1/admin/invoices/red` — 冲红

| 字段 | 必填 | 说明 |
|------|------|------|
| invoiceId | 是 | 发票记录 ID |
| redReason | 是 | 冲红原因 |

### 6.3 开票统计

**GET** `/api/v1/admin/invoices/stats`

| 参数 | 必填 | 说明 |
|------|------|------|
| startDate | 否 | 开始日期 |
| endDate | 否 | 结束日期 |

**响应：**
```json
{
  "data": {
    "totalApply": 1280,
    "totalIssued": 1245,
    "totalRed": 15,
    "totalAmount": "89450.00",
    "byCategory": [
      { "category": "旅游服务", "count": 1100, "amount": "68500.00" },
      { "category": "商品销售", "count": 145, "amount": "20950.00" }
    ]
  }
}
```

---

## 7. 商户管理

### 7.1 商户列表与审核

**GET** `/api/v1/admin/merchants`

| 参数 | 必填 | 说明 |
|------|------|------|
| status | 否 | 0待审核/1正常/2禁用/3驳回 |
| category | 否 | 业态筛选 |

**POST** `/api/v1/admin/merchants/{merchantId}/audit`

| 字段 | 必填 | 说明 |
|------|------|------|
| auditResult | 是 | 1通过 / 2驳回 |
| auditRemark | 否 | 审核备注 |

**PUT** `/api/v1/admin/merchants/{merchantId}/status` — 启用/停用商户 `{ "status": 1 }`

### 7.2 商户商品审核

**GET** `/api/v1/admin/merchants/products?auditStatus=0` — 待审核商品

**POST** `/api/v1/admin/merchants/products/{productId}/audit`

| 字段 | 必填 | 说明 |
|------|------|------|
| auditResult | 是 | 1通过 / 2驳回 |
| auditRemark | 否 | 驳回原因（价格不合理/内容违规等） |

### 7.3 佣金设置

**POST** `/api/v1/admin/merchants/{merchantId}/commission`

| 字段 | 必填 | 说明 |
|------|------|------|
| categoryCode | 是 | 业态 |
| commissionRate | 是 | 佣金比例（0.1000 = 10%） |
| effectiveDate | 是 | 生效日期 |

---

## 8. 营销管理

### 8.1 优惠券模板

**GET** `/api/v1/admin/marketing/coupons` — 列表

**POST** `/api/v1/admin/marketing/coupons`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| couponCode | string | 是 | 模板编码 |
| couponName | string | 是 | 券名称（"满100减20"） |
| couponType | string | 是 | discount / fixed / cash |
| couponValue | string | 是 | 面值/折扣率 |
| minAmount | string | 是 | 使用门槛 |
| totalCount | int | 是 | 发放总量 |
| perUserLimit | int | 是 | 每人限领 |
| productScope | string | 是 | all / category / product |
| productIds | array | 否 | 限定商品 ID 列表 |
| effectiveDays | int | 是 | 领取后有效天数 |
| startTime | datetime | 是 | 活动开始 |
| endTime | datetime | 是 | 活动结束 |
| status | int | 是 | 0/1 |

**POST** `/api/v1/admin/marketing/coupons/{couponId}/publish` — 发放（批量/定向）

### 8.2 秒杀活动

**POST** `/api/v1/admin/marketing/seckill`

| 字段 | 必填 | 说明 |
|------|------|------|
| seckillCode | 是 | 编码 |
| skuId | 是 | 商品 SKU |
| seckillPrice | 是 | 秒杀价 |
| stock | 是 | 秒杀库存（独立管理） |
| perUserLimit | 是 | 每人限购 |
| startTime | 是 | 开始时间 |
| endTime | 是 | 结束时间 |

### 8.3 拼团活动

**POST** `/api/v1/admin/marketing/group-buy`

| 字段 | 必填 | 说明 |
|------|------|------|
| groupCode | 是 | 编码 |
| skuId | 是 | 商品 SKU |
| groupPrice | 是 | 拼团价 |
| minCount | 是 | 成团人数 |
| expireHours | 是 | 成团时限（小时） |

### 8.4 全民分销

**GET** `/api/v1/admin/marketing/distributions` — 分销员列表

**GET** `/api/v1/admin/marketing/commissions` — 佣金记录

---

## 9. 会员管理

**GET** `/api/v1/admin/members`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| level | string | 否 | BRONZE/SILVER/GOLD/PLATINUM |
| tag | string | 否 | 标签筛选（如"亲子游"） |
| keyword | string | 否 | 会员编号/手机号 |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**GET** `/api/v1/admin/members/{userId}` — 会员详情（含等级/积分/消费/标签/权益）

**PUT** `/api/v1/admin/members/{userId}/tags` — 编辑标签 `{ "tags": "VIP,高频消费,亲子游" }`

**POST** `/api/v1/admin/members/{userId}/send-coupon` — 定向发券 `{ "couponId": "CP001" }`

**GET/POST** `/api/v1/admin/members/benefits` — 权益配置

---

## 10. 进销存管理

### 10.1 供应商

**GET/POST/PUT/DELETE** `/api/v1/admin/inventory/suppliers`

### 10.2 商品/物料

**GET/POST** `/api/v1/admin/inventory/goods` — 含条码、规格、成本价、售价

### 10.3 入库

**POST** `/api/v1/admin/inventory/inbound`

| 字段 | 必填 | 说明 |
|------|------|------|
| supplierId | 是 | 供应商 ID |
| storeId | 是 | 目标门店 |
| items | 是 | 入库商品明细 |
| items[].goodsId | 是 | 商品 ID |
| items[].quantity | 是 | 入库数量 |
| items[].unitPrice | 是 | 采购单价 |

> 入库成功后自动生成一物一码（unique_code），打印唯一条码标签。

### 10.4 出库/调拨

**POST** `/api/v1/admin/inventory/outbound`

| 字段 | 必填 | 说明 |
|------|------|------|
| outboundType | 是 | sale / transfer / internal_use |
| fromStoreId | 是 | 来源门店 |
| toStoreId | 否 | 目标门店（调拨时） |
| items | 是 | 出库明细 |

### 10.5 库存查询

**GET** `/api/v1/admin/inventory/stock?storeId=X&goodsId=Y&keyword=Z`

### 10.6 盘点

**POST** `/api/v1/admin/inventory/check`

| 字段 | 必填 | 说明 |
|------|------|------|
| storeId | 是 | 门店 |
| checkType | 是 | regular / temporary |
| checkDate | 是 | 盘点日期 |

**PUT** `/api/v1/admin/inventory/check/{checkId}/adjust` — 确认差异调整，自动更新库存（差异数自动生成入库/出库单）

| 字段 | 必填 | 说明 |
|------|------|------|
| items | 是 | 盘点明细 |
| items[].goodsId | 是 | 商品 |
| items[].actualQty | 是 | 实际数量 |
| items[].diffReason | 否 | 差异原因 |

### 10.7 收银记录

**GET** `/api/v1/admin/inventory/checkouts?storeId=X&cashierId=Y&sessionId=Z&startDate=A&endDate=B`

### 10.8 收银班次

**POST** `/api/v1/admin/cashier/sessions` — 开班 `{ "sellerId": "S001", "areaId": "AREA_001" }`

**PUT** `/api/v1/admin/cashier/sessions/{sessionId}/close` — 收班（自动汇总本班售票/退票/净收入）

**GET** `/api/v1/admin/cashier/sessions` — 班次列表 `?sellerId=X&startDate=Y&endDate=Z`

**GET** `/api/v1/admin/cashier/sessions/{sessionId}` — 班次日结详情

---

## 11. 规则管理

### 11.1 销售规则

**GET/POST/PUT** `/api/v1/admin/rules/sale`

**请求示例（创建规则）：**
```json
{
  "ruleCode": "RULE_PASSPORT_ELDER",
  "ruleName": "禁止护照购买老人票",
  "targetType": "sku",
  "targetId": "SKU002",
  "ruleType": "id_type_limit",
  "ruleParams": "{\"blocked_id_types\":[\"PASSPORT\"]}",
  "errorMsg": "外籍游客请购买全价票",
  "priority": 10,
  "status": 1
}
```

### 11.2 退票规则

**POST** `/api/v1/admin/rules/refund`

| 字段 | 必填 | 说明 |
|------|------|------|
| ruleCode | 是 | 编码 |
| ruleName | 是 | 规则名 |
| targetType | 是 | spu / sku / combo |
| targetId | 是 | 绑定对象 ID |
| refundTimeScope | 是 | before_visit / after_visit / anytime |
| hoursBefore | 否 | 游览前 N 小时内 |
| refundRate | 是 | 退款比例（1.0000=全额） |
| allowPartial | 是 | 是否允许部分退（布尔） |
| cancelFeeJson | 否 | 携程格式退改费率 `[{"dayBeforeVisitDate":1,"time":"00:00","unit":"PERCENTAGE","value":50}]` |

### 11.3 验票规则

**POST** `/api/v1/admin/rules/verify`

| 字段 | 必填 | 说明 |
|------|------|------|
| ruleCode | 是 | 编码 |
| ruleName | 是 | "南北口防重复入园" |
| targetType | 是 | spu / sku / scenic_spot / scenic_area |
| targetId | 是 | 绑定对象 ID |
| ruleType | 是 | unique_entry / verify_deadline / offline_allow / gate_whitelist / single_use |
| ruleParams | 是 | 参数 JSON |

---

## 12. 预警与监控

### 12.1 预警规则

**GET/POST** `/api/v1/admin/alerts/rules`

| 字段 | 必填 | 说明 |
|------|------|------|
| alertCode | 是 | 编码 |
| alertName | 是 | "游客量骤降预警" |
| alertType | 是 | visitor_drop / sales_drop / device_fault / capacity / ... |
| targetType | 是 | 监控对象类型 |
| targetId | 是 | 监控对象 ID |
| metricField | 是 | 监控字段 |
| compareType | 是 | absolute / yoy / mom |
| threshold | 是 | 阈值 |
| thresholdDirection | 是 | below / above |
| notifyChannels | 是 | sms,email,wechat,led,weibo |
| notifyRoles | 是 | 通知角色 |

**POST** `/api/v1/admin/alerts/rules/{id}/test` — 测试发送（向当前用户发送模拟预警通知，验证通知渠道配置）

### 12.2 预警日志

**GET** `/api/v1/admin/alerts/logs?alertId=X&startTime=Y&endTime=Z`

---

## 13. 设备管理

**GET/POST/PUT/DELETE** 统一模式，按设备类型分路由：

| 设备 | 路由 |
|------|------|
| 闸机 | `/api/v1/admin/devices/gates` |
| 手持机 | `/admin/devices/handhelds` |
| 打印机 | `/admin/devices/printers` |
| 扫描枪 | `/admin/devices/scanners` |
| 护照阅读器 | `/admin/devices/passport-readers` |
| 客流计数器 | `/admin/devices/counters` |
| 收银机 | `/admin/devices/cashiers` |

**闸机 POST 请求示例：**
```json
{
  "gateCode": "GATE_SOUTH_01",
  "gateName": "南口1号闸机",
  "scenicSpotId": "SPOT_001",
  "areaId": "AREA_001",
  "gateLocation": "南口1号通道",
  "gateType": "fixed",
  "ipAddress": "192.168.1.101",
  "macAddress": "AA:BB:CC:DD:EE:01"
}
```

**GET** `/api/v1/admin/devices/gates/{gateId}/heartbeat` — 心跳历史

**POST** `/api/v1/admin/devices/gates/{gateId}/sync-local` — 强制同步底库数据至闸机本地

---

## 14. 旅行社管理

**GET** `/api/v1/admin/travel-agencies?auditStatus=X&groupType=Y`

**POST** `/api/v1/admin/travel-agencies/{agencyId}/audit`

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/group-type` — 设置分组 { "groupType": "key" }

**GET** `/api/v1/admin/travel-agencies/{agencyId}/accounts` — 账户信息

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/credit` — 调整授信 { "creditLimit": "50000.00" }

**GET** `/api/v1/admin/travel-agencies/{agencyId}/transactions` — 账户流水

**POST** `/api/v1/admin/travel-agencies/{agencyId}/group-prices` — 设置分组定价
```json
{
  "prices": [
    { "skuId": "SKU001", "price": "35.00" },
    { "skuId": "SKU002", "price": "18.00" }
  ]
}
```

---

## 15. 报表

### 15.1 经营大屏

**GET** `/api/v1/admin/reports/dashboard`

**响应示例：**
```json
{
  "data": {
    "today": {
      "visitorCount": 8520,
      "yesterdayCompare": "+12%",
      "onlineRatio": "68%",
      "ticketRevenue": "340800.00",
      "totalRevenue": "562400.00"
    },
    "realTime": {
      "currentInPark": 3240,
      "ticketSold": { "08-10": 1200, "10-12": 2400, "12-14": 1800 },
      "gateStats": [
        { "gateName": "南口", "verified": 4520, "waiting": 120 },
        { "gateName": "北口", "verified": 4000, "waiting": 80 }
      ]
    },
    "ranking": {
      "otaTop5": ["携程 3200", "美团 2100", "抖音 1500", "去哪儿 800", "小红书 500"],
      "agentTop5": ["中国国旅 1200", "中青旅 800", "..."]
    },
    "visitorProfile": {
      "domesticRatio": "82%",
      "sourceTop5": ["北京 35%", "河北 15%", "天津 10%", "上海 8%", "广东 6%"],
      "genderRatio": { "male": "48%", "female": "52%" },
      "ageDistribution": { "0-18": "12%", "19-35": "38%", "36-55": "32%", "56+": "18%" }
    }
  }
}
```

### 15.2 售票报表

**GET** `/api/v1/admin/reports/ticket?startDate=X&endDate=Y&granularity=day/hour`

**响应：** 按日期/时段的售出数量 + 金额 + 渠道占比趋势。

### 15.3 营收报表

**GET** `/api/v1/admin/reports/revenue?startDate=X&endDate=Y&dimension=product/channel/seller`

### 15.4 游客画像分析

**GET** `/api/v1/admin/reports/visitor-profile?startDate=X&endDate=Y`

**响应：** 客源地分布、年龄性别、客单价区间、平均停留时长、复游率。

### 15.5 销售核销差异报表

**GET** `/api/v1/admin/reports/diff?date=X`

> 按核销日期做账时，销售数 vs 核销数的差异分析。

### 15.6 退票专题分析

**GET** `/api/v1/admin/reports/refund?startDate=X&endDate=Y`

**响应：** 退票数量、退票金额、退票率、退票原因分布、强制退票占比。

### 15.7 自定义报表

**POST** `/api/v1/admin/reports/custom`

> 通过 JimuReport 自定义报表查询。

| 字段 | 必填 | 说明 |
|------|------|------|
| reportId | 是 | 报表模板 ID |
| params | 否 | 参数 Map |

### 15.8 报表导出

**GET** `/api/v1/admin/reports/{type}/export`

> 各类型报表导出为文件。

| 参数 | 必填 | 说明 |
|------|------|------|
| type | 是 | ticket / revenue / visitor-profile / diff / refund |
| format | 是 | xlsx / pdf |
| startDate | 是 | 开始日期 |
| endDate | 是 | 结束日期 |
| ...其他报表查询参数 | 否 | 同各报表查询参数 |

**响应：** `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` 或 `application/pdf`

---

## 16. 年卡管理

**GET** `/api/v1/admin/annual-cards?status=X&keyword=Y&pageNo=1`

**GET** `/api/v1/admin/annual-cards/{cardNo}` — 年卡详情（含持卡人/证件/有效期/人脸状态）

**POST** `/api/v1/admin/annual-cards/issue` — 窗口办卡 `{ "skuId": "...", "name": "...", "idType": "...", "idNo": "<AES>", "faceImageId": "..." }`

**PUT** `/api/v1/admin/annual-cards/{cardNo}/status` — 状态变更（激活/挂失/注销）`{ "status": 2 }`

**PUT** `/api/v1/admin/annual-cards/{cardNo}/face-rebind` — 人脸重绑 `{ "faceImageId": "..." }`

---

## 17. 租赁管理

**GET** `/api/v1/admin/rental/devices` — 设备台账列表 `?status=X&areaId=Y`

**POST/PUT/DELETE** `/api/v1/admin/rental/devices` — 设备台账 CRUD

**GET** `/api/v1/admin/rental/orders` — 租赁订单列表 `?status=X&deviceNo=Y`

**GET** `/api/v1/admin/rental/orders/{rentalNo}` — 租赁订单详情

**PUT** `/api/v1/admin/rental/orders/{rentalNo}/damage` — 标记设备损坏 `{ "damageRemark": "...", "deductionAmount": "50.00" }`

---

## 18. AI 知识库管理

**GET** `/api/v1/admin/ai/knowledge?category=X&keyword=Y` — 知识条目列表

**POST/PUT/DELETE** `/api/v1/admin/ai/knowledge` — CRUD

**GET** `/api/v1/admin/ai/sessions` — AI 对话日志 `?userId=X&status=Y`

**GET** `/api/v1/admin/ai/sessions/{sessionId}/messages` — 对话记录

**POST** `/api/v1/admin/ai/messages/{msgId}/correct` — 对话纠错 `{ "correctContent": "..." }`

---

## 19. 分销提现审批

**GET** `/api/v1/admin/marketing/withdrawals?status=X` — 提现申请列表

**PUT** `/api/v1/admin/marketing/withdrawals/{withdrawId}/approve` — 审批 `{ "result": 1, "remark": "" }`

---

## 20. 电子围栏

**GET/POST/PUT/DELETE** `/api/v1/admin/geofences` — 围栏 CRUD

---

## 21. 消息模板与批量通知

**GET/POST/PUT** `/api/v1/admin/message-templates` — 模板 CRUD（短信/邮件/微信）

**POST** `/api/v1/admin/notifications/batch` — 批量发送 `{ "templateId": "T001", "targetType": "role/merchant/user", "targetIds": [...], "params": {...} }`

---

## 22. 客服会话管理

**GET** `/api/v1/admin/cs/sessions` — 客服会话列表 `?status=queuing/active&agentId=X`

**PUT** `/api/v1/admin/cs/sessions/{sessionId}/assign` — 指派坐席 `{ "agentId": "..." }`

**GET** `/api/v1/admin/cs/sessions/{sessionId}/messages` — 会话消息查询

---

## 23. 内容管理

**GET/POST/PUT** `/api/v1/admin/content/articles` — 资讯管理

**GET** `/api/v1/admin/content/notes?auditStatus=0` — 笔记审核列表

**POST** `/api/v1/admin/content/notes/{noteId}/audit` — 审核 { "auditResult": 1, "remark": "" }

**PUT** `/api/v1/admin/content/notes/{noteId}/top` — 置顶/加精 { "isTop": true, "isEssence": true }

**GET/POST/PUT** `/api/v1/admin/content/audio` — 语音讲解管理

| 字段 | 必填 | 说明 |
|------|------|------|
| scenicSpot | 是 | 景点英文标识 |
| audioUrl | 是 | 音频 OSS URL |
| transcript | 否 | 文字稿 |
| language | 是 | zh-CN / en |
| duration | 否 | 时长（秒） |

**GET/POST/PUT** `/api/v1/admin/content/routes` — 导览路线

| 字段 | 必填 | 说明 |
|------|------|------|
| routeCode | 是 | 路线编码 |
| routeName | 是 | 路线名称 |
| areaId | 是 | 景区 |
| routeType | 是 | recommend / family / senior / quick |
| waypoints | 是 | `[{"spotId":"...","order":1,"duration":30}]` |

**GET/POST/PUT** `/api/v1/admin/content/map/{markerId}` — 地图标注

---

## 17. OTA 管理

### 17.1 OTA 商品映射

**GET** `/api/v1/admin/ota/mappings?otaCode=X&status=Y`

**POST** `/api/v1/admin/ota/mappings`

| 字段 | 必填 | 说明 |
|------|------|------|
| skuId | 是 | 内部 SKU |
| otaCode | 是 | ctrip / meituan / qunar / douyin / xiaohongshu / feizhu / linktivity |
| otaOptionId | 是 | OTA 侧 OptionId |
| otaSupplierOptionId | 否 | OTA 侧 PLU |
| otaProductId | 否 | OTA 产品 ID |
| contractId | 是 | OTA 合同 ID |
| dateType | 是 | DATE_REQUIRED / DATE_NOT_REQUIRED |
| syncPrice | 是 | 是否同步价格（布尔） |
| syncInventory | 是 | 0/1 |
| pushEnabled | 是 | 是否启用推送（布尔） |

### 17.2 OTA POI 映射

**GET/POST** `/api/v1/admin/ota/poi-mappings`

### 17.3 OTA 同步日志

**GET** `/api/v1/admin/ota/sync-logs?otaCode=X&syncType=Y&status=Z&startTime=A&endTime=B`

### 17.4 OTA 对账

**GET** `/api/v1/admin/ota/reconciliations?otaCode=X&bizDate=Y`

**POST** `/api/v1/admin/ota/reconciliations/process` — 处理差异

| 字段 | 必填 | 说明 |
|------|------|------|
| reconciliationId | 是 | 对账单 ID |
| action | 是 | confirm / ignore |
| remark | 否 | 备注 |

### 17.5 OTA 通道管理

**POST** `/api/v1/admin/ota/mappings/{mappingId}/resync` — 手动触发重新同步（产品/价格/库存）

**PUT** `/api/v1/admin/ota/mappings/{mappingId}/toggle` — 启用/停用 OTA 通道 `{ "pushEnabled": false }`
