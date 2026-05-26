# 游客端 API — `/api/v1/tourist/`

> 面向小程序、H5、PC 购票端的终端游客。查询类接口可选 Token，下单、个人中心等需要登录。

---

## 1. 商品浏览

### 1.0 景区实时信息

**GET** `/api/v1/tourist/scenic/{areaId}/realtime`

> 无需 Token。返回景区实时数据（天气、排队、舒适度等）。

**响应示例：**
```json
{ "code": 0, "data": { "areaId": "AREA_001", "weather": { "temperature": 22, "condition": "晴" }, "crowdLevel": "适中", "queueTime": { "southGate": "5分钟" } } }
```

### 1.0a 个性化推荐

**GET** `/api/v1/tourist/products/recommend`

> 基于当前浏览商品或历史订单的个性化推荐。无需 Token（未登录使用全局热门推荐）。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| spuId | string | 否 | 基于此商品推荐关联商品 |
| lang | string | 否 | 语言，默认 Accept-Language |

### 1.1 商品列表

**GET** `/api/v1/tourist/products`

> 首页/列表页商品搜索。返回 SPU 级别商品列表，每个 SPU 附带最低价 SKU 信息。

**Auth:** 可选 Token（传 Token 则返回个性化价格）。

**请求参数（Query）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| category | string | 否 | 业态分类：TICKET / COMBO / SPOT / VEHICLE / PERFORMANCE / HOTEL / CATERING / CULTURAL / STUDY / RENTAL / PARKING |
| keyword | string | 否 | 搜索关键词（商品名称模糊匹配） |
| visitDate | string | 否 | 游览日期 yyyy-MM-dd。传入后返回当日有库存的商品及实时价格 |
| timeSlot | string | 否 | 时段筛选，如 `08:00-10:00` |
| channel | string | 否 | 渠道：self_miniapp / self_pc / self_kiosk / agent / ota / window。默认当前端 |
| lang | string | 否 | 语言：zh-CN / en / ja / ko 等，影响返回的商品名称和描述 |
| sortBy | string | 否 | price_asc / price_desc / sales_desc。默认按推荐排序 |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20，最大 100 |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "records": [
      {
        "spuId": "SPU001",
        "spuCode": "TICKET_ADULT",
        "spuName": "绥中长城成人票",
        "categoryCode": "TICKET",
        "scenicSpotId": "SPOT_001",
        "scenicSpotName": "绥中长城景区",
        "scenicAreaId": "AREA_001",
        "mainImage": "https://cdn.example.com/images/p001.jpg",
        "minPrice": "40.00",
        "maxPrice": "60.00",
        "highlight": ["世界文化遗产", "5A级景区", "明代长城精华段"],
        "tags": ["热门", "必去"],
        "deliveryMethod": "DIGITAL",
        "redemptionType": "Direct_Entry",
        "duration": { "value": 1, "unit": "Day" },
        "bookingCutoffTime": { "dayBeforeVisitDate": 0, "time": "08:00" },
        "soldCount": 12580,
        "status": 1
      },
      {
        "spuId": "SPU002",
        "spuCode": "COMBO_TICKET_CABLE",
        "spuName": "门票+缆车组合套票",
        "categoryCode": "COMBO",
        "mainImage": "https://cdn.example.com/images/p002.jpg",
        "minPrice": "90.00",
        "highlight": ["超值组合", "节省20元"],
        "deliveryMethod": "DIGITAL",
        "redemptionType": "Direct_Entry",
        "status": 1
      }
    ],
    "total": 45,
    "pageNo": 1,
    "pageSize": 20
  }
}
```

---

### 1.2 商品详情

**GET** `/api/v1/tourist/products/{spuId}`

> 商品详情页。含完整 SPU 信息、多语言内容、SKU 列表、购买须知、退改规则。

**Auth:** 可选 Token。

**路径参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| spuId | string | SPU ID |

**查询参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| lang | string | 否 | 语言。默认请求头 Accept-Language，缺省 zh-CN。`spuName`/`description`/`notice`/`refundPolicy` 等字段由 `product_language` 表驱动，未录入时 fallback 至默认语言 |
| visitDate | string | 否 | 传入后返回当日价格和库存 |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "spuId": "SPU001",
    "spuCode": "TICKET_ADULT",
    "spuName": "绥中长城成人票",
    "categoryCode": "TICKET",
    "scenicSpotId": "SPOT_001",
    "mainImage": "https://cdn.example.com/images/p001.jpg",
    "images": [
      "https://cdn.example.com/images/p001_1.jpg",
      "https://cdn.example.com/images/p001_2.jpg"
    ],
    "description": "绥中长城位于北京市怀柔区，是明代长城...",
    "highlight": ["世界文化遗产", "5A级景区"],
    "inclusions": ["成人门票1张"],
    "exclusions": ["缆车费用", "个人消费"],
    "howToUse": ["凭二维码扫码入园", "无需换票", "限选定日期当日有效"],
    "additionalInfo": "60岁以上老人凭身份证免票入园",
    "notice": "1. 请提前预约时段\n2. 禁止携带宠物\n3. 景区内禁止吸烟",
    "refundPolicy": "游览前1天20:00前免费取消，之后收取50%手续费，游览当日不可退",
    "deliveryMethod": "DIGITAL",
    "redemptionType": "Direct_Entry",
    "bookingCutoffTime": { "dayBeforeVisitDate": 0, "time": "08:00" },
    "duration": { "value": 1, "unit": "Day" },
    "skus": [
      {
        "skuId": "SKU001",
        "skuName": "成人全价票",
        "priceType": "FULL",
        "specDesc": { "ageMin": 19, "ageMax": 59, "idType": ["ID_CARD", "PASSPORT"] },
        "sellPrice": "40.00",
        "originalPrice": "45.00",
        "needRealName": true,
        "maxBuyPerOrder": 5,
        "ageLimitMin": 19,
        "ageLimitMax": 59
      },
      {
        "skuId": "SKU002",
        "skuName": "老人优惠票",
        "priceType": "DISCOUNT",
        "specDesc": { "ageMin": 60, "ageMax": null, "idType": ["ID_CARD"] },
        "sellPrice": "20.00",
        "originalPrice": "25.00",
        "needRealName": true,
        "maxBuyPerOrder": 5,
        "ageLimitMin": 60,
        "ageLimitMax": null
      }
    ],
    "options": [
      {
        "optionCode": "Time_Slot",
        "optionType": "time_slot",
        "values": [
          { "valueCode": "TS_1", "valueName": "08:00-10:00", "available": true },
          { "valueCode": "TS_2", "valueName": "10:00-12:00", "available": true },
          { "valueCode": "TS_3", "valueName": "12:00-14:00", "available": false, "unavailableReason": "已售罄" }
        ]
      }
    ]
  }
}
```

---

### 1.3 价格日历

**GET** `/api/v1/tourist/products/{spuId}/calendar`

> 查某商品未来 90 天的每日价格和库存状态。

**请求参数（Query）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| skuId | string | 是 | SKU ID |
| startDate | string | 否 | 起始日期 yyyy-MM-dd，默认今天 |
| endDate | string | 否 | 结束日期，默认今天+90天 |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "skuId": "SKU001",
    "skuName": "成人全价票",
    "calendar": [
      { "date": "2026-05-26", "price": "40.00", "stock": 800, "status": "available" },
      { "date": "2026-05-27", "price": "40.00", "stock": 450, "status": "available" },
      { "date": "2026-05-28", "price": "45.00", "stock": 0, "status": "sold_out" },
      { "date": "2026-05-29", "price": "45.00", "stock": 200, "status": "available" }
    ]
  }
}
```

| status | 说明 |
|--------|------|
| available | 可购 |
| sold_out | 售罄 |
| off_sale | 停售（景区关闭日/维护日） |
| not_yet | 尚未开放预订 |

---

### 1.4 组合产品列表

**GET** `/api/v1/tourist/products/combo`

> 组合套餐列表，如门票+缆车、门票+餐饮等。

**请求参数（Query）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| category | string | 否 | 组合类型：cross_category / cross_merchant / cross_scenic |
| visitDate | string | 否 | 游览日期 |
| pageNo | int | 否 | 默认 1 |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "records": [{
      "comboId": "COMBO001",
      "comboName": "门票+缆车超值套票",
      "comboType": "cross_category",
      "totalPrice": "90.00",
      "originalPrice": "110.00",
      "description": "含绥中长城成人票1张 + 上行缆车1张，比单独购买节省20元",
      "items": [
        { "skuId": "SKU001", "skuName": "成人全价票", "quantity": 1, "sellPrice": "40.00" },
        { "skuId": "SKU100", "skuName": "上行缆车票", "quantity": 1, "sellPrice": "50.00" }
      ],
      "status": 1
    }],
    "pageNo": 1
  }
}
```

---

## 2. 购物车

### 2.1 购物车管理

> 未登录用户以设备 ID 标识购物车，登录后自动合并。

**GET** `/api/v1/tourist/cart`

**Auth:** 可选 Token。未登录时使用 Header `X-Device-Id`。

**POST** `/api/v1/tourist/cart` — 加入购物车
```json
{ "skuId": "SKU001", "date": "2026-06-01", "timeSlot": "08:00-10:00", "quantity": 1 }
```

**PUT** `/api/v1/tourist/cart/{cartItemId}` — 修改数量

**DELETE** `/api/v1/tourist/cart/{cartItemId}` — 移除

**POST** `/api/v1/tourist/cart/merge` — 登录后合并设备购物车（前端调起）

---

## 3. 下单与支付

### 3.1 订单预览（价格试算 + 规则校验）

**POST** `/api/v1/tourist/orders/preview`

> 下单前预览总价，同时校验销售规则（年龄限制、证件类型、限购、库存等）。不锁定库存。

**Auth:** 可选（登录后可享受会员价）。

**请求参数（Body）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| channel | string | 是 | 渠道：self_miniapp / self_pc / self_kiosk |
| items | array | 是 | 订单项列表 |
| couponId | string | 否 | 优惠券 ID |

**items 元素：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| spuId | string | 是 | SPU ID |
| skuId | string | 是 | SKU ID |
| comboId | string | 否 | 组合产品 ID（组合产品下单时传入，非组合产品为空） |
| visitDate | string | 是 | 游览日期 yyyy-MM-dd |
| timeSlot | string | 是 | 时段，如 `08:00-10:00` |
| quantity | int | 是 | 购买数量 |
| visitors | array | 是 | 游客信息（数量 = quantity） |

**visitors 元素：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 真实姓名 |
| idType | string | 是 | 证件类型：ID_CARD / PASSPORT / HK_MO / TAIWAN / PERMANENT_RESIDENCE |
| idNo | string | 是 | 证件号（AES 加密） |
| nationality | string | 否 | 国籍（外宾必填） |
| gender | string | 否 | M / F / U |
| age | int | 否 | 年龄（前端传入用于规则校验） |
| phone | string | 否 | 手机号（AES 加密） |

**请求示例：**
```json
{
  "channel": "self_miniapp",
  "items": [{
    "spuId": "SPU001",
    "skuId": "SKU001",
    "visitDate": "2026-06-01",
    "timeSlot": "08:00-10:00",
    "quantity": 1,
    "visitors": [{
      "name": "张三",
      "idType": "ID_CARD",
      "idNo": "<AES-encrypted>",
      "gender": "M",
      "age": 30
    }]
  }],
  "couponId": "CP001"
}
```

**响应示例（成功）：**
```json
{
  "code": 0,
  "data": {
    "items": [{
      "skuId": "SKU001",
      "skuName": "成人全价票",
      "quantity": 1,
      "unitPrice": "40.00",
      "subAmount": "40.00"
    }],
    "totalAmount": "40.00",
    "discountAmount": "5.00",
    "paidAmount": "35.00",
    "couponDiscount": "5.00",
    "valid": true
  }
}
```

**响应示例（规则校验失败）：**
```json
{
  "code": 1001,
  "msg": "订单校验未通过",
  "data": {
    "valid": false,
    "errors": [
      {
        "skuId": "SKU001",
        "visitorIndex": 0,
        "errorCode": "RULE_AGE_LIMIT",
        "message": "该票种要求年龄 19-59岁，当前游客年龄为 17岁，不满足购买条件"
      }
    ]
  }
}
```

---

### 3.2 创建订单

**POST** `/api/v1/tourist/orders`

> 下单成功即锁定库存（Redis + DB 双重锁）。支付超时自动释放库存。

**Auth:** Bearer Token（必填）。

**请求头：**
```
X-Idempotent-Key: uuid-v4
```

**请求参数（Body）：** 同订单预览接口 `items` + 以下字段：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| contactName | string | 是 | 联系人姓名 |
| contactPhone | string | 是 | 联系人手机（E.164 格式: `+[国家码][号码]`，AES 加密传输） |
| couponId | string | 否 | 优惠券 ID |
| insuranceFlag | bool | 否 | 是否购买保险 |
| insuranceProductId | string | 否 | 保险产品 SKU ID（insuranceFlag=true 时必填） |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "orderId": "ORD202606010001",
    "orderNo": "MTY06010001",
    "totalAmount": "35.00",
    "discountAmount": "5.00",
    "paidAmount": "35.00",
    "status": 1,
    "statusText": "待支付",
    "expireTime": "2026-05-26 10:30:00",
    "payParams": {
      "payMethod": "wechat_jsapi",
      "prepayId": "wx261030xxxxxxxx",
      "nonceStr": "abc123",
      "timeStamp": "1716710400",
      "signType": "RSA",
      "paySign": "xxxxxxxx"
    }
  }
}
```

**错误码：**

| code | 说明 |
|------|------|
| BIZ_002 | 库存不足/时段已售罄 |
| USR_001 | 用户不存在或未实名 |
| AUTH_003 | 证件号重复下单（防黄牛） |

> **库存锁定流程：** Redis `SET lock:ticket:SKU001:2026-06-01:08:00-10:00 NX EX 1800` → DB `update ticket_stock set used_stock = used_stock + N where used_stock + N <= total_stock` → 失败回滚 Redis。
> **支付超时处理：** 订单创建时 `ticket_visitor.verify_status` 标记为 0（待支付）。支付超时（30分钟未支付）由定时任务扫描 `order_main.status=1 AND expire_time < NOW()` → 批量取消订单 → 释放库存 → 标记 `ticket_visitor.del_flag=1` + `order_main.status=5`。

---

### 3.3 发起支付

**POST** `/api/v1/tourist/orders/{orderId}/pay`

> 返回前端支付参数（JSAPI / App SDK / H5 收银台）。

**Auth:** Bearer Token。

**请求参数（Body）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| payMethod | string | 是 | wechat / alipay / unionpay / apple_pay / google_pay / visa / master |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "payMethod": "wechat_jsapi",
    "orderNo": "MTY06010001",
    "paidAmount": "35.00",
    "payParams": {
      "appId": "wx1234567890",
      "prepayId": "wx261030xxxxxxxx",
      "nonceStr": "abc123",
      "timeStamp": "1716710400",
      "signType": "RSA",
      "paySign": "xxxxxxxx"
    }
  }
}
```

**错误码：**

| code | 说明 |
|------|------|
| BIZ_003 | 订单状态不允许支付（已支付/已取消/已过期） |
| PAY_001 | 支付渠道不可用 |
| PAY_002 | 支付金额与订单金额不一致 |

---

### 3.4 支付状态查询

**GET** `/api/v1/tourist/orders/{orderId}/pay-status`

**Auth:** Bearer Token。

**轮询建议：** 前端调起支付后每 2 秒轮询一次，最多轮询 30 次。

**响应示例（支付中）：**
```json
{ "code": 0, "data": { "status": 1, "statusText": "待支付" } }
```

**响应示例（支付成功）：**
```json
{
  "code": 0,
  "data": {
    "status": 2,
    "statusText": "已支付",
    "paidTime": "2026-05-26 10:15:00",
    "voucherUrl": "https://cdn.example.com/vouchers/MTY06010001/qrcode.png"
  }
}
```

---

### 3.5 改签

**PUT** `/api/v1/tourist/orders/{orderId}/reschedule`

> 改签到其他日期/时段。校验目标库存。

**Auth:** Bearer Token。

**请求参数（Body）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| newVisitDate | string | 是 | 新游览日期 |
| newTimeSlot | string | 是 | 新时段 |

**响应示例：**
```json
  "code": 0, "msg": "改签成功",
  "data": { "orderId": "ORD202606010001", "newVisitDate": "2026-06-02", "newTimeSlot": "10:00-12:00", "reservationStatus": 3 }
}
```

**错误码：**

| code | 说明 |
|------|------|
| BIZ_003 | 已核销不可改签 |
| BIZ_002 | 目标时段库存不足 |

---

### 3.6 取消订单

**POST** `/api/v1/tourist/orders/{orderId}/cancel`

> 未支付订单可直接取消，已支付订单取消即申请退款。

**响应示例：**
```json
{ "code": 0, "msg": "订单已取消" }
```

---

### 3.7 申请退款

**POST** `/api/v1/tourist/orders/{orderId}/refund`

> 已支付订单申请退款。系统根据退票规则计算退款金额和手续费。**多条退票规则按 priority 升序排列，第一条命中即生效。** 响应中返回命中的规则 ID 供前端展示规则详情。

**Auth:** Bearer Token。

**请求参数（Body）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| subOrderIds | array | 否 | 部分退票时传入具体子订单 ID。不传则全部退款 |
| refundReason | string | 是 | 退款原因 |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "refundNo": "REF06010001",
    "refundAmount": "28.00",
    "feeAmount": "7.00",
    "feeRate": "0.2",
    "feeReason": "游览前1天内退票收取20%手续费",
    "appliedRuleId": "RULE_REF_001",
    "appliedRuleName": "成人票退票规则",
    "status": 1,
    "statusText": "待审核"
  }
}
```

**错误码：**

| code | 说明 |
|------|------|
| BIZ_003 | 订单状态不可退款 |
| BIZ_004 | 部分票种不支持退款 |
| REFUND_001 | 退票规则不允许（如已过退票时限） |

**GET** `/api/v1/tourist/orders/{orderId}/refund-status` — 查询退款进度

---

## 4. 订单管理

### 4.1 我的订单列表

**GET** `/api/v1/tourist/orders`

**Auth:** Bearer Token。

**请求参数（Query）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | int | 否 | 1待支付 / 2已支付 / 3已核销 / 4已退款 / 5已取消 / 6部分退款 / 7已关闭 |
| startDate | string | 否 | 下单开始日期 |
| endDate | string | 否 | 下单结束日期 |
| keyword | string | 否 | 订单号/商品名关键字 |
| pageNo | int | 否 | 默认 1 |

**响应字段（records 元素）：**

| 字段 | 类型 | 说明 |
|------|------|------|
| orderId | string | 订单 ID |
| orderNo | string | 订单号 |
| totalAmount | string | 总金额 |
| paidAmount | string | 实付 |
| status | int | 1/2/3/4/5/6 |
| statusText | string | 待支付 / 已支付 / ... |
| subOrders | array | 子订单摘要（产品名+数量） |
| contactName | string | 联系人 |
| travelDate | date | 出行日期 |
| createTime | datetime | 下单时间 |

---

### 4.2 订单详情

**GET** `/api/v1/tourist/orders/{orderId}`

**Auth:** Bearer Token。只能查看自己的订单。

**响应包含：** 订单基本信息 + 子订单列表 + 每位游客信息 + 电子票凭证 + 退款记录 + 操作时间线。

**响应示例（关键字段）：**
```json
{
  "code": 0,
  "data": {
    "orderId": "ORD202606010001",
    "orderNo": "MTY06010001",
    "totalAmount": "35.00",
    "discountAmount": "5.00",
    "paidAmount": "35.00",
    "status": 2,
    "statusText": "已支付",
    "contactName": "张三",
    "contactPhone": "138****1234",
    "travelDate": "2026-06-01",
    "subOrders": [
      {
        "subOrderId": "SUB001",
        "productName": "绥中长城成人票",
        "priceType": "全价",
        "quantity": 1,
        "unitPrice": "40.00",
        "verifyCount": 0,
        "visitors": [{
          "name": "张*",
          "idNo": "110***********1234",
          "verifyStatus": 0,
          "verifyStatusText": "未核销",
          "voucher": {
            "voucherType": "barcode_128",
            "voucherContent": "https://cdn.example.com/vouchers/SUB001/barcode.png"
          }
        }]
      }
    ],
    "refundRecords": [],
    "timeline": [
      { "time": "2026-05-26 10:00:01", "action": "提交订单", "operator": "user" },
      { "time": "2026-05-26 10:15:00", "action": "支付成功", "operator": "system" }
    ]
  }
}
```

---

### 4.3 获取电子票凭证

**GET** `/api/v1/tourist/orders/{orderId}/voucher`

> 返回订单下所有子订单的电子票凭证（条形码/二维码/编码）。

**Auth:** Bearer Token。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "vouchers": [{
      "subOrderId": "SUB001",
      "visitorName": "张*",
      "voucherType": "barcode_128",
      "voucherContent": "https://cdn.example.com/vouchers/SUB001/barcode.png",
      "voucherCode": "6901234567890",
      "visitDate": "2026-06-01",
      "timeSlot": "08:00-10:00",
      "verifyStatus": 0
    }]
  }
}
```

---

## 5. 年卡

### 5.1 可购买年卡列表

**GET** `/api/v1/tourist/annual-cards`

**响应示例：**
```json
{
  "code": 0,
  "data": [{
    "cardType": "ANNUAL_ADULT",
    "cardName": "成人年卡",
    "price": "200.00",
    "validDays": 365,
    "description": "一年内不限次数入园"
  }]
}
```

### 5.2 购买年卡

**POST** `/api/v1/tourist/annual-cards/purchase`

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| cardType | string | 是 | ANNUAL_ADULT / ANNUAL_SENIOR |
| quantity | int | 是 | 购买数量（可为家人购买） |
| visitors | array | 是 | 持卡人信息（姓名+证件号+AES加密+人脸采集） |

### 5.3 我的年卡

**GET** `/api/v1/tourist/annual-cards/my`

**Auth:** Bearer Token。

**响应示例：**
```json
{
  "code": 0,
  "data": [{
    "cardNo": "AC20260501001",
    "cardType": "成人年卡",
    "holderName": "张三",
    "effectiveDate": "2026-05-26",
    "expireDate": "2027-05-25",
    "remainingDays": 364,
    "faceEnabled": true,
    "status": 1,
    "statusText": "正常"
  }]
}
```

### 5.4 年卡人脸采集

**POST** `/api/v1/tourist/annual-cards/{cardNo}/face-collect`

> 年卡购买后引导用户采集人脸，用于刷脸入园。

**Auth:** Bearer Token。

**Content-Type:** multipart/form-data

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| faceImage | file | 是 | 人脸照片（jpg/png，正面免冠，需活体检测） |

**响应示例：**
```json
{
  "code": 0,
  "data": { "faceImageId": "FACE001", "qualityScore": "0.95", "status": 1, "statusText": "已入库" }
}
```

> 质量分 < 0.8 时返回 "照片质量不合格，请重新拍摄"，并提示具体要求。

### 5.5 年卡挂失

**POST** `/api/v1/tourist/annual-cards/{cardNo}/report-loss`

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| reason | string | 是 | 挂失原因 |

**响应：** `{ "code": 0, "data": { "cardNo": "AC20260501001", "status": 2, "statusText": "已挂失" } }`

### 5.6 年卡补办

**POST** `/api/v1/tourist/annual-cards/{cardNo}/reissue`

> 已挂失年卡可申请补办。原卡作废，生成新卡（新 cardNo），关联同用户+同证件。

**Auth:** Bearer Token。

**响应：**
```json
{
  "code": 0,
  "data": { "newCardNo": "AC20260601002", "oldCardNo": "AC20260501001", "effectiveDate": "2026-05-26", "expireDate": "2027-05-25" }
}
```

### 5.7 年卡预约入园

**POST** `/api/v1/tourist/annual-cards/{cardNo}/reserve`

> 年卡用户预约游览时段（不购票，仅占时段容量）。

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| visitDate | string | 是 | 游览日期 |
| timeSlot | string | 是 | 时段 |

**响应：** `{ "code": 0, "data": { "reservationNo": "RES20260501001", "visitDate": "2026-06-01", "timeSlot": "08:00-10:00" } }`

**GET** `/api/v1/tourist/annual-cards/{cardNo}/reservations` — 我的年卡预约记录

---

## 6. 会员与营销

### 6.1 我的会员信息

**GET** `/api/v1/tourist/member/info`

**Auth:** Bearer Token。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "memberNo": "M20260001",
    "level": "SILVER",
    "levelText": "银卡会员",
    "nextLevel": "GOLD",
    "nextLevelNeed": "500.00",
    "totalPoints": 2500,
    "availablePoints": 1500,
    "totalConsumption": "1250.00",
    "benefits": [
      { "benefitType": "discount", "benefitValue": "9折优惠" },
      { "benefitType": "birthday", "benefitValue": "生日当月双倍积分" }
    ]
  }
}
```

### 6.2a 每日签到

**POST** `/api/v1/tourist/member/sign-in`

**Auth:** Bearer Token。

**响应：** `{ "code": 0, "data": { "points": 10, "consecutiveDays": 7, "bonusPoints": 5 } }`

### 6.2 积分明细

**GET** `/api/v1/tourist/member/points-log`

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |
| scene | string | 否 | consumption / sign_in / interaction / exchange / deduction / lottery |

**响应records字段：**

| 字段 | 说明 |
|------|------|
| points | +100（获得）/-50（消耗） |
| scene | 场景 |
| sceneText | 消费得积分 / 签到 / 积分兑换 / 抵扣 |
| description | 业务描述 |
| createTime | 时间 |

### 6.3 积分兑换

**POST** `/api/v1/tourist/member/points-exchange`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| benefitId | string | 是 | 权益 ID |
| quantity | int | 是 | 兑换数量 |

### 6.4 优惠券列表

**GET** `/api/v1/tourist/coupons`

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | int | 否 | 0未使用(已领取) / 1已使用 / 2已过期 |
| usableFor | string | 否 | 传入 SPU ID，筛选该商品可用的券 |

### 6.5 领取优惠券

**POST** `/api/v1/tourist/coupons/receive`

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| couponId | string | 是 | 券模板 ID |

---

### 6.6 秒杀活动

**GET** `/api/v1/tourist/seckill`

> 进行中/即将开始的秒杀活动列表。

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "records": [{
      "seckillId": "SK001",
      "skuId": "SKU001",
      "skuName": "成人全价票",
      "originalPrice": "45.00",
      "seckillPrice": "19.90",
      "stock": 100,
      "soldCount": 45,
      "perUserLimit": 1,
      "startTime": "2026-05-26 12:00:00",
      "endTime": "2026-05-26 14:00:00",
      "status": 0,
      "statusText": "即将开始",
      "countdown": 3600
    }]
  }
}
```

### 6.7 拼团活动

**GET** `/api/v1/tourist/group-buy`

**POST** `/api/v1/tourist/group-buy/join`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| groupBuyId | string | 是 | 拼团活动 ID |
| groupId | string | 否 | 已有团 ID（参团），不传则自动开新团 |

**GET** `/api/v1/tourist/group-buy/my` — 我参与的拼团列表（含状态: 进行中/成功/失败）

**GET** `/api/v1/tourist/group-buy/{groupId}/status` — 团实例详情（当前人数/成团阈值/倒计时）

### 6.8 全民分销

**POST** `/api/v1/tourist/distribution/apply` — 申请成为分销员

**Auth:** Bearer Token。响应: `{ "code": 0, "data": { "shareCode": "SHARE001", "status": 0, "statusText": "待审核" } }`

**GET** `/api/v1/tourist/distribution/profile` — 我的推广码/链接/佣金汇总

**Auth:** Bearer Token。响应: `{ "shareCode": "SHARE001", "shareLink": "https://...", "totalCommission": "250.00" }`

**GET** `/api/v1/tourist/distribution/commissions` — 佣金明细（含状态：待结算/已结算/已提现）

**POST** `/api/v1/tourist/distribution/withdraw` — 提现申请。`{ "amount": "100.00" }`

---

## 7. 发票

### 7.1 可开票订单

**GET** `/api/v1/tourist/invoices/orders`

**Auth:** Bearer Token。返回已支付/已核销且未开票的订单。

### 7.2 申请开票

**POST** `/api/v1/tourist/invoices/apply`

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| orderIds | array | 是 | 订单 ID 列表 |
| invoiceType | string | 是 | personal / enterprise |
| invoiceTitle | string | 是 | 抬头（个人姓名/企业名称） |
| taxNo | string | 否 | 企业税号（invoiceType=enterprise 必填） |
| email | string | 是 | 接收邮箱 |

**响应示例：**
```json
{
  "code": 0,
  "data": { "applyNo": "INV20260501001", "status": 0, "statusText": "待开具" }
}
```

### 7.3 我的发票列表

**GET** `/api/v1/tourist/invoices`

**Auth:** Bearer Token。

**响应records字段：** applyNo, amount, invoiceType, invoiceTitle, status, statusText, invoiceUrl, createTime。

---

## 8. 导览服务

### 8.1 手绘地图数据

**GET** `/api/v1/tourist/guide/map`

> 无需 Token。返回地图底图 URL + 全部标注点。标注名称/描述由 `?lang=` 参数和 `content_hand_map_i18n` 表驱动。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| lang | string | 否 | 语言，默认 Accept-Language，缺省 zh-CN |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "mapImageUrl": "https://cdn.example.com/maps/suizhong-hand-drawn.jpg",
    "markers": [
      { "id": "M001", "markerType": "scenic", "markerName": "正关台", "longitude": 116.56, "latitude": 40.43, "iconUrl": "...", "description": "绥中关，明代建筑" },
      { "id": "M010", "markerType": "toilet", "markerName": "卫生间", "longitude": 116.55, "latitude": 40.42, "iconUrl": "..." },
      { "id": "M020", "markerType": "parking", "markerName": "停车场", "longitude": 116.54, "latitude": 40.42, "iconUrl": "..." }
    ]
  }
}
```

### 8.2 导览路线

**GET** `/api/v1/tourist/guide/routes`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| routeType | string | 否 | recommend / family / senior / quick |
| lang | string | 否 | 语言，默认 Accept-Language，缺省 zh-CN |

### 8.3 附近服务点

**GET** `/api/v1/tourist/guide/nearby`

> 根据当前坐标返回附近的服务设施（卫生间、餐厅、停车场、缆车站、ATM 等）。无需 Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| longitude | decimal | 是 | 当前经度 |
| latitude | decimal | 是 | 当前纬度 |
| radius | int | 否 | 搜索半径（米），默认 500 |
| markerType | string | 否 | 筛选类型：toilet / restaurant / parking / cable_car / atm / facility。不传返回全部 |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "markers": [
      { "markerName": "南口卫生间", "markerType": "toilet", "distance": 120, "bearing": 30, "longitude": 116.55, "latitude": 40.42 },
      { "markerName": "长城餐厅", "markerType": "restaurant", "distance": 350, "bearing": 345, "longitude": 116.56, "latitude": 40.43 }
    ]
  }
}
```

### 8.4 语音讲解

**GET** `/api/v1/tourist/guide/audio/{markerId}`

> 返回对应景点的语音讲解信息。无需 Token。默认返回请求语言(`?lang=`/`Accept-Language`)对应音频+文字稿。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| lang | string | 否 | 语言，默认 Accept-Language，缺省 zh-CN。未命中 fallback 至 zh-CN |
| listAll | bool | 否 | true 时返回所有语言的 `languages` 数组

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "markerId": "M001",
    "markerName": "正关台",
    "audioUrl": "https://cdn.example.com/audio/m001_zh.mp3",
    "transcript": "正关台是绥中关的...",
    "duration": 180,
    "languages": [
      { "lang": "zh-CN", "audioUrl": "https://cdn.example.com/audio/m001_zh.mp3" },
      { "lang": "en", "audioUrl": "https://cdn.example.com/audio/m001_en.mp3" }
    ]
  }
}
```

---

## 9. 内容与互动

### 9.1 资讯/公告列表

**GET** `/api/v1/tourist/articles`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| category | string | 否 | news / notice / culture / activity |
| lang | string | 否 | 语言，默认 Accept-Language，缺省 zh-CN |

### 9.2 游记/攻略列表

**GET** `/api/v1/tourist/notes`

> **游客端仅展示审核通过的笔记（auditStatus=1）。**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sortBy | string | 否 | hot（按热度）/ new（最新）/ default |
| tags | string | 否 | 标签筛选 |

**响应records字段：** noteId, title, summary(前200字), images[0], authorName, authorAvatar, likeCount, collectCount, tags, publishTime。

### 9.3 笔记详情

**GET** `/api/v1/tourist/notes/{noteId}` — 含完整内容 + 图片 + 互动数据 + 是否已点赞/收藏。

### 9.4 发布笔记

**POST** `/api/v1/tourist/notes`

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| title | string | 是 | 标题 |
| content | string | 是 | 正文（支持富文本） |
| images | array | 是 | 图片 URL 列表 |
| videoUrl | string | 否 | 短视频 URL（mp4 格式，上限 60 秒） |
| tags | array | 否 | 标签 |

**响应：** `{ "code": 0, "data": { "noteId": "N001", "auditStatus": 0, "auditStatusText": "待审核" } }`

### 9.5 互动操作

**POST** `/api/v1/tourist/interactions`

**Auth:** Bearer Token。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| targetType | string | 是 | note / article / audio_guide |
| targetId | string | 是 | 目标 ID |
| interactionType | string | 是 | like / collect / share |

> 同一用户对同一目标重复点击视为取消。

---

## 10. AI 问答

### 10.1 AI 问答对话

**POST** `/api/v1/tourist/ai/chat`

> 发送文字/语音问题到 AI 问答助手。7x24 自动回复，无法回答时转人工。

**请求参数（Body）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| question | string | 否 | 文字问题（与 voiceUrl 二选一） |
| voiceUrl | string | 否 | 语音文件 URL（先通过 `/common/upload` 上传获取 URL，AI 服务内部做语音识别转文字） |
| sessionId | string | 否 | 会话 ID（首次不传，后续传入实现连续对话） |
| lang | string | 否 | 语言 |

**响应示例（AI 回复）：**
```json
{
  "code": 0,
  "data": {
    "sessionId": "AI_SESSION_001",
    "answer": "绥中长城位于北京市怀柔区，距北京城区约73公里。开放时间为每天08:00-17:00。成人票价40元。",
    "answerType": "ai",
    "references": [
      { "title": "景区开放时间", "url": "..." },
      { "title": "门票价格说明", "url": "..." }
    ]
  }
}
```

**响应示例（转人工）：**
```json
{
  "code": 0,
  "data": {
    "sessionId": "AI_SESSION_001",
    "answer": "您的问题比较复杂，已为您转接人工客服，请稍候...",
    "answerType": "transfer",
    "waitTime": 60
  }
}
```

**错误码：**

| code | 说明 |
|------|------|
| AI_001 | AI 服务暂时不可用 |
| AI_002 | 语音识别失败 |

### 12.4 通知收件箱

**GET** `/api/v1/tourist/notifications`

**Auth:** Bearer Token。查看我的历史通知消息（支付成功/核销提醒/租赁逾期等）。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**PUT** `/api/v1/tourist/notifications/{id}/read` — 标记已读

---

## 12. 用户信息管理

### 12.1 个人信息

**GET** `/api/v1/tourist/user/profile`

**Auth:** Bearer Token。

**PUT** `/api/v1/tourist/user/profile` — 修改个人信息（手机号需验证码）。

### 12.2 出行人管理

**GET** `/api/v1/tourist/user/travelers`

**Auth:** Bearer Token。管理常用出行人，下单时快速选择。

**POST** `/api/v1/tourist/user/travelers` — 新增

**PUT** `/api/v1/tourist/user/travelers/{id}` — 编辑

**DELETE** `/api/v1/tourist/user/travelers/{id}` — 删除

### 12.3 NFC 证件识别

**POST** `/api/v1/tourist/user/ocr-id`

> 通过 NFC 或拍照识别身份证件信息，自动填入出行人表单。

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| imageUrl | string | 是 | 证件照片 OSS URL |
| idType | string | 是 | ID_CARD / PASSPORT / HK_MO / TAIWAN |

**响应示例：**
```json
{
  "code": 0,
  "data": {
    "name": "张三",
    "idType": "ID_CARD",
    "idNo": "110***********1234",
    "gender": "M",
    "nationality": "CHN",
    "birthDate": "1996-01-15"
  }
}
```
