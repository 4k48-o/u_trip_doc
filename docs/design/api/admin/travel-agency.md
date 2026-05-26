# 旅行社管理 — `/api/v1/admin/travel-agencies`

---

### 14.1 旅行社 CRUD

**GET** `/api/v1/admin/travel-agencies`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| auditStatus | int | 否 | 0待审核/1通过/2驳回 |
| groupType | string | 否 | key/normal/society/restaurant（固定枚举） |
| status | int | 否 | 0禁用/1启用 |
| keyword | string | 否 | 名称/编码/联系人 |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**响应records字段：** agencyId, agencyCode, agencyName, groupType, groupTypeText, contactName, contactPhone(脱敏), auditStatus, auditStatusText, status, statusText, operatorCount, createTime。

---

**POST** `/api/v1/admin/travel-agencies` — 创建旅行社

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| agencyCode | string | 是 | 旅行社编码 |
| agencyName | string | 是 | 旅行社名称 |
| groupType | string | 是 | key/normal/society/restaurant |
| contactName | string | 是 | 联系人 |
| contactPhone | string | 是 | 联系电话 |
| businessLicense | string | 是 | 营业执照 URL |
| qualificationFiles | array | 否 | 资质文件 URL 列表 |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001", "agencyCode": "..." } }`

---

**GET** `/api/v1/admin/travel-agencies/{agencyId}` — 旅行社详情

**响应：**
```json
{
  "code": 0,
  "data": {
    "agencyId": "AG001",
    "agencyCode": "AG_CODE_001",
    "agencyName": "中国国旅",
    "groupType": "key",
    "groupTypeText": "重点旅行社",
    "contactName": "李四",
    "contactPhone": "139****5678",
    "businessLicense": "https://cdn.example.com/license/ag001.jpg",
    "qualificationFiles": ["https://cdn.example.com/qual/ag001_1.jpg", "..."],
    "auditStatus": 1,
    "auditStatusText": "已通过",
    "status": 1,
    "statusText": "启用",
    "account": { "balance": "25000.00", "creditLimit": "50000.00", "usedCredit": "7000.00" },
    "operators": [{ "operatorId": "OP001", "userName": "王五", "role": "order", "status": 1 }],
    "contract": { "contractUrl": "https://...", "uploadTime": "2026-05-26" },
    "createTime": "2026-01-15"
  }
}
```

---

**PUT** `/api/v1/admin/travel-agencies/{agencyId}` — 编辑旅行社（部分更新）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| agencyName | string | 否 | 旅行社名称 |
| contactName | string | 否 | 联系人 |
| contactPhone | string | 否 | 联系电话 |
| businessLicense | string | 否 | 营业执照 URL |
| qualificationFiles | array | 否 | 资质文件 URL 列表 |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001" } }`

---

**DELETE** `/api/v1/admin/travel-agencies/{agencyId}` — 删除（软删除）

**响应：** `{ "code": 0, "msg": "已删除" }`

---

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/status` — 启停

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | int | 是 | 0禁用/1启用 |
| reason | string | 是 | 原因 |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001", "status": 0, "statusText": "已禁用" } }`

---

### 14.2 审核与分组

**POST** `/api/v1/admin/travel-agencies/{agencyId}/audit` — 审核

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| auditResult | int | 是 | 1通过/2驳回 |
| auditRemark | string | 否 | 审核备注 |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001", "auditStatus": 1, "auditStatusText": "已通过" } }`

> 审核通过后自动创建 travel_agency_account + 默认 operator 账号 + 发送通知。

---

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/group-type` — 设置分组

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| groupType | string | 是 | key/normal/society/restaurant |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001", "groupType": "key", "groupTypeText": "重点旅行社" } }`

> 分组变更后定价/授信需手动调整。

---

### 14.3 操作人员管理

**GET** `/api/v1/admin/travel-agencies/{agencyId}/operators`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | int | 否 | 0禁用/1启用 |
| role | string | 否 | order/verify/finance/admin |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**响应records字段：** operatorId, userId, userName, role, roleText, mobile(脱敏), status, statusText, createTime。

---

**POST** `/api/v1/admin/travel-agencies/{agencyId}/operators` — 添加操作人员

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | string | 是 | sys_user.id（操作员登录账号） |
| role | string | 是 | order(下单)/verify(核销)/finance(财务)/admin(管理员) |
| mobile | string | 是 | 手机号 |

**响应：** `{ "code": 0, "data": { "operatorId": "OP001", "userId": "U100", "role": "order" } }`

---

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/operators/{operatorId}` — 编辑角色

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| role | string | 是 | order/verify/finance/admin |

**响应：** `{ "code": 0, "data": { "operatorId": "OP001", "role": "finance", "roleText": "财务" } }`

---

**DELETE** `/api/v1/admin/travel-agencies/{agencyId}/operators/{operatorId}` — 移除

**响应：** `{ "code": 0, "msg": "已移除" }`

---

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/operators/{operatorId}/status` — 启停

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | int | 是 | 0禁用/1启用 |
| reason | string | 否 | 原因 |

**响应：** `{ "code": 0, "data": { "operatorId": "OP001", "status": 0 } }`

---

### 14.4 账户与钱包

**GET** `/api/v1/admin/travel-agencies/{agencyId}/accounts` — 账户信息

**响应：**
```json
{
  "code": 0,
  "data": {
    "balance": "25000.00",
    "creditLimit": "50000.00",
    "usedCredit": "7000.00",
    "availableCredit": "43000.00",
    "totalRecharge": "30000.00",
    "totalConsumption": "12000.00",
    "payMethod": "credit",
    "status": 1,
    "statusText": "正常"
  }
}
```

---

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/account/status` — 冻结/解冻

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | int | 是 | 0冻结/1正常 |
| reason | string | 是 | 原因 |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001", "status": 0, "statusText": "已冻结" } }`

---

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/credit` — 手动调授信

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| creditLimit | decimal | 是 | 新授信额度 |
| reason | string | 是 | 原因（记录 credit_log） |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001", "creditLimit": "80000.00", "availableCredit": "73000.00" } }`

---

**POST** `/api/v1/admin/travel-agencies/{agencyId}/account/recharge` — 手动充值

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| amount | decimal | 是 | 充值金额 |
| payMethod | string | 是 | transfer(对公转账)/cash(现金) |
| remark | string | 否 | 备注 |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001", "balanceAfter": "35000.00", "transactionNo": "TXN001" } }`

---

**POST** `/api/v1/admin/travel-agencies/{agencyId}/account/adjust` — 调账

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| amount | decimal | 是 | 金额（+入/-出） |
| type | string | 是 | debit(扣款)/credit(入款) |
| reason | string | 是 | 差错更正/客服补偿 |

**响应：** `{ "code": 0, "data": { "agencyId": "AG001", "balanceAfter": "24900.00", "transactionNo": "TXN002" } }`

---

**GET** `/api/v1/admin/travel-agencies/{agencyId}/account/credit-log?startDate=X&endDate=Y&pageNo=1&pageSize=20` — 授信调整历史

**响应records字段：** logId, fromCredit, toCredit, reason, operator, createTime。

---

**GET** `/api/v1/admin/travel-agencies/credit-applies?status=0&pageNo=1&pageSize=20` — 授信申请审批列表

**响应records字段：** applyId, agencyId, agencyName, applyAmount, currentCredit, reason, status, statusText, applyTime。

---

**POST** `/api/v1/admin/travel-agencies/{agencyId}/credit-apply/{applyId}/audit` — 审批授信申请

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| result | int | 是 | 1通过/2驳回 |
| remark | string | 否 | 审批备注 |

**响应：** `{ "code": 0, "data": { "applyId": "APP001", "status": 1, "statusText": "已通过", "newCreditLimit": "80000.00" } }`

---

### 14.5 账户流水

**GET** `/api/v1/admin/travel-agencies/{agencyId}/transactions`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| transactionType | string | 否 | recharge/deduct/refund/adjust |
| startDate | string | 否 | 起始日期 |
| endDate | string | 否 | 截止日期 |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**响应records字段：** transactionNo, transactionType, transactionTypeText, amount(±), balanceAfter, refId, remark, createTime。

---

### 14.6 订单与批量导入

**GET** `/api/v1/admin/travel-agencies/{agencyId}/orders`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | int | 否 | 1待支付/2已支付/3已核销/4已退款/5已取消/6部分退款/7已关闭 |
| orderNo | string | 否 | 订单号 |
| travelDate | string | 否 | 出行日期 |
| startDate | string | 否 | 下单起始 |
| endDate | string | 否 | 下单截止 |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**响应records字段：** orderId, orderNo, orderType, totalAmount, paidAmount, status, statusText, contactName, contactPhone(脱敏), travelDate, verifyCount, createTime。

---

**GET** `/api/v1/admin/travel-agencies/{agencyId}/batch-orders`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | int | 否 | 0处理中/1完成/2部分失败 |
| pageNo | int | 否 | 默认 1 |
| pageSize | int | 否 | 默认 20 |

**响应records字段：** batchNo, fileName, orderId, totalVisitors, successCount, failCount, status, statusText, createTime。

---

**GET** `/api/v1/admin/travel-agencies/batch-orders/{batchNo}` — 批次详情

**响应：**
```json
{
  "code": 0,
  "data": {
    "batchNo": "BATCH06010001",
    "agencyName": "中国国旅",
    "fileName": "tourist_list_0601.xlsx",
    "totalVisitors": 200,
    "successCount": 195,
    "failCount": 5,
    "failDetails": [
      { "row": 3, "name": "王五", "idType": "ID_CARD", "reason": "证件号重复" },
      { "row": 12, "name": "赵六", "idType": "ID_CARD", "reason": "年龄不满足购买条件" }
    ],
    "status": 2,
    "statusText": "部分失败"
  }
}
```

---

**POST** `/api/v1/admin/travel-agencies/batch-orders/{batchNo}/retry` — 失败重试

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| retryType | string | 是 | all(全部失败)/specific(指定行) |
| rows | array | 否 | 指定行号列表（retryType=specific 时必填） |

**响应：** `{ "code": 0, "data": { "retryCount": 5, "successCount": 4, "failCount": 1, "failDetails": [...] } }`

---

### 14.7 折扣定价

**GET** `/api/v1/admin/travel-agencies/{agencyId}/group-prices?skuId=X&pageNo=1&pageSize=20` — 查看定价

**响应records字段：** priceId, skuId, skuName, price, updateTime, updateBy。

---

**POST** `/api/v1/admin/travel-agencies/{agencyId}/group-prices` — 批量设置

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| prices | array | 是 | 定价列表 |
| prices[].skuId | string | 是 | SKU ID |
| prices[].price | decimal | 是 | 定价（元） |

**响应：** `{ "code": 0, "data": { "successCount": 5, "failCount": 2, "failDetails": [{"skuId":"SKU099","reason":"SKU不存在"}] } }`

---

**PUT** `/api/v1/admin/travel-agencies/{agencyId}/group-prices/{priceId}` — 修改单条定价

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| price | decimal | 是 | 新价格（元） |

**响应：** `{ "code": 0, "data": { "priceId": "PR001", "skuId": "SKU001", "price": "35.00" } }`

---

**DELETE** `/api/v1/admin/travel-agencies/{agencyId}/group-prices/{priceId}` — 删除定价

**响应：** `{ "code": 0, "msg": "已删除" }`

> group-prices 定价在 product_channel_price（channel=agent）基础之上生效，group-prices 优先级更高。

---

### 14.8 合同与统计

**POST** `/api/v1/admin/travel-agencies/{agencyId}/contract` — 上传合同

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| contractUrl | string | 是 | 合同文件 URL（先通过 /common/upload 上传） |
| contractStart | date | 否 | 合同起始日期 |
| contractEnd | date | 否 | 合同截止日期 |

**响应：** `{ "code": 0, "data": { "contractId": "CON001", "contractUrl": "https://..." } }`

---

**GET** `/api/v1/admin/travel-agencies/{agencyId}/contract` — 查看合同

**响应：** `{ "code": 0, "data": { "contractUrl": "https://...", "contractStart": "2026-01-01", "contractEnd": "2027-12-31", "uploadTime": "2026-01-15", "expiringSoon": false } }`

---

**GET** `/api/v1/admin/travel-agencies/{agencyId}/dashboard` — 旅行社统计看板

**响应：**
```json
{
  "code": 0,
  "data": {
    "totalOrders": 320,
    "totalAmount": "128000.00",
    "creditUsageRate": "70%",
    "totalVisitors": 6500,
    "batchSuccessRate": "95%",
    "monthlyTrend": [
      { "month": "2026-05", "orders": 85, "amount": "32000.00" }
    ]
  }
}
```
