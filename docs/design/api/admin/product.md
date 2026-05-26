# 商品管理 — `/api/v1/admin/products`

### 2.1 商品列表

**GET** `/api/v1/admin/products`

| 参数 | 必填 | 说明 |
|------|------|------|
| categoryCode | 否 | 业态筛选 |
| scenicSpotId | 否 | 按景点筛选 |
| merchantId | 否 | 按商户筛选 |
| passengerType | 否 | Adult/Child/Senior/Youth/Infant |
| primaryLanguage | 否 | 按原始录入语言筛选 |
| keyword | 否 | SPU 名称/编码 |
| status | 否 | 0下架/1上架 |
| sortBy | 否 | price_asc/price_desc/sales_desc/create_desc，默认 create_desc |
| pageNo | 否 | 默认 1 |
| pageSize | 否 | 默认 20 |

**响应records关键字段：** spuId, spuCode, spuName, categoryCode, categoryCodes(JSON数组), categoryName, minPrice, maxPrice, status, statusText, soldCount, primaryLanguage, tags, createTime, skuCount。

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
  "categoryCodes": "[\"ATTRACTION_TICKET\"]",
  "primaryLanguage": "zh-CN",
  "reference": "TICKET_REF_001",
  "metaData": "{\"extInfo\":\"custom\"}",
  "serviceLanguages": "[\"zh-CN\",\"en\",\"ja\",\"ko\"]",
  "guestInfoType": "PER_PERSON",
  "guestInfoCodes": "[\"GUEST_NAME\",\"ID_CARD\",\"COUNTRY\"]",
  "paymentConfirmationTime": 30,
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
    "minBuy": 1,
    "maxBuyPerOrder": 5,
    "unitPax": 1,
    "companionRequired": false,
    "customCode": "ADULT_FULL",
    "passengerType": "Adult",
    "netPriceCurrency": "CNY",
    "retailPriceCurrency": "CNY",
    "billingType": "one_time",
    "validDays": 1,
    "ageLimitMin": 19,
    "ageLimitMax": 59
  }, {
    "skuCode": "TICKET_ADULT_DISCOUNT",
    "skuName": "老人优惠票",
    "specDesc": "{\"ageMin\": 60, \"ageMax\": null}",
    "priceType": "DISCOUNT",
    "originalPrice": "25.00",
    "sellPrice": "20.00",
    "costPrice": "0.00",
    "needRealName": true,
    "minBuy": 1,
    "maxBuyPerOrder": 5,
    "unitPax": 1,
    "companionRequired": false,
    "passengerType": "Senior",
    "netPriceCurrency": "CNY",
    "retailPriceCurrency": "CNY",
    "validDays": 1,
    "ageLimitMin": 60,
    "ageLimitMax": null
  }]
}
```

> **specDesc 字段 schema：** SKU 的 `specDesc` 为 JSON 字符串，支持 `ageMin`（int）、`ageMax`（int/null）、`idType`（string数组如 `["ID_CARD","PASSPORT"]`）等规格维度。由前端自由组合，后端透传不做额外校验。

### 2.3 编辑商品

**PUT** `/api/v1/admin/products/{spuId}` — 编辑 SPU 基本信息（部分更新，传什么改什么）

**PUT** `/api/v1/admin/products/{spuId}/sku/{skuId}` — 编辑 SKU（部分更新）

**DELETE** `/api/v1/admin/products/{spuId}/sku/{skuId}` — 删除 SKU（仅当该 SKU 无未核销订单时允许）

### 2.4 组合产品管理

**GET** `/api/v1/admin/products/combo` — 组合产品列表 `?keyword=X&comboType=Y&status=Z`

**GET** `/api/v1/admin/products/combo/{comboId}` — 组合产品详情

**POST** `/api/v1/admin/products/combo` — 创建组合产品

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

**PUT** `/api/v1/admin/products/combo/{comboId}` — 编辑组合产品

**PUT** `/api/v1/admin/products/combo/{comboId}/status` — 启停 `{ "status": 0 }`

**DELETE** `/api/v1/admin/products/combo/{comboId}` — 删除组合（仅无关联订单时允许）

> 子 SKU 下架时，包含该 SKU 的组合产品自动标记为 inactive，并在管理端预警提示。

---

### 2.5 价格日历批量设置

**GET** `/api/v1/admin/products/{spuId}/price-calendar?skuId=X&startDate=Y&endDate=Z` — 查询已有日历

**PUT** `/api/v1/admin/products/{spuId}/price-calendar`

> 为指定 SKU 批量设置未来日期的价格和库存。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| skuId | string | 是 | SKU ID |
| currency | string | 否 | 币种 (ISO 4217)，默认 CNY |
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

**GET** `/api/v1/admin/products/{spuId}/languages` — 查询所有已录入语言内容

**POST** `/api/v1/admin/products/{spuId}/languages` — 设置 SPU 级多语言

| 字段 | 必填 | 说明 |
|------|------|------|
| languageCode | 是 | zh-CN / en / ja / ko / ru / fr / es / ar |
| title | 否 | 多语言名称 |
| description | 否 | 多语言描述 |
| notice | 否 | 多语言须知 |
| refundPolicy | 否 | 多语言退改规则 |

> **结构化子表多语言：** inclusions/exclusions/highlights/how-to-use 已拆分独立子表，翻译通过子表记录 ID 关联：

**POST** `/api/v1/admin/products/{spuId}/inclusions/{itemId}/languages` — `{ "languageCode":"en", "content":"Adult ticket x1" }`

**GET** `/api/v1/admin/products/{spuId}/inclusions/{itemId}/languages` — 查指定条目的翻译

> exclusions/highlights/how-to-use 同理。

### 2.8 选项/时段管理

**GET** `/api/v1/admin/products/{spuId}/options` — 查询当前选项配置

**PUT** `/api/v1/admin/products/{spuId}/options`

| 字段 | 类型 | 说明 |
|------|------|------|
| optionCode | string | 选项编码 |
| optionType | string | option / time_slot |
| optionStatus | string | active / inactive（携程 optionStatus） |
| optionDesc | string | 套餐描述（≤200字符） |
| optionBookingCutoffTime | string | 套餐级提前预订时间（JSON） |
| primaryLanguage | string | value_name 对应的语言代码 |
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
| newTotalStock | int | 否 | 新总库存（与 adjustUsedStock 互斥，二选一必填） |
| adjustUsedStock | int | 否 | 调整已售（与 newTotalStock 互斥，二选一必填） |
| adjustReason | string | 是 | 调整原因 |

---

### 2.10 商品批量导入导出

**POST** `/api/v1/admin/products/batch-import` — Excel 批量导入（multipart/form-data）

> 模板列：`序号 | SPU编码 | SPU名称 | 业态分类 | SKU编码 | SKU名称 | 价格类型(FULL/DISCOUNT/FREE) | 人群类型(Adult/Child/Senior/Youth/Infant/Student/Traveler/Customized) | 原价 | 售价 | 成本价 | 币种(CNY/USD/...) | 最小份数 | 最大限购 | 每份人数 | 年龄下限 | 年龄上限 | 是否实名 | 有效期 | 是否必组合`

> **错误处理：** 部分行校验失败不阻塞成功行（逐行处理），失败明细通过异步通知返回。导入结果通过 `GET /admin/products/batch-import/{batchId}` 查询。

**GET** `/api/v1/admin/products/export?categoryCode=X&status=Y&startDate=A&endDate=B` — 导出全量商品 Excel

### 2.11 结构化子表管理

> 以下端点管理 product_spu 的关联子表，完整 schema 如下。

**行程 CRUD：** `/api/v1/admin/products/{spuId}/itinerary`

| 方法 | 说明 | 请求体 |
|------|------|--------|
| GET | 查行程（含 day/item/food/accommodation/start/end 嵌套） | — |
| POST | 创建行程 | `{ "startType":"Meet_at_Start_Point", "endType":"End_on_the_Spot", "days":[{ "dayNumber":1, "items":[{ "time":"09:00","poiId":"...","feeInclusions":"Exclude" }], "food":[{ "time":"12:00","mealType":"Lunch","feeInclusions":"Exclude" }], "accommodation":[{"poiId":"...","description":"..."}] }], "starts":[{ "startTime":"08:00","startType":"Meet_at_Start_Point","poiId":"...","description":"..." }], "ends":[{ "endTime":"18:00","endType":"End_on_the_Spot","poiId":"...","description":"..." }] }` |
| PUT | 编辑行程 | 同 POST |
| DELETE | 删除行程 | — |

**核销地点：** `/api/v1/admin/products/{spuId}/redemption-locations`

| 方法 | 请求体 |
|------|--------|
| GET | — |
| POST | `{ "name":"南口售票处", "addressDetail":"...", "longitude":116.56, "latitude":40.43, "description":"请在此换票" }` |
| PUT `.../{id}` | 同 POST |
| DELETE `.../{id}` | — |

**预订问题：** `/api/v1/admin/products/{spuId}/booking-questions`

| 方法 | 请求体 |
|------|--------|
| GET | —（返回 question + answers 嵌套） |
| POST | `{ "code":"Q001", "name":"请选择语言", "answerType":"Single_Selection", "answers":[{ "code":"A1","name":"中文导游" },{ "code":"A2","name":"English guide" }] }` |
| PUT `.../{id}` | 同 POST |
| DELETE `.../{id}` | — |

**标签：** `/api/v1/admin/products/{spuId}/tags`

| 方法 |
|------|
| GET（返回已关联标签列表） |
| PUT（全量替换关联 `{ "tagIds": ["T001","T002"] }`） |

**目的地/出发地：** `/api/v1/admin/products/{spuId}/destination` + `.../departure`

| 方法 | 请求体 |
|------|--------|
| GET/POST/PUT/DELETE | `{ "supplierId":"...", "googlePlaceId":"...", "name":"北京" }` |

**列表项 CRUD：**

| 端点 | 方法 | 请求体 |
|------|------|--------|
| `/products/{spuId}/inclusions` | GET/POST/PUT/DELETE | `{ "content":"成人门票1张", "sortNo":1 }` |
| `/products/{spuId}/exclusions` | GET/POST/PUT/DELETE | `{ "content":"缆车费用", "sortNo":1 }` |
| `/products/{spuId}/highlights` | GET/POST/PUT/DELETE | `{ "content":"世界文化遗产", "sortNo":1 }` |
| `/products/{spuId}/how-to-use` | GET/POST/PUT/DELETE | `{ "content":"凭二维码扫码入园", "sortNo":1 }` |

> inclusion ≤500字符×20条，highlight_item ≤200字符×3条，exclusion/how_to_use ≤500字符×20条。

---

