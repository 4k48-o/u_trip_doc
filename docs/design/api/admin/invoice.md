# 发票管理 — `/api/v1/admin/invoices`

### 6.1 开票申请

**GET** `/api/v1/admin/invoices/apply`

| 参数 | 必填 | 说明 |
|------|------|------|
| status | 否 | 0待开具/1已开具/2已冲红/3已作废 |
| invoiceType | 否 | personal / enterprise |
| applyNo | 否 | 申领单号 |
| userId | 否 | 申请人 ID |
| startDate | 否 | 申请起始 |
| endDate | 否 | 申请截止 |
| pageNo | 否 | 默认 1 |
| pageSize | 否 | 默认 20 |

**响应records字段：** applyId, applyNo, userId, userName, orderCount(关联订单数), invoiceType, invoiceTypeText, invoiceTitle, taxNo(脱敏), amount, email, status, statusText, createTime。

**GET** `/api/v1/admin/invoices/apply/{applyId}` — 申领详情（含关联订单列表 `invoice_apply_order`、用户信息、完整抬头/税号）

**POST** `/api/v1/admin/invoices/apply/{applyId}/audit` — 审核 `{ "auditResult": 1\|2, "auditRemark": "..." }`

**POST** `/api/v1/admin/invoices/apply/{applyId}/void` — 作废（仅 status=0 可作废）`{ "reason": "..." }`

---

### 6.2 发票开具

**POST** `/api/v1/admin/invoices/issue`

| 字段 | 必填 | 说明 |
|------|------|------|
| applyId | 是 | 申领 ID |

**响应：** `{ "code": 0, "data": { "invoiceNo": "044002100111", "invoiceCode": "044002100111", "invoiceUrl": "https://...", "taxPlatformNo": "..." } }`（invoiceNo/code 由税控平台返回，invoiceUrl 同步返回下载链接）

**POST** `/api/v1/admin/invoices/batch-issue` — 批量开具 `{ "applyIds": ["...","..."] }`。返回: `{ "successCount": 18, "failCount": 2, "failDetails": [...] }`

**POST** `/api/v1/admin/invoices/red` — 冲红 `{ "invoiceId": "...", "redReason": "..." }`

> 冲红后：原发票 `issue_type=red`，`invoice_apply.status` → `2=已冲红`。系统自动生成新的 `invoice_record`（issue_type=red，通过 `parent_invoice_id` 关联原发票）。原发票文件 URL 仍然可访问（标注"已冲红"水印）。

---

### 6.3 发票记录查询

**GET** `/api/v1/admin/invoices/records`

| 参数 | 必填 | 说明 |
|------|------|------|
| applyId | 否 | 申领 ID |
| issueType | 否 | develop(开具) / red(冲红) |
| invoiceNo | 否 | 发票号码 |
| startDate | 否 | 开具起始 |
| endDate | 否 | 开具截止 |
| pageNo | 否 | 默认 1 |

**响应records字段：** id, applyId, invoiceNo, invoiceCode, taxPlatformNo, invoiceUrl, issueType, issueTypeText, issueTime。

**GET** `/api/v1/admin/invoices/records/{id}` — 详情（含完整发票内容 JSON、税控流水号、冲红关联原票）

---

### 6.4 开票统计

**GET** `/api/v1/admin/invoices/stats`

| 参数 | 必填 | 说明 |
|------|------|------|
| category | 否 | 旅游服务/商品销售 |
| invoiceType | 否 | personal/enterprise |
| startDate | 是 | 起始 |
| endDate | 是 | 截止 |

响应同原有格式。

**GET** `/api/v1/admin/invoices/export` — 导出 Excel（筛选参数同申领列表 + records 列表）

---

