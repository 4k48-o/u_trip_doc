# 数据库设计

> 基于 JEECG Boot v3.9.2 基础数据库 + 绥中项目业务表设计。

---

## 1. 设计约定

| 项目 | 约定 |
|------|------|
| 字符集 | utf8mb4 |
| 排序规则 | utf8mb4_unicode_ci |
| 引擎 | InnoDB |
| 主键 | `id varchar(32)` 雪花/分布式 ID，非自增 |
| 审计字段 | `create_by varchar(50)`, `create_time datetime`, `update_by varchar(50)`, `update_time datetime` |
| 逻辑删除 | `del_flag tinyint(1) default 0` |
| 外键 | 不使用，关联关系由应用层保证 |
| 命名 | 表名/列名 snake_case，表名不使用复数 |
| 多租户 | 本项目不使用，表中不设 `tenant_id` 字段 |

### 1.1 数据库拆分与边界

> 遵循微服务架构，各服务独立数据库。JEECG Boot 系统表（sys_* 等 61 张）保留在 `jeecg-boot` 库中供所有服务通过 API 调用共享。服务间通过 API 或消息队列通信，禁止跨库直连。

| 数据库 | 所属服务 | 业务表 |
|--------|----------|--------|
| ticket_db | ticket-service | ticket_*, device_gate, device_handheld |
| product_db | product-service | product_*, product_language, product_option_slot, season_config, season_config_slot |
| order_db | order-service | order_*, order_payment |
| settlement_db | settlement-service | settlement_* |
| invoice_db | invoice-service | invoice_* |
| face_db | face-service | face_* |
| merchant_db | merchant-service | merchant_*, shop_page |
| marketing_db | marketing-service | marketing_* |
| member_db | member-service | member_*, member_tag |
| inventory_db | inventory-service | inv_* |
| traffic_db | traffic-service | traffic_* |
| integration_db | integration-service | ota_*, travel_agency_* |
| scenic_db | scenic-service | scenic_*, biz_*, biz_property_space |
| vehicle_db | vehicle-service | vehicle_* |
| content_db | content-service | content_* |
| device_db | admin-service | device_printer, device_scanner, device_passport_reader, device_counter, device_cashier |
| analytics_db | analytics-service | alert_*, report_*（分析类，与交易DB物理分离） |
| rule_db | rule-service | rule_* |
| rental_db | rental-service | rental_* |
| notification_db | notification-service | 消息模板/发送日志复用 JEECG 的 sys_sms/sys_sms_template，通知服务通过 API 调用 jeecg-boot 库 |

> 所有业务表遵循统一约定：InnoDB、utf8mb4_unicode_ci、snake_case、雪花 ID、审计字段、逻辑删除、无外键、无多租户。

---

## 2. JEECG Boot 基础数据库（不改动）

> 以下表由 JEECG Boot 平台自带，本文档仅作参考索引。所有核心系统表（61 张）位于数据库 `jeecg-boot`。微服务模式下还需独立数据库 `nacos`（7 张表）和 `xxl_job`（4 张表）。

| 数据库 | 用途 | 表数 | 模式 |
|--------|------|------|------|
| jeecg-boot | 核心系统 | 61 | 必须 |
| nacos | Nacos 注册/配置中心 | 7 | 微服务 |
| xxl_job | XXL-JOB 分布式任务调度 | 4 | 微服务 |

**核心系统表速览：** sys_user（用户/职员，仅后台管理）、sys_role、sys_permission、sys_depart、sys_dict、sys_dict_item、sys_log（MyISAM，支持分表）、sys_data_log、sys_announcement、sys_sms、sys_sms_template、sys_quartz_job + qrtz_*（11张）、oss_file、sys_data_source、sys_gateway_route、open_api 等。
**可选模块：** Online 低代码（26张）、JimuReport（16张）、AI/RAG（9张），共计 62 张。

---

## 3. 绥中项目业务表（按服务分库）

### 3.1 游客用户表 — tourist_db

#### tourist_user（C 端游客用户）

> 独立于 sys_user（后台管理系统用户）。C 端游客通过微信/支付宝/手机号登录。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| user_no | varchar(50) | 游客编号，唯一索引 |
| nickname | varchar(100) | 昵称 |
| avatar | varchar(500) | 头像 URL |
| phone | varchar(50) | 手机号（AES 加密），唯一索引 |
| email | varchar(100) | 邮箱（AES 加密），唯一索引 |
| real_name | varchar(100) | 真实姓名 |
| id_type | varchar(20) | 证件类型 |
| id_no | varchar(200) | 证件号（AES 加密） |
| id_no_hash | varchar(64) | 证件号 SHA-256 哈希，唯一索引，用于去重查询（加密存储的同时支持唯一性校验） |
| gender | varchar(10) | M/F/U |
| birthday | date | 生日 |
| nationality | varchar(50) | 国籍 |
| register_source | varchar(20) | miniapp/h5/pc/window |
| status | tinyint(1) | 1正常/0禁用 |
| last_login_time | datetime | 最后登录时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_user_no, uk_phone, uk_id_no_hash, idx_status

> **说明：** `tourist_user.id_no_hash` 为证件号 SHA-256 哈希（同证件号生成相同哈希值），用于唯一性去重校验。原始证件号 `id_no` 仍为 AES 加密存储。`sys_third_account.sys_user_id` 存的是 `tourist_user.id`（C 端游客），与 `sys_user.id`（后台管理用户）互斥。一个微信/支付宝 UnionID 只绑定一个游客账号，通过 `sys_third_account.third_type + third_user_id` 唯一索引保证。双 Token 体系：C 端用 `tourist_*` 组 Token（仅可访问 `/api/v1/tourist/`），后台用 `admin/merchant/agent` 组 Token（仅可访问各自路由组）。

---

### 3.2 景区管理 — scenic_db

#### scenic_area（景区）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| area_code | varchar(50) | 景区编码，唯一索引 |
| area_name | varchar(200) | 景区名称 |
| area_name_en | varchar(200) | 英文名称 |
| address | varchar(500) | 景区地址 |
| longitude | decimal(12,8) | 经度 |
| latitude | decimal(12,8) | 纬度 |
| contact_phone | varchar(50) | 联系电话 |
| description | text | 景区简介 |
| opening_hours | varchar(500) | 开放时间说明（JSON） |
| logo | varchar(500) | 景区 Logo URL |
| status | tinyint(1) | 0关闭/1开放 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_area_code, idx_status

#### scenic_spot（景点）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| area_id | varchar(32) | 所属景区 ID |
| spot_code | varchar(50) | 景点编码，唯一索引 |
| spot_name | varchar(200) | 景点名称 |
| spot_name_en | varchar(200) | 英文名称 |
| spot_type | varchar(20) | scenic/cultural/facility/entrance |
| longitude | decimal(12,8) | 经度 |
| latitude | decimal(12,8) | 纬度 |
| google_place_id | varchar(200) | Google Place ID（OTA 映射用） |
| description | text | 景点介绍 |
| sort_no | int(11) | 排序 |
| status | tinyint(1) | 0隐藏/1显示 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_spot_code, idx_area_id, idx_spot_type

#### ota_poi_mapping（OTA POI 映射）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| target_type | varchar(20) | scenic_spot/scenic_area/merchant_shop/redemption_location |
| target_id | varchar(32) | 映射目标 ID |
| ota_code | varchar(20) | ctrip/meituan/qunar/douyin/xiaohongshu |
| supplier_poi_id | varchar(200) | 供应商 POI ID |
| ota_poi_id | varchar(200) | OTA 返回的 POI ID |
| google_place_id | varchar(200) | Google Place ID |
| poi_name | varchar(200) | POI 名称 |
| address_detail | varchar(200) | 详细地址 |
| longitude | decimal(12,8) | 经度 |
| latitude | decimal(12,8) | 纬度 |
| status | tinyint(1) | 0未映射/1已映射 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_target_ota（联合唯一），idx_ota_code

---

### 3.3 票务服务 — ticket_db

#### ticket_reservation（分时预约）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| reservation_no | varchar(50) | 预约单号，唯一索引 |
| user_id | varchar(32) | 游客用户 ID，关联 tourist_user.id |
| product_id | varchar(32) | 票种 SKU ID |
| visit_date | date | 游览日期 |
| time_slot | varchar(20) | 时段（如 08:00-10:00） |
| quantity | int(11) | 数量 |
| contact_name | varchar(100) | 联系人姓名 |
| contact_phone | varchar(45) | 联系人手机 |
| contact_id_type | varchar(20) | 证件类型 |
| contact_id_no | varchar(100) | 证件号 |
| channel | varchar(20) | online/window/ota/agent |
| status | tinyint(1) | 1待支付/2已支付/3已改签/4已取消/5已核销 |
| expire_time | datetime | 支付过期时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_reservation_no, idx_user_id, idx_visit_date, idx_status

#### ticket_visitor（游客信息）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| reservation_id | varchar(32) | 预约 ID |
| order_id | varchar(32) | 订单 ID |
| name | varchar(100) | 姓名 |
| id_type | varchar(20) | ID_CARD/PASSPORT/HK_MO/TAIWAN/PERMANENT_RESIDENCE |
| id_no | varchar(200) | 证件号（AES 加密） |
| nationality | varchar(50) | 国籍 |
| gender | varchar(10) | M/F/U |
| age | tinyint(3) UNSIGNED | 年龄 |
| verify_status | tinyint(1) | 0未核销/1已核销/2部分核销 |
| verify_time | datetime | 核销时间 |
| verify_gate | varchar(32) | 核销闸机 ID |
| verify_method | varchar(20) | qr_code/id_card/passport/face/manual |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_reservation_id, idx_id_no, idx_verify_status

#### ticket_stock（库存）

> **权威源：** 库存唯一写入通路。可用库存 = product_price_calendar.stock - used_stock - frozen_stock。Redis Lua 脚本做原子预扣。MySQL 层 CHECK (used_stock + frozen_stock <= total_stock)。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| product_id | varchar(32) | 票种 SKU ID |
| visit_date | date | 游览日期 |
| time_slot | varchar(20) | 时段 |
| channel | varchar(20) | online/window/ota/agent |
| total_stock | int(11) | 总库存 |
| used_stock | int(11) | 已售库存 |
| frozen_stock | int(11) | 冻结库存（未支付订单） |
| status | tinyint(1) | 1启用/0停用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_product_date_slot_channel（联合唯一）, CHECK (used_stock + frozen_stock <= total_stock)

#### season_config（季节/时段规则）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| config_code | varchar(50) | 配置编码，唯一索引 |
| config_name | varchar(200) | 如"旺季日场" |
| product_id | varchar(32) | 适用票种 SKU ID |
| scene_type | varchar(20) | day(日场)/night(夜场) |
| start_date | date | 生效开始日期 |
| end_date | date | 生效结束日期 |
| advance_days | int(11) | 最大提前预约天数 |
| release_rule | varchar(20) | auto(到时间自动释放)/manual(手动) |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_config_code, idx_product_id, idx_start_date

#### season_config_slot（时段明细）

> 从 season_config.time_slots JSON 拆出。每个时段一行，可独立查询和更新。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| config_id | varchar(32) | 关联 season_config.id |
| slot | varchar(20) | 时段（如 08:00-10:00） |
| max_capacity | int(11) | 该时段最大容量 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_config_slot（联合唯一）

#### ticket_annual_card（年卡）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| card_no | varchar(50) | 年卡编号，唯一索引 |
| user_id | varchar(32) | 游客用户 ID，关联 tourist_user.id |
| name | varchar(100) | 持卡人姓名 |
| id_type | varchar(20) | 证件类型 |
| id_no | varchar(200) | 证件号（AES 加密） |
| face_image_id | varchar(32) | 人脸图片 ID |
| effective_date | date | 生效日期 |
| expire_date | date | 到期日期 |
| status | tinyint(1) | 1正常/0过期/2挂失 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_card_no, idx_user_id, idx_status

#### ticket_verify_log（检票记录）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| visitor_id | varchar(32) | 游客记录 ID |
| order_id | varchar(32) | 订单 ID |
| product_id | varchar(32) | 票种 SKU ID |
| verify_method | varchar(20) | qr_code/id_card/passport/face/manual |
| device_gate_id | varchar(32) | 闸机设备 ID |
| device_handheld_id | varchar(32) | 手持机设备 ID |
| gate_location | varchar(50) | 检票口位置（冗余自 device_gate） |
| verify_result | tinyint(1) | 1成功/0失败 |
| fail_reason | varchar(255) | 失败原因 |
| verify_time | datetime | 核销时间 |
| offline_flag | tinyint(1) | 0在线/1离线 |
| create_time | datetime | 创建时间 |

**索引：** idx_visitor_id, idx_order_id, idx_verify_time, idx_device_gate_id

---

### 3.3a 旅行社管理 — integration_db

#### travel_agency（旅行社信息）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| agency_code | varchar(50) | 旅行社编码，唯一索引 |
| agency_name | varchar(200) | 旅行社名称 |
| group_type | varchar(20) | key(重点)/normal(普通)/society(社会团体)/restaurant(周边餐馆) |
| contact_name | varchar(100) | 联系人 |
| contact_phone | varchar(50) | 联系电话 |
| business_license | varchar(200) | 营业执照 URL |
| qualification_files | text | 资质文件列表（JSON） |
| audit_status | tinyint(1) | 0待审核/1通过/2驳回 |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_agency_code, idx_group_type, idx_audit_status

#### travel_agency_account（旅行社账户）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| agency_id | varchar(32) | 旅行社 ID，唯一索引 |
| balance | decimal(10,2) | 账户余额 |
| credit_limit | decimal(10,2) | 授信额度 |
| used_credit | decimal(10,2) | 已用额度 |
| total_recharge | decimal(10,2) | 累计充值 |
| total_consumption | decimal(10,2) | 累计消费 |
| pay_method | varchar(50) | qrcode/transfer/auto_deduct/credit/on_site |
| status | tinyint(1) | 0冻结/1正常 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_agency_id

#### travel_agency_transaction（旅行社账户流水）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| agency_id | varchar(32) | 旅行社 ID |
| transaction_no | varchar(50) | 流水号，唯一索引 |
| transaction_type | varchar(20) | recharge(充值)/deduct(扣款)/refund(退款)/adjust(调账) |
| amount | decimal(10,2) | 金额（+收入/-支出） |
| balance_after | decimal(10,2) | 操作后余额 |
| ref_id | varchar(32) | 关联业务 ID |
| remark | varchar(500) | 备注 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_transaction_no, idx_agency_id, idx_create_time

#### travel_agency_batch_order（旅行社批量导入）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| agency_id | varchar(32) | 旅行社 ID |
| batch_no | varchar(50) | 批次号，唯一索引 |
| order_id | varchar(32) | 主订单 ID |
| file_name | varchar(200) | 上传文件名 |
| total_visitors | int(11) | 总游客数 |
| success_count | int(11) | 成功导入数 |
| fail_count | int(11) | 失败数 |
| fail_detail | text | 失败明细（JSON） |
| status | tinyint(1) | 0处理中/1完成/2部分失败 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_batch_no, idx_agency_id, idx_order_id

---

### 3.3b 设备管理 — ticket_db + device_db

#### device_gate（闸机设备）— ticket_db

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| gate_code | varchar(50) | 闸机编号，唯一索引 |
| gate_name | varchar(200) | 闸机名称 |
| scenic_spot_id | varchar(32) | 所属景点 ID |
| area_id | varchar(32) | 所属景区 ID |
| gate_location | varchar(100) | 位置描述（如南口1号通道） |
| gate_type | varchar(20) | fixed(固定式)/mobile(移动式) |
| ip_address | varchar(45) | 设备 IP |
| mac_address | varchar(50) | MAC 地址 |
| status | tinyint(1) | 0离线/1在线/2故障 |
| last_heartbeat | datetime | 最后心跳时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_gate_code, idx_scenic_spot_id, idx_status

#### device_handheld（手持验票设备）— ticket_db

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| device_code | varchar(50) | 设备编号，唯一索引 |
| device_name | varchar(200) | 设备名称 |
| area_id | varchar(32) | 所属景区 ID |
| os_version | varchar(50) | 系统版本（Android 13+） |
| ip_address | varchar(45) | 当前 IP |
| status | tinyint(1) | 0离线/1在线/2故障 |
| battery_level | tinyint(2) | 电量百分比 |
| last_heartbeat | datetime | 最后心跳时间 |
| last_sync_time | datetime | 最后数据同步时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_device_code, idx_status

#### device_printer、device_scanner、device_passport_reader、device_counter、device_cashier — device_db

> 五张表结构同 `device_handheld` 模式（device_code、area_id、状态、心跳），各有型号/接口等特定字段。略。

---

### 3.4 商品服务 — product_db

#### product_spu（商品 SPU）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| spu_code | varchar(50) | SPU 编码，唯一索引 |
| spu_name | varchar(200) | 商品名称 |
| category_code | varchar(50) | TICKET(门票)/COMBO(联票)/SPOT(景点票)/VEHICLE(车船票)/PERFORMANCE(演艺票)/HOTEL(酒店)/CATERING(餐饮)/CULTURAL(文创)/STUDY(研学)/RENTAL(租赁)/PARKING(停车) |
| scenic_spot_id | varchar(32) | 所属景点 ID |
| description | text | 商品描述 |
| main_image | varchar(500) | 主图 URL |
| images | text | 图片列表（JSON） |
| notice | text | 购买须知 |
| refund_policy | text | 退改规则 |
| inclusions | text | 费用包含 |
| exclusions | text | 费用不含 |
| highlight | text | 产品亮点 |
| how_to_use | text | 使用方法/步骤 |
| additional_info | text | 附加信息 |
| delivery_method | varchar(20) | DIGITAL(电子票)/PRINT(打印)/VALID_ID(证照入园) |
| redemption_type | varchar(30) | Direct_Entry/Need_Ticket_Exchange/Meet_at_Start_Point/Pick_Up_Everyone |
| booking_cutoff_time | varchar(50) | 预订截止时间（JSON） |
| duration_value | int(11) | 时长数值 |
| duration_unit | varchar(10) | Day/Hour/Minute |
| status | tinyint(1) | 0下架/1上架 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_spu_code, idx_category_code, idx_status

#### product_sku（商品 SKU）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| spu_id | varchar(32) | SPU ID |
| sku_code | varchar(50) | SKU 编码，唯一索引 |
| sku_name | varchar(200) | SKU 名称 |
| spec_desc | varchar(500) | 规格描述（JSON） |
| price_type | varchar(20) | FULL(全价)/DISCOUNT(优惠)/FREE(免费) |
| original_price | decimal(10,2) | 原价 |
| sell_price | decimal(10,2) | 售价 |
| cost_price | decimal(10,2) | 成本价（清分用） |
| need_real_name | tinyint(1) | 是否需要实名：0否/1是 |
| max_buy_per_order | int(11) | 每单限购 |
| valid_days | int(11) | 有效期（天） |
| billing_type | varchar(20) | 计费方式：one_time(一次性，如门票)/hourly(按小时，如讲解器)/daily(按天，如婴儿车) |
| age_limit_min | int(11) | 年龄下限（硬限制——SKU 级别快速校验，如老人票 age≥60） |
| age_limit_max | int(11) | 年龄上限（硬限制——SKU 级别快速校验） |
| status | tinyint(1) | 0下架/1上架 |

> **年龄校验优先级：** SKU 的 age_limit_min/max 作为**第一层硬限制**（在订单预览阶段即拦截）。rule_sale(rule_type=age_limit) 作为**第二层可配置规则**（用于复杂场景，如组合票的跨年龄限制）。两者不冲突：SKU 先判断，通过后再走 rule_sale 校验。如果 SKU 字段为 NULL 表示无限，仅靠 rule_sale 限制。
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_sku_code, idx_spu_id, idx_status

#### product_combo（组合产品）

> 组合产品本质上是 SPU 的一种形态，通过 spu_id 关联 product_spu，可复用价格日历、渠道定价、多语言等能力。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| spu_id | varchar(32) | 关联 product_spu.id（组合产品可出现在商品搜索列表中） |
| combo_code | varchar(50) | 组合编码，唯一索引 |
| combo_name | varchar(200) | 组合名称 |
| combo_type | varchar(20) | cross_category/cross_merchant/cross_scenic |
| total_price | decimal(10,2) | 组合总价（缓存字段，由 combo_item 实时求和，只读不维护） |
| description | text | 组合说明 |
| usage_rule | text | 使用规则（JSON） |
| status | tinyint(1) | 0下架/1上架 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_combo_code, idx_spu_id, idx_combo_type

#### product_combo_item（组合产品明细）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| combo_id | varchar(32) | 组合产品 ID |
| sku_id | varchar(32) | 关联 SKU ID |
| quantity | int(11) | 数量 |
| sell_price | decimal(10,2) | 分项售价 |
| settle_price | decimal(10,2) | 分项结算价（清分基准） |
| verify_rule | varchar(200) | 核销规则（JSON） |
| required | tinyint(1) | 是否必选：0否/1是 |
| sort_no | int(11) | 排序 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_combo_id

#### product_price_calendar（价格日历）

> 管理员配置的总库存额度（变更频率低）。实时锁定量在 ticket_stock 中。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| sku_id | varchar(32) | SKU ID |
| date | date | 日期 |
| price | decimal(10,2) | 当日价格 |
| stock | int(11) | 当日库存额度（总库存，由管理员配置） |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_sku_date（联合唯一）, idx_date

#### product_channel_price（渠道定价）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| sku_id | varchar(32) | SKU ID |
| channel | varchar(20) | self_miniapp(小程序)/self_pc(PC)/self_kiosk(自助售票机)/agent(旅行社)/ota/窗口(window) |
| price | decimal(10,2) | 渠道价格 |
| stock_ratio | decimal(5,2) | 库存分配比例 |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_sku_channel（联合唯一）

#### product_language（商品多语言内容）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| spu_id | varchar(32) | SPU ID |
| language_code | varchar(10) | 语言代码 |
| title | varchar(200) | 多语言商品名称 |
| description | text | 多语言商品描述 |
| notice | text | 多语言购买须知 |
| refund_policy | text | 多语言退改规则 |
| inclusions | text | 多语言费用包含 |
| exclusions | text | 多语言费用不含 |
| highlight | text | 多语言亮点 |
| how_to_use | text | 多语言使用方法 |
| additional_info | text | 多语言附加信息 |
| status | tinyint(1) | 0待翻译/1已发布 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_spu_language（联合唯一），idx_language_code

#### product_option_slot（商品选项/时段）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| spu_id | varchar(32) | SPU ID |
| option_code | varchar(50) | 选项编码 |
| option_type | varchar(20) | option / time_slot |
| value_code | varchar(50) | 选项值编码 |
| value_name | varchar(200) | 选项值名称 |
| value_name_i18n | text | 多语言名称（JSON） |
| sort_no | int(11) | 排序 |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_spu_option_value（联合唯一），idx_option_code

---

### 3.5 订单服务 — order_db

#### order_main（订单主表）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| order_no | varchar(50) | 订单编号，唯一索引 |
| user_id | varchar(32) | 游客用户 ID，关联 tourist_user.id |
| agent_id | varchar(32) | 旅行社 ID（channel=agent 时必填） |
| order_type | varchar(20) | ticket/combo/hotel/catering/cultural/study/performance/rental |
| channel | varchar(20) | online/window/ota/agent/merchant |
| distribution_channel | varchar(50) | OTA 分销渠道：meituan/ctrip/qunar/douyin/xiaohongshu/feizhu/linktivity，自营为空 |
| channel_order_no | varchar(100) | 第三方单号 |
| total_amount | decimal(10,2) | 订单总金额 |
| discount_amount | decimal(10,2) | 优惠金额 |
| paid_amount | decimal(10,2) | 实付金额 |
| deposit_amount | decimal(10,2) | 押金金额（order_type=rental 时使用，退还时走 order_refund） |
| status | tinyint(1) | 1待支付/2已支付/3已核销/4已退款/5已取消/6部分退款/7已关闭 |
| expire_time | datetime | 支付超时时间（创建时间+30分钟），定时任务扫描超时自动取消释放库存 |
| contact_name | varchar(100) | 联系人 |
| contact_phone | varchar(50) | 联系电话（AES 加密） |
| travel_date | date | 出行日期 |
| refundable | tinyint(1) | 是否可退：1是/0否 |
| insurance_flag | tinyint(1) | 是否含保险：0否/1是 |
| insurance_product_id | varchar(32) | 保险产品 SKU ID（insurance_flag=1 时必填，用于存储保费和清分） |
| insurance_amount | decimal(10,2) | 保费金额（冗余，便于对账，insurance_flag=1 时必填） |
| pay_time | datetime | 支付时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_order_no, idx_user_id, idx_status, idx_travel_date

#### order_sub（子订单）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| order_id | varchar(32) | 主订单 ID |
| sub_order_no | varchar(50) | 子订单编号，唯一索引 |
| product_type | varchar(20) | 产品类型 |
| product_id | varchar(32) | SKU ID |
| merchant_id | varchar(32) | 商户 ID（跨商户订单归属判定，自营为空） |
| product_name | varchar(200) | 商品名称 |
| quantity | int(11) | 数量 |
| unit_price | decimal(10,2) | 单价 |
| sub_amount | decimal(10,2) | 子单金额 |
| verify_count | int(11) | 已核销数量 |
| status | tinyint(1) | 1待支付/2已支付/3已核销/4已退款/5部分退款 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_sub_order_no, idx_order_id, idx_status

#### order_refund（退款记录）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| refund_no | varchar(50) | 退款单号，唯一索引 |
| order_id | varchar(32) | 主订单 ID |
| sub_order_id | varchar(32) | 子订单 ID |
| refund_amount | decimal(10,2) | 退款金额 |
| refund_reason | varchar(500) | 退款原因 |
| refund_type | varchar(20) | partial/full/force/deposit_return（押金退还） |
| refund_method | varchar(20) | original_path/manual |
| status | tinyint(1) | 1待审核/2审核通过/3退款中/4已退款/5已拒绝 |
| approve_by | varchar(50) | 审核人 |
| approve_time | datetime | 审核时间 |
| refund_time | datetime | 退款到账时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_refund_no, idx_order_id, idx_status

#### order_payment（支付流水）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| payment_no | varchar(50) | 支付流水号，唯一索引 |
| order_id | varchar(32) | 主订单 ID |
| pay_method | varchar(20) | wechat/alipay/unionpay/bank_card/visa/master/apple_pay/google_pay |
| pay_amount | decimal(10,2) | 支付金额 |
| transaction_id | varchar(100) | 第三方支付流水号 |
| pay_status | tinyint(1) | 1待支付/2支付成功/3支付失败/4已退款 |
| pay_time | datetime | 支付完成时间 |
| callback_raw | text | 回调原始数据 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_payment_no, idx_order_id, idx_transaction_id

---

### 3.6 清分服务 — settlement_db

#### settlement_detail（清分明细）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| settlement_no | varchar(50) | 清分单号，唯一索引 |
| order_id | varchar(32) | 订单 ID |
| sub_order_id | varchar(32) | 子订单 ID |
| product_id | varchar(32) | SKU ID |
| combo_id | varchar(32) | 组合产品 ID（如有） |
| merchant_id | varchar(32) | 商户 ID |
| channel | varchar(20) | 渠道（冗余自 order_main，避免跨库 JOIN 做分渠道清算） |
| total_amount | decimal(10,2) | 订单金额 |
| commission_rate | decimal(5,4) | 佣金比例 |
| commission_amount | decimal(10,2) | 佣金金额 |
| settle_amount | decimal(10,2) | 结算金额 |
| settle_date | date | 结算日期 |
| settle_status | tinyint(1) | 0待结算/1已结算/2已打款 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_settlement_no, idx_order_id, idx_merchant_id, idx_settle_date

#### settlement_ledger（对账单）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| ledger_no | varchar(50) | 对账单号，唯一索引 |
| merchant_id | varchar(32) | 商户 ID |
| period_start | date | 账期开始 |
| period_end | date | 账期结束 |
| total_orders | int(11) | 总订单数 |
| total_amount | decimal(10,2) | 总金额 |
| total_commission | decimal(10,2) | 总佣金 |
| total_settle | decimal(10,2) | 总结算额 |
| status | tinyint(1) | 0待确认/1已确认/2已开票/3已打款 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_ledger_no, idx_merchant_id, idx_period

#### settlement_nc_voucher（用友 NC 凭证）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| voucher_no | varchar(50) | 凭证号 |
| nc_biz_id | varchar(100) | 用友 NC 业务 ID，唯一索引 |
| settlement_id | varchar(32) | 清分明细 ID |
| voucher_type | varchar(20) | 凭证类型 |
| debit_account | varchar(50) | 借方科目 |
| credit_account | varchar(50) | 贷方科目 |
| amount | decimal(10,2) | 金额 |
| voucher_date | date | 凭证日期 |
| sync_status | tinyint(1) | 0待同步/1同步中/2已同步/3同步失败 |
| sync_time | datetime | 同步时间 |
| sync_error | text | 同步错误信息 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_voucher_no, uk_nc_biz_id, idx_settlement_id, idx_sync_status

---

### 3.7 发票服务 — invoice_db

#### invoice_apply（发票申领）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| apply_no | varchar(50) | 申领单号，唯一索引 |
| order_id | varchar(32) | 订单 ID |
| user_id | varchar(32) | 用户 ID |
| invoice_type | varchar(20) | personal/enterprise |
| invoice_title | varchar(200) | 发票抬头 |
| tax_no | varchar(50) | 税号（企业） |
| category | varchar(20) | 商品/服务类目 |
| amount | decimal(10,2) | 开票金额 |
| email | varchar(100) | 接收邮箱 |
| status | tinyint(1) | 0待开具/1已开具/2已冲红/3已作废 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_apply_no, idx_order_id, idx_status

#### invoice_record（开具记录）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| apply_id | varchar(32) | 申领 ID |
| invoice_no | varchar(50) | 发票号码，唯一索引 |
| invoice_code | varchar(50) | 发票代码 |
| tax_platform_no | varchar(100) | 税控平台流水号 |
| invoice_url | varchar(500) | 发票文件 URL |
| invoice_content | text | 发票内容（JSON） |
| issue_type | varchar(20) | develop/red |
| issue_time | datetime | 开具时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_invoice_no, idx_apply_id

---

### 3.8 人脸服务 — face_db

#### face_image（人像信息）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| user_id | varchar(32) | 用户 ID |
| image_url | varchar(500) | 图片 URL（AES 加密路径） |
| image_hash | varchar(100) | 图片哈希 |
| collect_channel | varchar(20) | miniapp/kiosk/window/gate |
| quality_score | decimal(5,2) | 质量分 |
| status | tinyint(1) | 0待审核/1已入库/2已失效 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_user_id, idx_status

#### face_lib（人脸底库同步记录）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| face_image_id | varchar(32) | 人像 ID |
| target_device | varchar(100) | 目标终端（闸机/手持机编号） |
| sync_status | tinyint(1) | 0待同步/1同步中/2已同步/3同步失败 |
| sync_time | datetime | 同步时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_face_image_id, idx_sync_status

#### face_verify_log（人脸识别记录）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| face_image_id | varchar(32) | 人像 ID |
| device_id | varchar(32) | 终端 ID |
| device_type | varchar(20) | gate/handheld |
| score | decimal(5,2) | 比对分数 |
| threshold | decimal(5,2) | 阈值 |
| result | tinyint(1) | 1通过/0不通过 |
| liveness_result | tinyint(1) | 1活体/0非活体 |
| cost_time | int(11) | 耗时（毫秒） |
| verify_time | datetime | 识别时间 |
| create_time | datetime | 创建时间 |

**索引：** idx_face_image_id, idx_device_id, idx_verify_time

---

### 3.9 商户服务 — merchant_db

#### merchant_info（商户信息）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| merchant_code | varchar(50) | 商户编码，唯一索引 |
| merchant_name | varchar(200) | 商户名称 |
| area_id | varchar(32) | 所属景区 ID，关联 scenic_area.id |
| contact_name | varchar(100) | 联系人 |
| contact_phone | varchar(50) | 联系电话 |
| business_license | varchar(200) | 营业执照 URL |
| category | varchar(50) | 业态分类 |
| status | tinyint(1) | 0待审核/1正常/2禁用/3驳回 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_merchant_code, idx_status, idx_category

#### merchant_shop（商户店铺）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| merchant_id | varchar(32) | 商户 ID |
| area_id | varchar(32) | 所属景区 ID |
| shop_name | varchar(200) | 店铺名称 |
| shop_logo | varchar(500) | 店铺 Logo |
| shop_desc | text | 店铺描述 |
| status | tinyint(1) | 0关闭/1营业 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_merchant_id, idx_status

#### merchant_commission（佣金配置）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| merchant_id | varchar(32) | 商户 ID |
| category_code | varchar(50) | 业态编码 |
| commission_rate | decimal(5,4) | 佣金比例 |
| effective_date | date | 生效日期 |
| status | tinyint(1) | 0失效/1有效 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_merchant_category（联合唯一：merchant_id + category_code）

#### shop_page（店铺装修页面）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| shop_id | varchar(32) | 店铺 ID |
| page_code | varchar(50) | 页面编码，唯一索引 |
| page_name | varchar(200) | 页面名称 |
| page_type | varchar(20) | home/detail/activity |
| page_config | longtext | 页面配置（JSON） |
| is_published | tinyint(1) | 0草稿/1已发布 |
| publish_time | datetime | 发布时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_page_code, idx_shop_id

---

### 3.10 营销服务 — marketing_db

#### marketing_coupon（优惠券模板）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| coupon_code | varchar(50) | 券模板编码，唯一索引 |
| coupon_name | varchar(200) | 券名称 |
| coupon_type | varchar(20) | discount/fixed/cash |
| coupon_value | decimal(10,2) | 面值/折扣率 |
| min_amount | decimal(10,2) | 使用门槛 |
| total_count | int(11) | 发放总量 |
| used_count | int(11) | 已领取数量 |
| per_user_limit | int(11) | 每人限领 |
| product_scope | varchar(20) | all/指定业态/指定商品 |
| product_ids | text | 适用商品 ID（JSON 数组） |
| effective_days | int(11) | 领取后有效天数 |
| start_time | datetime | 活动开始时间 |
| end_time | datetime | 活动结束时间 |
| status | tinyint(1) | 0下架/1上架/2已过期 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_coupon_code, idx_status, idx_start_time, idx_end_time

#### marketing_coupon_use（优惠券使用记录）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| coupon_id | varchar(32) | 券模板 ID |
| user_id | varchar(32) | 用户 ID |
| order_id | varchar(32) | 使用订单 ID |
| coupon_value | decimal(10,2) | 抵扣金额 |
| status | tinyint(1) | 0已领取/1已使用/2已过期 |
| receive_time | datetime | 领取时间 |
| expire_time | datetime | 券到期时间（receive_time + effective_days），可走索引过滤 |
| use_time | datetime | 使用时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

#### marketing_seckill、marketing_group_buy、marketing_distribution、marketing_commission — 见下文

#### marketing_seckill（秒杀活动）— marketing_db

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| seckill_code | varchar(50) | 秒杀编码，唯一索引 |
| sku_id | varchar(32) | 商品 SKU ID |
| seckill_price | decimal(10,2) | 秒杀价 |
| stock | int(11) | 秒杀库存 |
| sold_count | int(11) | 已售数量 |
| per_user_limit | int(11) | 每人限购 |
| start_time | datetime | 开始时间 |
| end_time | datetime | 结束时间 |
| status | tinyint(1) | 0未开始/1进行中/2已结束 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_seckill_code, idx_sku_id, idx_start_time

#### marketing_group_buy（拼团活动）— marketing_db

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| group_code | varchar(50) | 拼团编码，唯一索引 |
| sku_id | varchar(32) | 商品 SKU ID |
| group_price | decimal(10,2) | 拼团价 |
| min_count | int(11) | 成团人数 |
| expire_hours | int(11) | 成团有效时间（小时） |
| start_time | datetime | 活动开始时间 |
| end_time | datetime | 活动结束时间 |
| status | tinyint(1) | 0未开始/1进行中/2已结束 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_group_code, idx_sku_id, idx_start_time

#### marketing_distribution（分销配置）— marketing_db

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| user_id | varchar(32) | 分销员用户 ID |
| share_code | varchar(50) | 推广码，唯一索引 |
| share_link | varchar(500) | 推广链接 |
| status | tinyint(1) | 0待审核/1通过/2驳回 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_share_code, idx_user_id

#### marketing_commission（分销佣金记录）— marketing_db

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| distributor_id | varchar(32) | 分销员 ID |
| order_id | varchar(32) | 来源订单 ID |
| product_id | varchar(32) | 商品 SKU ID |
| order_amount | decimal(10,2) | 订单金额 |
| commission_rate | decimal(5,4) | 佣金比例 |
| commission_amount | decimal(10,2) | 佣金金额 |
| status | tinyint(1) | 0待结算/1已结算/2已提现 |
| settle_time | datetime | 结算时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_distributor_id, idx_order_id, idx_status

---

### 3.11 会员服务 — member_db

#### member_info（会员信息）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| user_id | varchar(32) | 用户 ID，唯一索引 |
| member_no | varchar(50) | 会员编号，唯一索引 |
| level | varchar(20) | BRONZE/SILVER/GOLD/PLATINUM |
| total_points | int(11) | 总积分 |
| available_points | int(11) | 可用积分 |
| total_consumption | decimal(10,2) | 累计消费金额 |
| birthday | date | 生日 |
| status | tinyint(1) | 1正常/0冻结 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_user_id, uk_member_no, idx_level

#### member_tag（会员标签）

> 从 member_info.tags 逗号字符串拆出，支持索引查询。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| member_id | varchar(32) | 会员 ID |
| tag | varchar(50) | 标签（如"亲子游"、"高频消费"） |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_member_tag（联合唯一），idx_tag

#### member_points_log（积分明细）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| member_id | varchar(32) | 会员 ID |
| points | int(11) | 积分数量（+获得/-消耗） |
| scene | varchar(50) | consumption/sign_in/interaction/exchange/deduction/lottery |
| ref_id | varchar(32) | 关联业务 ID |
| description | varchar(255) | 说明 |
| expire_date | date | 积分到期日期（计入时计算：当年12月31日或按规则） |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_member_id, idx_scene, idx_expire_date, idx_create_time

#### member_benefits（会员权益配置）— 结构从略

---

### 3.12 进销存服务 — inventory_db

#### inv_supplier（供应商）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| supplier_code | varchar(50) | 供应商编码，唯一索引 |
| supplier_name | varchar(200) | 供应商名称 |
| contact_name | varchar(100) | 联系人 |
| contact_phone | varchar(50) | 联系电话 |
| address | varchar(500) | 地址 |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_supplier_code

#### inv_goods（商品/物料）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| goods_code | varchar(50) | 商品条码，唯一索引 |
| goods_name | varchar(200) | 商品名称 |
| category_id | varchar(32) | 商品类别 |
| supplier_id | varchar(32) | 供应商 ID |
| spec | varchar(200) | 规格 |
| unit | varchar(20) | 单位 |
| cost_price | decimal(10,2) | 成本价 |
| sell_price | decimal(10,2) | 售价 |
| low_stock_alert | int(11) | 低库存预警线 |
| status | tinyint(1) | 0下架/1上架 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_goods_code, idx_category_id, idx_supplier_id

#### inv_stock（库存）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| goods_id | varchar(32) | 商品 ID |
| store_id | varchar(32) | 门店 ID |
| quantity | int(11) | 当前库存数量 |
| frozen_quantity | int(11) | 冻结数量 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_goods_store（联合唯一）

#### inv_inbound（入库单）— 不含 unique_code，明细独立

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| inbound_no | varchar(50) | 入库单号，唯一索引 |
| supplier_id | varchar(32) | 供应商 ID |
| store_id | varchar(32) | 目标门店 ID |
| total_amount | decimal(10,2) | 采购总金额（由明细求和） |
| inbound_time | datetime | 入库时间 |
| operator | varchar(50) | 经手人 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_inbound_no, idx_inbound_time

#### inv_inbound_item（入库明细，一物一码）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| inbound_id | varchar(32) | 入库单 ID |
| goods_id | varchar(32) | 商品 ID |
| unique_code | varchar(100) | 唯一码（一物一码），唯一索引 |
| unit_price | decimal(10,2) | 采购单价 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_unique_code, idx_inbound_id

#### inv_outbound（出库/调拨单）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| outbound_no | varchar(50) | 出库单号，唯一索引 |
| outbound_type | varchar(20) | sale(销售)/transfer(调拨)/internal_use(内部领用) |
| from_store_id | varchar(32) | 来源门店 ID |
| to_store_id | varchar(32) | 目标门店 ID（调拨时） |
| goods_id | varchar(32) | 商品 ID |
| quantity | int(11) | 出库数量 |
| ref_order_id | varchar(32) | 关联销售单/收银单 ID（outbound_type=sale 时必填） |
| outbound_time | datetime | 出库时间 |
| operator | varchar(50) | 经手人 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_outbound_no, idx_goods_id, idx_outbound_time

#### inv_check（盘点单）、inv_check_item（盘点明细）、inv_checkout（收银单）、inv_checkout_item（收银明细）— 结构从略

---

### 3.12a 租赁服务 — rental_db

> 租赁业务独立于普通电商商品。核心区别：实物设备一进一出，需追踪每件物品的实时状态，而非库存数量模型。

#### rental_device（租赁设备台账）

> 每件实物设备一行，追踪唯一状态。贴二维码/条码标签。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| device_no | varchar(50) | 设备编号（贴码），唯一索引 |
| sku_id | varchar(32) | 关联 product_sku.id（如"婴儿车-标准型"、"讲解器-多语言版"） |
| area_id | varchar(32) | 所属景区 ID |
| location | varchar(100) | 归还点/存放位置（如"南口租赁站"） |
| status | tinyint(1) | 0空闲/1租借中/2维修/3报废 |
| last_order_id | varchar(32) | 最后一笔租借订单 ID |
| purchase_date | date | 购置日期 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_device_no, idx_sku_status（联合: sku_id + status）

#### rental_order（租赁订单）

> 独立于 order_main 的租赁专属订单。含押金、计时计费、逾期费。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| rental_no | varchar(50) | 租赁单号，唯一索引 |
| order_id | varchar(32) | 关联 order_main.id（主支付单，含租金+押金） |
| user_id | varchar(32) | 游客 ID，关联 tourist_user.id |
| device_id | varchar(32) | 租赁设备 ID，关联 rental_device.id |
| sku_id | varchar(32) | 设备类型 SKU ID |
| rent_start_time | datetime | 开始租借时间（扫码取设备时） |
| rent_end_time | datetime | 实际归还时间 |
| expected_return_time | datetime | 预计归还时间（取设备时按计费规则计算） |
| fee_type | varchar(20) | hourly（按小时）/ daily（按天）/ fixed（固定） |
| unit_price | decimal(10,2) | 计费单价 |
| rent_fee | decimal(10,2) | 租赁费（归还时结算） |
| deposit_amount | decimal(10,2) | 押金金额（支付时一并收取） |
| deposit_refund_amount | decimal(10,2) | 实际退还押金（设备损坏可扣减） |
| overdue_fee | decimal(10,2) | 逾期费（超期累加） |
| status | tinyint(1) | 1租借中/2已归还/3逾期/4设备损坏/5已结算 |
| return_location | varchar(100) | 归还点 |
| damage_remark | varchar(500) | 损坏说明 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_rental_no, idx_user_id, idx_device_id, idx_status, idx_rent_start_time

> **流程：** 游客在线预约/现场扫码 → 创建 order_main(order_type=rental, deposit_amount) → 支付租金+押金 → 创建 rental_order(status=1) → 更新 rental_device(status=1租借中) → 归还时扫码 → 结算 rent_fee + overdue_fee → 原路退还 deposit_refund_amount → rental_device(status=0空闲)
> 
> **押金退还逻辑：** 无损归还 → deposit_refund_amount = deposit_amount。设备损坏 → deposit_refund_amount = deposit_amount - 扣款（人工核定）。退款类型标记 `refund_type=deposit_return`。

---

### 3.13 客流服务 — traffic_db

#### traffic_store_flow（店铺客流明细）

> 高写入频率表。使用 BIGINT 自增主键避免 B+ 树页分裂。按月分区，历史数据定期归档。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键，AUTO_INCREMENT |
| store_id | varchar(32) | 店铺 ID |
| device_id | varchar(32) | 客流计数器设备 ID |
| enter_count | int(11) | 进店人数 |
| exit_count | int(11) | 出店人数 |
| current_count | int(11) | 实时在店人数 |
| record_time | datetime | 记录时间 |
| create_time | datetime | 创建时间 |

**索引：** idx_store_time（联合: store_id + record_time）, idx_device_id
**分区：** PARTITION BY RANGE (TO_DAYS(record_time)) 按月分区

#### traffic_store_view（店铺客流汇总）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键，AUTO_INCREMENT |
| store_id | varchar(32) | 店铺 ID |
| stat_date | date | 统计日期 |
| total_enter | int(11) | 总进店数 |
| total_exit | int(11) | 总出店数 |
| peak_hour | tinyint(2) | 高峰时段 |
| peak_count | int(11) | 高峰人数 |
| avg_duration | int(11) | 平均停留时长（分钟） |
| total_sales_amount | decimal(10,2) | 当日 POS 销售额（客流-销售联动分析） |
| create_time | datetime | 创建时间 |

**索引：** uk_store_date（联合唯一）

---

### 3.14 外部对接服务 — integration_db

#### ota_product_mapping（OTA 商品映射）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| sku_id | varchar(32) | 内部 SKU ID |
| ota_code | varchar(20) | ctrip/meituan/qunar/douyin/xiaohongshu/feizhu/linktivity |
| ota_option_id | varchar(200) | OTA optionId |
| ota_supplier_option_id | varchar(200) | OTA PLU |
| ota_product_id | varchar(200) | OTA 产品 ID |
| contract_id | varchar(100) | OTA 合同 ID |
| date_type | varchar(20) | DATE_REQUIRED / DATE_NOT_REQUIRED / FREE_SALE |
| sync_price | tinyint(1) | 0否/1是 |
| sync_inventory | tinyint(1) | 0否/1是 |
| push_enabled | tinyint(1) | 0否/1是 |
| last_sync_time | datetime | 最后同步时间 |
| status | tinyint(1) | 0未上线/1已上线 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_sku_ota（联合唯一）, idx_ota_code, idx_status

#### ota_product_mapping_log（OTA 映射变更日志）

> 审计报告 #26 要求。每次映射变更写入历史，对账时可还原当时映射状态。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| mapping_id | varchar(32) | 映射 ID |
| operation | varchar(20) | create/update/delete |
| change_snapshot | text | 变更内容（JSON diff） |
| operator | varchar(50) | 操作人 |
| create_time | datetime | 操作时间 |
| del_flag | tinyint(1) | 0正常/1删除 |
#### ota_voucher（OTA 凭证）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| voucher_no | varchar(50) | 凭证编号，唯一索引 |
| order_id | varchar(32) | 关联订单 ID |
| passenger_id | varchar(32) | 游客信息 ID |
| ota_code | varchar(20) | OTA 编码 |
| ota_voucher_id | varchar(200) | OTA 侧凭证 ID |
| voucher_type | varchar(20) | text/code/html/image_base64/pdf_base64/url/barcode_128/barcode_417 |
| voucher_content | longtext | 凭证内容 |
| voucher_status | varchar(20) | pending/sent/consumed/expired |
| seat_info | varchar(200) | 座位信息（演艺票用） |
| send_time | datetime | 凭证发送时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_voucher_no, idx_order_id, idx_ota_voucher_id

#### ota_sync_log、ota_reconciliation

#### ota_reconciliation_item（对账差异明细）

> 审计报告 #25 要求。补充行级差异明细，用于逐单核查。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| reconciliation_id | varchar(32) | 对账单 ID |
| channel_order_no | varchar(100) | OTA 订单号 |
| local_order_no | varchar(50) | 本地订单号 |
| ota_amount | decimal(10,2) | OTA 侧金额 |
| local_amount | decimal(10,2) | 本地侧金额 |
| diff_amount | decimal(10,2) | 差异金额 |
| diff_type | varchar(50) | ota_only(OTA有本地无)/local_only(本地有OTA无)/amount_mismatch(金额不一致) |
| status | tinyint(1) | 0待处理/1已处理/2已忽略 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_reconciliation_id, idx_status

---

### 3.15 物业管理 — scenic_db

#### biz_property_space（物业空间表）

> 审计报告 #21 要求。景区拥有的物理物业空间（铺位），租约签约的对象。

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| space_code | varchar(50) | 铺位编号，唯一索引 |
| space_name | varchar(200) | 铺位名称 |
| area_id | varchar(32) | 所属景区 ID |
| location | varchar(200) | 位置描述（如商业街 B 区 3号） |
| area_sqm | decimal(8,2) | 面积（平方米） |
| floor | varchar(20) | 楼层 |
| status | tinyint(1) | 0空闲/1已租/2维修 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_space_code, idx_area_id, idx_status

#### biz_shop_lease（商铺租约）— shop_id 关联 biz_property_space.id，而非 merchant_shop.id
#### biz_rent_record（租金收缴记录）、biz_property_fee（物业费）— 结构从略

---

### 3.16 车辆服务 — vehicle_db

#### vehicle_info、vehicle_gps_log、vehicle_route_alert — 见下文

#### vehicle_info（车辆信息）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| vehicle_code | varchar(50) | 车辆编号，唯一索引 |
| vehicle_name | varchar(100) | 车辆名称 |
| plate_no | varchar(20) | 车牌号 |
| vehicle_type | varchar(20) | 车辆类型 |
| device_id | varchar(32) | 车载终端 ID |
| status | tinyint(1) | 0离线/1在线/2维修 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_vehicle_code

#### vehicle_gps_log（GPS 定位记录）— BIGINT 自增，按月分区，同 IoT 优化策略

| 列名 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键，AUTO_INCREMENT |
| vehicle_id | varchar(32) | 车辆 ID |
| longitude | decimal(12,8) | 经度 |
| latitude | decimal(12,8) | 纬度 |
| speed | decimal(6,2) | 速度 |
| direction | int(11) | 方向角 |
| gps_time | datetime | 定位时间 |
| create_time | datetime | 创建时间 |

**索引：** idx_vehicle_gps_time（联合），PARTITION BY RANGE (TO_DAYS(gps_time))

#### vehicle_route_alert（路线异常告警）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| vehicle_id | varchar(32) | 车辆 ID |
| alert_type | varchar(20) | route_deviation/over_speed/long_stop |
| alert_detail | text | 告警详情（JSON） |
| gps_log_id | varchar(32) | 对应的 GPS 记录 ID |
| status | tinyint(1) | 0未处理/1已处理 |
| operator | varchar(50) | 处理人 |
| create_time | datetime | 创建时间 |

**索引：** idx_vehicle_id, idx_alert_type

> vehicle_gps_log 使用 BIGINT 自增主键，按月分区，同 IoT 表优化策略。结构从略。

---

### 3.17 内容服务 — content_db

#### content_article、content_note — 见下文

#### content_article（资讯/公告文章）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| title | varchar(200) | 标题 |
| content | longtext | 内容 |
| category | varchar(20) | news/culture/notice/activity |
| cover_image | varchar(500) | 封面图 |
| author | varchar(100) | 作者 |
| publish_time | datetime | 发布时间 |
| status | tinyint(1) | 0草稿/1已发布/2已下架 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_category, idx_publish_time, idx_status

#### content_note（游客笔记/攻略）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| user_id | varchar(32) | 发布用户 ID |
| title | varchar(200) | 标题 |
| content | longtext | 内容 |
| images | text | 图片列表（JSON） |
| tags | varchar(200) | 标签 |
| like_count | int(11) | 点赞数 |
| collect_count | int(11) | 收藏数 |
| share_count | int(11) | 分享数 |
| audit_status | tinyint(1) | 0待审核/1通过/2驳回 |
| is_top | tinyint(1) | 0不置顶/1置顶 |
| is_essence | tinyint(1) | 0不加精/1加精 |
| publish_time | datetime | 发布时间 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_user_id, idx_audit_status, idx_publish_time

#### content_audio_guide（语音讲解）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| scenic_spot_id | varchar(32) | 景点 ID，关联 scenic_spot.id |
| audio_url | varchar(500) | 音频 URL |
| transcript | text | 讲解文字稿 |
| language | varchar(10) | 语言 |
| duration | int(11) | 时长（秒） |
| sort_no | int(11) | 排序 |
| status | tinyint(1) | 0下架/1上架 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_scenic_spot_id, idx_language, idx_sort_no

#### content_hand_map（手绘地图标注）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| marker_name | varchar(100) | 标注名称 |
| marker_type | varchar(20) | scenic/toilet/restaurant/parking/cable_car/facility |
| longitude | decimal(12,8) | 经度 |
| latitude | decimal(12,8) | 纬度 |
| icon_url | varchar(500) | 图标 URL |
| description | varchar(500) | 描述 |
| sort_no | int(11) | 排序 |
| status | tinyint(1) | 0隐藏/1显示 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** idx_marker_type, idx_status

#### content_route（导览路线）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| route_code | varchar(50) | 路线编码，唯一索引 |
| route_name | varchar(200) | 路线名称 |
| area_id | varchar(32) | 所属景区 ID |
| route_type | varchar(20) | recommend/family/senior/quick |
| waypoints | longtext | 路线节点（JSON） |
| total_distance | int(11) | 总距离（米） |
| estimated_duration | int(11) | 预估时长（分钟） |
| description | text | 路线介绍 |
| status | tinyint(1) | 0下架/1上架 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_route_code, idx_area_id, idx_route_type

#### content_interaction（UGC 互动记录）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| user_id | varchar(32) | 操作用户 ID |
| target_type | varchar(20) | note/article/audio_guide |
| target_id | varchar(32) | 目标 ID |
| interaction_type | varchar(20) | like/collect/share/view |
| status | tinyint(1) | 1有效/0取消 |
| create_time | datetime | 交互时间 |

**索引：** idx_user_target（联合），idx_target_type_id

---

### 3.18 规则与预警 — rule_db + analytics_db

#### rule_sale（销售规则）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| rule_code | varchar(50) | 规则编码，唯一索引 |
| rule_name | varchar(200) | 规则名称 |
| target_type | varchar(20) | spu/sku/combo |
| target_id | varchar(32) | 绑定对象 ID |
| rule_type | varchar(50) | age_limit/id_type_limit/combo_required/channel_limit/buy_limit/user_type_limit |
| rule_params | text | 规则参数（JSON） |
| error_msg | varchar(500) | 触发规则提示 |
| priority | int(11) | 优先级（数字越小越高） |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_rule_code, idx_target_type_id, idx_rule_type, idx_status

#### rule_refund（退票规则）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| rule_code | varchar(50) | 规则编码，唯一索引 |
| rule_name | varchar(200) | 规则名称 |
| target_type | varchar(20) | spu/sku/combo |
| target_id | varchar(32) | 绑定对象 ID |
| refund_time_scope | varchar(20) | before_visit/after_visit/anytime |
| hours_before | int(11) | 游览前 N 小时内 |
| refund_rate | decimal(5,4) | 退款比例（1.0000=全额） |
| allow_partial | tinyint(1) | 0否/1是 |
| cancel_fee_json | text | 携程格式退改费率列表（JSON） |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_rule_code, idx_target_type_id, idx_status

#### rule_verify（验票规则）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| rule_code | varchar(50) | 规则编码，唯一索引 |
| rule_name | varchar(200) | 规则名称 |
| target_type | varchar(20) | spu/sku/scenic_spot/scenic_area |
| target_id | varchar(32) | 绑定对象 ID |
| rule_type | varchar(50) | unique_entry/verify_deadline/offline_allow/gate_whitelist/single_use/re_entry_interval |
| rule_params | text | 规则参数（JSON） |
| error_msg | varchar(500) | 触发规则提示 |
| priority | int(11) | 优先级 |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_rule_code, idx_target_type_id, idx_rule_type, idx_status

#### alert_rule（预警规则）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| alert_code | varchar(50) | 预警编码，唯一索引 |
| alert_name | varchar(200) | 预警名称 |
| alert_type | varchar(20) | visitor_drop/sales_drop/device_fault/rent_arrears/capacity/security |
| target_type | varchar(20) | scenic_area/scenic_spot/device_gate/device_handheld/store |
| target_id | varchar(32) | 监控对象 ID |
| metric_field | varchar(100) | 监控指标字段 |
| compare_type | varchar(20) | absolute/yoy/mom |
| threshold | decimal(10,2) | 阈值 |
| threshold_direction | varchar(10) | below/above |
| notify_channels | varchar(200) | sms,email,wechat,led,weibo |
| notify_roles | varchar(200) | 通知角色 |
| status | tinyint(1) | 0禁用/1启用 |
| create_by | varchar(50) | 创建人 |
| create_time | datetime | 创建时间 |
| update_by | varchar(50) | 更新人 |
| update_time | datetime | 更新时间 |
| del_flag | tinyint(1) | 0正常/1删除 |

**索引：** uk_alert_code, idx_alert_type, idx_status

#### alert_log（预警触发记录）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | varchar(32) | 主键 |
| alert_id | varchar(32) | 预警规则 ID |
| trigger_value | decimal(10,2) | 触发时的实际值 |
| threshold_value | decimal(10,2) | 触发时的阈值 |
| notify_result | varchar(20) | success/fail/partial |
| notify_detail | text | 通知详情（JSON） |
| handled_by | varchar(50) | 处理人 |
| handle_time | datetime | 处理时间 |
| handle_remark | varchar(500) | 备注 |
| create_time | datetime | 创建时间 |

**索引：** idx_alert_id, idx_create_time

---

## 4. 表统计

| 类别 | 表数 | 说明 |
|------|------|------|
| JEECG Boot 系统表（保留） | 61 | 用户/角色/权限/组织/字典/日志/消息/文件/任务/网关/多数据源 |
| JEECG Boot 可选表（按需） | 62 | Online 低代码/JimuReport/AI 模块 |
| Nacos + XXL-JOB（独立库） | 11 | 微服务基础设施 |
| **绥中业务表** | **~88** | 按 18 个服务库分布 |
