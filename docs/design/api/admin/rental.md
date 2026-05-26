# 租赁管理 — `/api/v1/admin/rental`

**GET** `/api/v1/admin/rental/devices` — 设备台账列表 `?status=X&areaId=Y`

**POST/PUT/DELETE** `/api/v1/admin/rental/devices` — 设备台账 CRUD

**GET** `/api/v1/admin/rental/orders` — 租赁订单列表 `?status=X&deviceNo=Y`

**GET** `/api/v1/admin/rental/orders/{rentalNo}` — 租赁订单详情

**PUT** `/api/v1/admin/rental/orders/{rentalNo}/damage` — 标记设备损坏 `{ "damageRemark": "...", "deductionAmount": "50.00" }`

---

