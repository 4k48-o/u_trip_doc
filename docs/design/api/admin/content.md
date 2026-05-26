# 内容管理 — `/api/v1/admin/content`

**GET/POST/PUT** `/api/v1/admin/content/articles` — 资讯管理

**GET** `/api/v1/admin/content/notes?auditStatus=0` — 笔记审核列表

**POST** `/api/v1/admin/content/notes/{noteId}/audit` — 审核 { "auditResult": 1, "remark": "" }

**PUT** `/api/v1/admin/content/notes/{noteId}/top` — 置顶/加精 { "isTop": true, "isEssence": true }

**GET/POST/PUT** `/api/v1/admin/content/audio` — 语音讲解管理

| 字段 | 必填 | 说明 |
|------|------|------|
| scenicSpot | 是 | 景点英文标识 |
| audioUrl | 是 | 音频 OSS URL |
| transcript | 否 | 文字稿 |
| language | 是 | zh-CN / en |
| duration | 否 | 时长（秒） |

**GET/POST/PUT** `/api/v1/admin/content/routes` — 导览路线

| 字段 | 必填 | 说明 |
|------|------|------|
| routeCode | 是 | 路线编码 |
| routeName | 是 | 路线名称 |
| areaId | 是 | 景区 |
| routeType | 是 | recommend / family / senior / quick |
| waypoints | 是 | `[{"spotId":"...","order":1,"duration":30}]` |

**GET/POST/PUT** `/api/v1/admin/content/map/{markerId}` — 地图标注

---

