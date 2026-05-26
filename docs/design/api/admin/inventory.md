# 进销存管理 — `/api/v1/admin/inventory`

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

