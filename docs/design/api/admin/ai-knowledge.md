# AI 知识库管理 — `/api/v1/admin/ai`

**GET** `/api/v1/admin/ai/knowledge?category=X&keyword=Y` — 知识条目列表

**POST/PUT/DELETE** `/api/v1/admin/ai/knowledge` — CRUD

**GET** `/api/v1/admin/ai/sessions` — AI 对话日志 `?userId=X&status=Y`

**GET** `/api/v1/admin/ai/sessions/{sessionId}/messages` — 对话记录

**POST** `/api/v1/admin/ai/messages/{msgId}/correct` — 对话纠错 `{ "correctContent": "..." }`

---

