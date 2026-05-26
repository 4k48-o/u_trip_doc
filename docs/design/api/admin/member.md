# 会员管理 — `/api/v1/admin/members`

### 9.1 会员列表与详情

**GET** `/api/v1/admin/members`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| level | string | 否 | BRONZE/SILVER/GOLD/PLATINUM |
| tag | string | 否 | 标签筛选 |
| status | int | 否 | 1正常/0冻结 |
| memberNo | string | 否 | 会员编号精确搜索 |
| keyword | string | 否 | 姓名/手机号 |
| consumptionMin | decimal | 否 | 最低累计消费 |
| consumptionMax | decimal | 否 | 最高累计消费 |
| registerStart | string | 否 | 注册起始 |
| registerEnd | string | 否 | 注册截止 |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**响应records字段：** userId, memberNo, userName, phone(脱敏), level, levelText, totalPoints, availablePoints, totalConsumption, tags, status, statusText, birthday, createTime。

**GET** `/api/v1/admin/members/{userId}` — 详情（含等级/积分/消费/标签/权益/最近订单列表）

**PUT** `/api/v1/admin/members/{userId}/status` — 冻结/解冻 `{ "status": 0|1, "reason": "违规刷单" }`

**PUT** `/api/v1/admin/members/{userId}/level` — 手动调整等级 `{ "level": "GOLD", "reason": "客服补偿" }`（写入 member_level_log）

**DELETE** `/api/v1/admin/members/{userId}` — 注销（GDPR合规，del_flag=1）

**GET** `/api/v1/admin/members/{userId}/orders` — 会员历史订单（分页，按 orderType/status 筛选）

**GET** `/api/v1/admin/members/export` — 导出 Excel

---

### 9.2 积分管理

**GET** `/api/v1/admin/members/{userId}/points-log?scene=X&startDate=Y&endDate=Z&pageNo=1` — 积分明细

**响应records字段：** id, points(+/-), scene, sceneText, refId, description, expireDate, createTime。

**POST** `/api/v1/admin/members/{userId}/adjust-points` — 手动调整 `{ "points": 100, "scene": "manual", "description": "投诉补偿", "expireDate": "2026-12-31" }`

**GET** `/api/v1/admin/members/points-expiring?days=30` — 即将到期积分提醒列表

---

### 9.3 标签管理

**POST** `/api/v1/admin/members/{userId}/tags` — 添加标签 `{ "tag": "亲子游" }`

**DELETE** `/api/v1/admin/members/{userId}/tags/{tag}` — 移除标签

**GET** `/api/v1/admin/members/tags/dict` — 预定义标签字典（防止标签混乱）

---

### 9.4 定向发券

**POST** `/api/v1/admin/members/{userId}/send-coupon` — 单人发券 `{ "couponId": "CP001", "count": 1, "customExpireDays": 30 }`

**POST** `/api/v1/admin/members/batch-send-coupon` — 批量发券 `{ "couponId": "CP001", "targetType": "level"|"tag"|"userIds", "levels":["GOLD"], "tag":"VIP", "userIds":["..."], "countPerUser": 1 }`。响应: `{ "sentCount": 50, "failCount": 2 }`

---

### 9.5 权益配置

**GET/POST** `/api/v1/admin/members/benefits` — 权益配置 `{ "level":"GOLD", "benefitType":"discount", "benefitValue":"{\"rate\":0.9}", "description":"9折优惠" }`

| 权益类型 | 说明 |
|----------|------|
| discount | 等级折扣率 |
| birthday | 生日礼遇 |
| priority | 优先购 |
| skip_queue | 免排队 |
| exclusive_service | 专属客服 |

---

### 9.6 黑名单管理

**GET** `/api/v1/admin/members/blacklist?type=X&status=Y&pageNo=1` — 黑名单列表

**POST** `/api/v1/admin/members/{userId}/blacklist` — 加入黑名单 `{ "type":"purchase"|"entry"|"marketing"|"all", "reason":"黄牛刷票", "endTime":"2027-06-01" }`

**PUT** `/api/v1/admin/members/{userId}/blacklist/remove` — 移出黑名单 `{ "reason": "申诉通过" }`

---

### 9.7 会员统计看板

**GET** `/api/v1/admin/members/dashboard`

**响应：** `{ "levelStats": [{ "level":"GOLD", "count":1200, "ratio":"15%" }], "newDaily": 45, "pointsSummary": { "totalEarned": 50000, "totalUsed": 35000 }, "tagDistribution": [{ "tag":"亲子游", "count":800 }] }`

---

