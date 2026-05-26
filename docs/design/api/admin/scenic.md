# 景区管理 — `/api/v1/admin/scenic`

> 景区与景点管理的完整 API 端点。

### 1.1 景区列表/详情

**GET** `/api/v1/admin/scenic`

| 参数 | 必填 | 说明 |
|------|------|------|
| keyword | 否 | 名称模糊搜索 |
| status | 否 | 0关闭/1开放 |
| pageNo | 否 | — |
| pageSize | 否 | — |

**POST** `/api/v1/admin/scenic` — 新增景区

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| areaCode | string | 是 | 景区编码 |
| areaName | string | 是 | 景区名称 |
| areaNameEn | string | 否 | 英文名称（OTA 使用） |
| address | string | 否 | 地址 |
| longitude | decimal | 是 | 经度 |
| latitude | decimal | 是 | 纬度 |
| contactPhone | string | 否 | 联系电话 |
| description | string | 否 | 景区简介 |
| openingHours | string | 否 | `[{"season":"spring","time":"08:00-17:00"}]` |
| logo | string | 否 | Logo URL |
| status | int | 是 | 0/1 |

**PUT** `/api/v1/admin/scenic/{areaId}` — 编辑景区

**DELETE** `/api/v1/admin/scenic/{areaId}` — 逻辑删除

**POST** `/api/v1/admin/scenic/{areaId}/languages` — 设置多语言内容 `{ "languageCode": "en", "areaName": "...", "description": "...", "address": "...", "openingHours": "..." }`

### 1.2 景点管理

**GET** `/api/v1/admin/scenic/{areaId}/spots` — 景区下景点列表

**POST** `/api/v1/admin/scenic/{areaId}/spots` — 新增景点

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| spotCode | string | 是 | 景点编码 |
| spotName | string | 是 | 景点名称 |
| spotNameEn | string | 否 | 英文名称 |
| spotType | string | 是 | scenic / cultural / facility / entrance |
| longitude | decimal | 是 | 经度 |
| latitude | decimal | 是 | 纬度 |
| googlePlaceId | string | 否 | Google Place ID（OTA 映射用） |
| description | string | 否 | 介绍 |
| sortNo | int | 否 | 排序 |
| status | int | 是 | 0/1 |

**PUT** `/api/v1/admin/scenic/{areaId}/spots/{spotId}` — 编辑

**DELETE** `/api/v1/admin/scenic/{areaId}/spots/{spotId}` — 逻辑删除

**POST** `/api/v1/admin/scenic/{areaId}/spots/{spotId}/languages` — 设置多语言内容 `{ "languageCode": "en", "spotName": "...", "description": "..." }`

---

