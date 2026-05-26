# 营销管理 — `/api/v1/admin/marketing`

### 8.1 优惠券模板

**GET** `/api/v1/admin/marketing/coupons?status=X&couponType=Y&keyword=Z&pageNo=1&pageSize=20` — 列表

**POST** `/api/v1/admin/marketing/coupons`
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| couponCode | string | 是 | 模板编码 |
| couponName | string | 是 | 券名称 |
| couponType | string | 是 | discount / fixed / cash |
| couponValue | decimal(10,2) | 是 | 面值/折扣率 |
| minAmount | decimal(10,2) | 是 | 使用门槛 |
| totalCount | int | 是 | 发放总量 |
| perUserLimit | int | 是 | 每人限领 |
| productScope | string | 是 | all / category / product |
| productIds | array | 否 | 限定商品 ID |
| effectiveDays | int | 是 | 领取后有效天数 |
| startTime | datetime | 是 | 活动开始 |
| endTime | datetime | 是 | 活动结束 |
| status | int | 是 | 0下架/1上架/2已过期 |

**GET** `/api/v1/admin/marketing/coupons/{couponId}` — 详情（含 usedCount/usageRate）

**PUT** `/api/v1/admin/marketing/coupons/{couponId}` — 编辑

**DELETE** `/api/v1/admin/marketing/coupons/{couponId}` — 删除

**PUT** `/api/v1/admin/marketing/coupons/{couponId}/status` — 状态变更 `{ "status": 0|1|2 }`

**POST** `/api/v1/admin/marketing/coupons/{couponId}/publish` — 发放 `{ "targetType": "all"|"level"|"userIds", "levels": ["SILVER"], "userIds": ["..."] }`。响应: `{ "sentCount": 500, "failCount": 0 }`

**GET** `/api/v1/admin/marketing/coupons/{couponId}/usage?status=X&pageNo=1` — 使用记录列表（含 userName/userPhone/orderNo/couponValue/receiveTime/useTime）

**GET** `/api/v1/admin/marketing/coupons/{couponId}/stats` — 统计（发放量/领取量/使用量/核销率）

**POST/PUT/GET** `/api/v1/admin/marketing/coupons/{couponId}/languages` — 优惠券多语言管理

---

### 8.2 秒杀活动

**GET** `/api/v1/admin/marketing/seckill?status=X&keyword=Y&pageNo=1` — 列表

**POST** `/api/v1/admin/marketing/seckill`
| 字段 | 必填 | 说明 |
|------|------|------|
| seckillCode | 是 | 编码 |
| skuId | 是 | 商品 SKU |
| seckillPrice | 是 | 秒杀价（decimal） |
| stock | 是 | 秒杀库存（独立管理） |
| perUserLimit | 是 | 每人限购 |
| startTime | 是 | 开始时间 |
| endTime | 是 | 结束时间 |

**GET** `/api/v1/admin/marketing/seckill/{id}` — 详情（含 soldCount 实时进度）

**PUT** `/api/v1/admin/marketing/seckill/{id}` — 编辑

**DELETE** `/api/v1/admin/marketing/seckill/{id}` — 删除

**PUT** `/api/v1/admin/marketing/seckill/{id}/status` — 状态 `{ "status": 0|1|2 }`

**PUT** `/api/v1/admin/marketing/seckill/{id}/stock` — 调整库存 `{ "adjustStock": +100 }`

---

### 8.3 拼团活动

**GET** `/api/v1/admin/marketing/group-buy?status=X&keyword=Y&pageNo=1` — 列表

**POST** `/api/v1/admin/marketing/group-buy`
| 字段 | 必填 | 说明 |
|------|------|------|
| groupCode | 是 | 编码 |
| skuId | 是 | 商品 SKU |
| groupPrice | 是 | 拼团价 |
| minCount | 是 | 成团人数 |
| expireHours | 是 | 成团时限（小时） |
| startTime | 是 | 活动开始 |
| endTime | 是 | 活动结束 |

**GET** `/api/v1/admin/marketing/group-buy/{id}` — 详情

**PUT/DELETE** `/api/v1/admin/marketing/group-buy/{id}` — 编辑/删除

**PUT** `/api/v1/admin/marketing/group-buy/{id}/status` — 状态 `{ "status": 0|1|2 }`

**GET** `/api/v1/admin/marketing/group-instances?groupBuyId=X&status=Y&pageNo=1` — 拼团实例列表

**GET** `/api/v1/admin/marketing/group-instances/{groupId}` — 团详情（含成员列表: userName/avatar/joinTime/orderNo）

**PUT** `/api/v1/admin/marketing/group-instances/{groupId}/refund` — 手动强退（异常团/刷单）

---

### 8.4 全民分销

**GET** `/api/v1/admin/marketing/distributions?status=X&keyword=Y&pageNo=1` — 分销员列表

**GET** `/api/v1/admin/marketing/distributions/{id}` — 详情（shareCode/shareLink/totalCommission/orderCount）

**POST** `/api/v1/admin/marketing/distributions/{id}/audit` — 审核 `{ "result": 1|2 }`

**PUT** `/api/v1/admin/marketing/distributions/{id}/status` — 启停

**GET** `/api/v1/admin/marketing/commissions?status=X&distributorId=Y&startDate=A&endDate=B&pageNo=1` — 佣金列表

**GET** `/api/v1/admin/marketing/commissions/{id}` — 详情

**POST** `/api/v1/admin/marketing/commissions/settle` — 批量结算 `{ "ids": [...], "status": 1|2 }`

**GET** `/api/v1/admin/marketing/dashboard` — 分销看板（top分销员/总佣金/转化率/订单来源分布）

**GET** `/api/v1/admin/marketing/export` — 导出活动数据/佣金记录

### 8.5 商品关联视图

**GET** `/api/v1/admin/marketing/product/{skuId}/activities` — 查看某商品关联的所有营销活动（含秒杀/拼团/优惠券，叠加冲突检测）

**响应：** `{ "code": 0, "data": { "skuId": "SKU001", "seckills": [{ "id":"...", "price":"19.90", "status":1 }], "groupBuys": [{ "id":"...", "groupPrice":"30.00", "minCount":3 }], "coupons": [{ "couponId":"CP001", "couponName":"满100减20", "status":1 }], "conflicts": [{ "message": "秒杀与优惠券不可叠加使用" }] } }`

---

