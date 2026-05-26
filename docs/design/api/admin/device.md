# 设备管理 — `/api/v1/admin/devices`

> 所有设备列表 GET 共享参数：`status`(0离线/1在线/2故障), `areaId`, `keyword`, `pageNo`, `pageSize`。

| 设备 | 路由前缀 | 特有字段 |
|------|----------|----------|
| 闸机 | `/api/v1/admin/devices/gates` | gateCode, gateName, scenicSpotId, areaId, gateLocation, gateType(fixed/mobile), ipAddress, macAddress |
| 手持机 | `/api/v1/admin/devices/handhelds` | deviceCode, deviceName, areaId, osVersion, ipAddress, batteryLevel |
| 打印机 | `/api/v1/admin/devices/printers` | deviceCode, areaId, printerModel, printType(thermal/dot/laser), interface(USB/COM/LAN/WiFi), ipAddress |
| 扫描枪 | `/api/v1/admin/devices/scanners` | deviceCode, areaId, scannerModel, scanType(1d/2d/rfid), interface(USB/COM/Bluetooth) |
| 护照阅读器 | `/api/v1/admin/devices/passport-readers` | deviceCode, areaId, readerModel, supportDocTypes(PASSPORT/ID_CARD/HK_MO/...), interface |
| 客流计数器 | `/api/v1/admin/devices/counters` | deviceCode, storeId, areaId, counterModel, countDirection(in/out/both) |
| 收银机 | `/api/v1/admin/devices/cashiers` | deviceCode, storeId, areaId, osVersion, supportPayments(cash/qrcode/card), screenSize |

> 全部 7 种设备支持 `GET/POST/PUT/DELETE` 统一 CRUD。列表查询参数含 `status`/`areaId`/`keyword`/`pageNo`/`pageSize`。

### 13.1 设备心跳与历史

**GET** `/api/v1/admin/devices/{type}/{deviceId}/heartbeat?startDate=X&endDate=Y` — 通用心跳历史查询（所有设备类型: gates/handhelds/printers/scanners/passport-readers/counters/cashiers）

**响应records字段：** id, deviceId, deviceType, status, batteryLevel(手持机), heartbeatTime。

### 13.2 设备状态与同步

**PUT** `/api/v1/admin/devices/{type}/{deviceId}/status` — 手动标记状态 `{ "status": 0|1|2, "reason": "闸机报修" }`

**POST** `/api/v1/admin/devices/{type}/{deviceId}/sync` — 通用强制同步（闸机:票务底库/人脸；手持机:票务底库/人脸；其他:配置同步）`{ "syncType": "face_lib|ticket_rules|all" }`

### 13.3 手持机专项

**GET** `/api/v1/admin/devices/handhelds/low-battery?threshold=20` — 低电量设备列表

**GET** `/api/v1/admin/devices/handhelds/os-stats` — 系统版本分布统计

### 13.4 设备-核销关联

**GET** `/api/v1/admin/devices/gates/{gateId}/verify-logs?date=X` — 该闸机当日核销记录（辅助判断设备是否异常）

**GET** `/api/v1/admin/devices/dashboard` — 设备总览 `{ "totalDevices": 65, "onlineRate": "92%", "faultDevices": 3, "gates": { "online": 23, "offline": 1, "fault": 1 }, "handhelds": { "online": 28, "offline": 2, "lowBattery": 5 } }`

### 13.5 设备维护

**PUT** `/api/v1/admin/devices/{type}/{deviceId}/maintenance` — 记录维修信息 `{ "maintenanceType": "repair|upgrade", "description": "...", "result": "finished", "cost": "500.00" }`

**GET** `/api/v1/admin/devices/{type}/{deviceId}/maintenance-log` — 维修历史

---

