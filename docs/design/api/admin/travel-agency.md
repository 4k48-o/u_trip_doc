# 旅行社管理 — `/api/v1/admin/travel-agencies`

### 14.1 旅行社 CRUD

**GET** `/api/v1/admin/travel-agencies`

| 参数 | 必填 | 说明 |
|------|------|------|
| auditStatus | 否 | 0待审核/1通过/2驳回 |
| groupType | 否 | key/normal/society/restaurant（固定枚举） |
| status | 否 | 0禁用/1启用 |
| keyword | 否 | 名称/编码/联系人 |
| pageNo | 否 | 默认 1 |
| pageSize | 否 | 默认 20 |

**响应records字段：** agencyId, agencyCode, agencyName, groupType, groupTypeText, contactName, contactPhone(脱敏), auditStatus, auditStatusText, status, statusText, operatorCount, createTime。

**POST** `/api/v1/admin/travel-agencies` — 创建 `{ "agencyCode":"...", "agencyName":"...", "groupType":"key", "contactName":"...", "contactPhone":"...", "businessLicense":"...", "qualificationFiles":"[\"url1\",\"url2\"]" }`

**GET** `/api/v1/admin/travel-agencies/{agencyId}` — 详情（含资质文件/营业执照/账户信息/操作人员列表/合同）

**PUT** `/api/v1/admin/travel-agencies/{agencyId}` — 编辑

**DELETE** `/api/v1/admin/travel-agencies/{agencyId}` — 删除

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/status` — 启停 `{ "status": 0|1, "reason": "..." }`

---

### 14.2 审核与分组

**POST** `/api/v1/admin/travel-agencies/{agencyId}/audit` — 审核 `{ "auditResult": 1|2, "auditRemark": "..." }`。通过后自动创建 travel_agency_account + 默认 operator 账号 + 发送通知。

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/group-type` — 设置分组 `{ "groupType": "key" }`。分组变更后定价/授信需手动调整。

---

### 14.3 操作人员管理

**GET/POST** `/api/v1/admin/travel-agencies/{agencyId}/operators` — 列表/新增 `{ "userId":"...", "role":"order|verify|finance|admin", "mobile":"..." }`

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/operators/{operatorId}` — 编辑角色

**DELETE** `/api/v1/admin/travel-agencies/{agencyId}/operators/{operatorId}` — 移除

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/operators/{operatorId}/status` — 启停

---

### 14.4 账户与钱包

**GET** `/api/v1/admin/travel-agencies/{agencyId}/accounts` — 账户信息

**响应：** `{ "balance":"25000.00", "creditLimit":"50000.00", "usedCredit":"7000.00", "availableCredit":"43000.00", "totalRecharge":"30000.00", "totalConsumption":"12000.00", "payMethod":"credit", "status": 1, "statusText":"正常" }`

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/account/status` — 冻结/解冻 `{ "status": 0|1, "reason": "..." }`

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/credit` — 手动调授信 `{ "creditLimit": "80000.00", "reason": "旺季追加" }`（记录 credit_log）

**POST** `/api/v1/admin/travel-agencies/{agencyId}/account/recharge` — 手动充值 `{ "amount":"10000.00", "payMethod":"transfer", "remark":"..." }`

**POST** `/api/v1/admin/travel-agencies/{agencyId}/account/adjust` — 调账 `{ "amount":"±100.00", "type":"debit|credit", "reason":"差错更正" }`

**GET** `/api/v1/admin/travel-agencies/{agencyId}/account/credit-log` — 授信调整历史

**GET** `/api/v1/admin/travel-agencies/credit-applies?status=0` — 授信申请列表（审批）

**POST** `/api/v1/admin/travel-agencies/{agencyId}/credit-apply/{applyId}/audit` — 审批 `{ "result": 1|2, "remark": "..." }`

---

### 14.5 账户流水

**GET** `/api/v1/admin/travel-agencies/{agencyId}/transactions?transactionType=X&startDate=Y&endDate=Z&pageNo=1` — 流水

**响应records字段：** transactionNo, transactionType(recharge/deduct/refund/adjust), transactionTypeText, amount(±), balanceAfter, refId, remark, createTime。

---

### 14.6 订单与批量导入

**GET** `/api/v1/admin/travel-agencies/{agencyId}/orders?status=X&travelDate=Y&pageNo=1` — 旅行社订单列表

**GET** `/api/v1/admin/travel-agencies/{agencyId}/batch-orders?status=X` — 批量导入列表

**GET** `/api/v1/admin/travel-agencies/batch-orders/{batchNo}` — 批次详情（含成功/失败明细）

**POST** `/api/v1/admin/travel-agencies/batch-orders/{batchNo}/retry` — 失败重试

---

### 14.7 折扣定价

**GET** `/api/v1/admin/travel-agencies/{agencyId}/group-prices` — 查看定价

**POST** `/api/v1/admin/travel-agencies/{agencyId}/group-prices` — 批量设置 `{ "prices": [{"skuId":"SKU001","price":"35.00"}] }`。返回: `{ "successCount": 5, "failCount": 0 }`

**PUT/DELETE** `/api/v1/admin/travel-agencies/{agencyId}/group-prices/{priceId}` — 修改/删除

> group-prices 定价在 product_channel_price（channel=agent）基础之上生效，group-prices 优先级更高。两者不叠加，group-prices 覆盖 channel=agent 价。

---

### 14.8 合同与统计

**POST/GET** `/api/v1/admin/travel-agencies/{agencyId}/contract` — 上传/查看合同

**GET** `/api/v1/admin/travel-agencies/{agencyId}/dashboard` — 旅行社统计看板 `{ "totalOrders": 320, "totalAmount": "128000.00", "creditUsageRate": "70%", "batchSuccessRate": "95%" }`

---

