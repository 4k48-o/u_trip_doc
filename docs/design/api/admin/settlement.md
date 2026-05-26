# 清分结算 — `/api/v1/admin/settlements`

> 佣金比例来源：§7.3 商户管理 → merchant_commission。对账依赖：第 4 节订单管理。

### 5.1 清分明细

**GET** `/api/v1/admin/settlements`

| 参数 | 必填 | 说明 |
|------|------|------|
| merchantId | 否 | 商户筛选 |
| orderId | 否 | 订单 ID |
| channel | 否 | 渠道 |
| settlementNo | 否 | 清分单号 |
| settleDate | 否 | 结算日期 |
| settleStatus | 否 | 0待结算/1已结算/2已打款 |
| startDate | 否 | 结算起始 |
| endDate | 否 | 结算截止 |
| sortBy | 否 | settle_date_desc/amount_desc |
| pageNo | 否 | 默认 1 |
| pageSize | 否 | 默认 20 |

**响应records字段：** id, settlementNo, orderId, orderNo, productId, productName, merchantId, merchantName, channel, totalAmount, commissionRate, commissionAmount, settleAmount, settleDate, settleStatus, settleStatusText, createTime。

**GET** `/api/v1/admin/settlements/{id}` — 单条详情

**PUT** `/api/v1/admin/settlements/{id}/settle` — 标记已结算（0→1）

**PUT** `/api/v1/admin/settlements/{id}/remit` — 标记已打款（1→2）

**POST** `/api/v1/admin/settlements/batch-settle` — 批量结算 `{ "ids": ["...","..."] }`

**GET** `/api/v1/admin/settlements/summary?groupBy=merchant\|channel\|date&startDate=X&endDate=Y` — 汇总视图

**GET** `/api/v1/admin/settlements/export` — 导出 Excel（筛选参数同列表）

---

### 5.2 对账单

**GET** `/api/v1/admin/settlements/ledgers?merchantId=X&status=Y&periodStart=A&periodEnd=B&pageNo=1` — 列表

| status 枚举 | 说明 |
|------|------|
| 0 | 待确认 |
| 1 | 已确认 |
| 2 | 已开票 |
| 3 | 已打款 |

**GET** `/api/v1/admin/settlements/ledgers/{ledgerId}` — 详情（含包含的清分明细项列表）

**POST** `/api/v1/admin/settlements/ledgers/generate` — 生成对账单 `{ "merchantId": "...", "periodStart": "...", "periodEnd": "..." }`。响应: `{ "ledgerNo": "...", "ledgerId": "..." }`

**PUT** `/api/v1/admin/settlements/ledgers/{ledgerId}/confirm` — 确认（0→1）

**PUT** `/api/v1/admin/settlements/ledgers/{ledgerId}/reject` — 驳回（1→0）`{ "reason": "..." }`

**PUT** `/api/v1/admin/settlements/ledgers/{ledgerId}/invoice` — 标记已开票（1→2）

**PUT** `/api/v1/admin/settlements/ledgers/{ledgerId}/remit` — 标记已打款（2→3）

**GET** `/api/v1/admin/settlements/ledgers/export` — 导出 Excel

---

### 5.3 用友 NC 凭证

**GET** `/api/v1/admin/settlements/nc-vouchers?syncStatus=X&settlementId=Y&startDate=A&endDate=B&pageNo=1` — 列表

**响应records字段：** id, voucherNo, ncBizId, settlementId, settlementNo, voucherType, debitAccount, creditAccount, amount, voucherDate, syncStatus, syncStatusText, syncTime, syncError。

**GET** `/api/v1/admin/settlements/nc-vouchers/{id}` — 详情（含完整同步错误信息）

**POST** `/api/v1/admin/settlements/nc-vouchers/sync` — 手动同步 `{ "ledgerId": "..." }`（同步指定对账单下的待同步凭证）

**POST** `/api/v1/admin/settlements/nc-vouchers/{id}/retry` — 单条重试

**GET** `/api/v1/admin/settlements/nc-vouchers/export` — 导出 Excel

---

