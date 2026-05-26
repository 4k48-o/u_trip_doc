# 客服会话管理 — `/api/v1/admin/cs`

**GET** `/api/v1/admin/cs/sessions` — 客服会话列表 `?status=queuing/active&agentId=X`

**PUT** `/api/v1/admin/cs/sessions/{sessionId}/assign` — 指派坐席 `{ "agentId": "..." }`

**GET** `/api/v1/admin/cs/sessions/{sessionId}/messages` — 会话消息查询

---

