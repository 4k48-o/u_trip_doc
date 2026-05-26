# OTA 管理 — `/api/v1/admin/ota`

### 17.1 OTA 商品映射

**GET** `/api/v1/admin/ota/mappings?otaCode=X&status=Y`

**POST** `/api/v1/admin/ota/mappings`

| 字段 | 必填 | 说明 |
|------|------|------|
| skuId | 是 | 内部 SKU |
| otaCode | 是 | ctrip / meituan / qunar / douyin / xiaohongshu / feizhu / linktivity |
| otaOptionId | 是 | OTA 侧 OptionId |
| otaSupplierOptionId | 否 | OTA 侧 PLU |
| otaProductId | 否 | OTA 产品 ID |
| contractId | 是 | OTA 合同 ID |
| dateType | 是 | DATE_REQUIRED / DATE_NOT_REQUIRED |
| syncPrice | 是 | 是否同步价格（布尔） |
| syncInventory | 是 | 是否同步库存（布尔） |
| pushEnabled | 是 | 是否启用推送（布尔） |

### 17.2 OTA POI 映射

**GET/POST** `/api/v1/admin/ota/poi-mappings`

### 17.3 OTA 同步日志

**GET** `/api/v1/admin/ota/sync-logs?otaCode=X&syncType=Y&status=Z&startTime=A&endTime=B`

### 17.4 OTA 对账

**GET** `/api/v1/admin/ota/reconciliations?otaCode=X&bizDate=Y`

**POST** `/api/v1/admin/ota/reconciliations/process` — 处理差异

| 字段 | 必填 | 说明 |
|------|------|------|
| reconciliationId | 是 | 对账单 ID |
| action | 是 | confirm / ignore |
| remark | 否 | 备注 |

### 17.5 OTA 通道管理

**POST** `/api/v1/admin/ota/mappings/{mappingId}/resync` — 手动触发重新同步（产品/价格/库存）

**PUT** `/api/v1/admin/ota/mappings/{mappingId}/toggle` — 启用/停用 OTA 通道 `{ "pushEnabled": false }`
