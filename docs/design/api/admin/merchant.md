# 商户管理 — `/api/v1/admin/merchants`

### 7.1 商户 CRUD

**GET** `/api/v1/admin/merchants`

| 参数 | 必填 | 说明 |
|------|------|------|
| status | 否 | 0待审核/1正常/2禁用/3驳回 |
| category | 否 | 业态筛选 |
| areaId | 否 | 景区筛选 |
| keyword | 否 | 商户名称/编码/联系人手机 |
| createStart | 否 | 入驻起始 |
| createEnd | 否 | 入驻截止 |
| pageNo | 否 | 默认 1 |
| pageSize | 否 | 默认 20 |

**响应records字段：** merchantId, merchantCode, merchantName, areaId, areaName, category, categoryText, contactName, contactPhone(脱敏), status, statusText, shopCount, createTime。

**POST** `/api/v1/admin/merchants` — 创建商户 `{ "merchantCode":"...", "merchantName":"...", "areaId":"...", "contactName":"...", "contactPhone":"...", "businessLicense":"...", "category":"..." }`

**GET** `/api/v1/admin/merchants/{merchantId}` — 详情（含完整 contact/area/营业执照 + 店铺列表 + 佣金配置 + 审核记录）

**PUT** `/api/v1/admin/merchants/{merchantId}` — 编辑商户信息

**DELETE** `/api/v1/admin/merchants/{merchantId}` — 软删除（del_flag=1）

**POST** `/api/v1/admin/merchants/{merchantId}/audit` — 审核 `{ "auditResult": 1|2, "auditRemark": "..." }`。响应: `{ "merchantId": "...", "status": 1, "statusText": "正常" }`（审核通过后自动创建商户 sys_user 登录账号）

**PUT** `/api/v1/admin/merchants/{merchantId}/status` — 启停 `{ "status": 1|2, "remark": "违规内容" }`

**POST** `/api/v1/admin/merchants/batch-audit` — 批量审核 `{ "merchantIds": [...], "auditResult": 1|2 }`

### 7.2 店铺管理

**GET** `/api/v1/admin/merchants/{merchantId}/shops` — 商户店铺列表

**POST** `/api/v1/admin/merchants/{merchantId}/shops` — 创建店铺 `{ "shopName":"...", "shopLogo":"...", "shopDesc":"...", "areaId":"..." }`

**PUT** `/api/v1/admin/merchants/{merchantId}/shops/{shopId}` — 编辑店铺

**PUT** `/api/v1/admin/merchants/{merchantId}/shops/{shopId}/status` — 启停 `{ "status": 0|1 }`

**GET** `/api/v1/admin/merchants/{merchantId}/shops/{shopId}/page` — 预览店铺装修页面（shop_page JSON 渲染预览）

**POST** `/api/v1/admin/merchants/{merchantId}/shops/{shopId}/page/audit` — 审核发布 `{ "result": 1|2, "remark": "..." }`。通过后 `is_published=1`

---

### 7.3 商户商品审核

**GET** `/api/v1/admin/merchants/products`

| 参数 | 必填 | 说明 |
|------|------|------|
| auditStatus | 否 | 0待审核/1通过/2驳回 |
| merchantId | 否 | 商户筛选 |
| categoryCode | 否 | 业态筛选 |
| keyword | 否 | 名称/编码 |
| submitStart | 否 | 提交起始 |
| submitEnd | 否 | 提交截止 |
| pageNo | 否 | 默认 1 |
| pageSize | 否 | 默认 20 |

**响应records字段：** spuId, spuName, merchantId, merchantName, categoryCode, minPrice, auditStatus, auditStatusText, auditRemark, submitTime, createTime。

**GET** `/api/v1/admin/merchants/products/{productId}` — 商品审核详情（含完整 SPU/SKU/图片/描述/购买须知/退改规则）

**POST** `/api/v1/admin/merchants/products/{productId}/audit` — 审核 `{ "auditResult": 1|2, "auditRemark": "..." }`

**POST** `/api/v1/admin/merchants/products/batch-audit` — 批量审核

---

### 7.4 佣金管理

**GET** `/api/v1/admin/merchants/{merchantId}/commissions` — 佣金配置列表

**POST** `/api/v1/admin/merchants/{merchantId}/commission` — 新增 `{ "categoryCode":"...", "commissionRate":"0.1000", "effectiveDate":"..." }`。响应: `{ "commissionId": "..." }`

**PUT** `/api/v1/admin/merchants/{merchantId}/commission/{id}` — 编辑佣金比例

**PUT** `/api/v1/admin/merchants/{merchantId}/commission/{id}/invalid` — 失效（status→0）

---

### 7.5 商户经营看板

**GET** `/api/v1/admin/merchants/{merchantId}/dashboard`

**响应：** `{ "code": 0, "data": { "totalOrders": 1240, "totalRevenue": "98500.00", "totalCommission": "9850.00", "totalSettle": "88650.00", "productCount": 45, "monthlyTrend": [...] } }`

---

