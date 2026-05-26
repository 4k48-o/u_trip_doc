# 年卡管理 — `/api/v1/admin/annual-cards`

**GET** `/api/v1/admin/annual-cards?status=X&keyword=Y&pageNo=1`

**GET** `/api/v1/admin/annual-cards/{cardNo}` — 年卡详情（含持卡人/证件/有效期/人脸状态）

**POST** `/api/v1/admin/annual-cards/issue` — 窗口办卡 `{ "skuId": "...", "name": "...", "idType": "...", "idNo": "<AES>", "faceImageId": "..." }`

**PUT** `/api/v1/admin/annual-cards/{cardNo}/status` — 状态变更（激活/挂失/注销）`{ "status": 2 }`

**PUT** `/api/v1/admin/annual-cards/{cardNo}/face-rebind` — 人脸重绑 `{ "faceImageId": "..." }`

---

