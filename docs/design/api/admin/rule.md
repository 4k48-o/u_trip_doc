# 规则管理 — `/api/v1/admin/rules`

> **架构说明：** 独立 rule_* 表优先级高于产品内联字段。同一商品同时有内联字段（如 product_sku.age_limit_min）和独立规则（如 rule_sale rule_type=age_limit）时，SKU 硬限制先校验（第一层），rule_sale 规则后校验（第二层）。内联字段为产品自身约束，独立规则为可配置策略，两者叠加求交集。

### 11.1 销售规则

**GET** `/api/v1/admin/rules/sale`

| 参数 | 必填 | 说明 |
|------|------|------|
| targetType | 否 | spu / sku / combo |
| ruleType | 否 | age_limit/id_type_limit/combo_required/channel_limit/buy_limit/user_type_limit |
| targetId | 否 | 绑定对象 ID |
| status | 否 | 0禁用/1启用 |
| keyword | 否 | 规则名称/编码 |
| pageNo | 否 | 默认 1 |
| pageSize | 否 | 默认 20 |

**响应records字段：** ruleId, ruleCode, ruleName, targetType, targetId, targetName, ruleType, ruleTypeText, ruleParams, errorMsg, priority, status, statusText, createTime。

**GET** `/api/v1/admin/rules/sale/{ruleId}` — 详情

**POST** `/api/v1/admin/rules/sale` — 创建

| 字段 | 必填 | 说明 |
|------|------|------|
| ruleCode | 是 | 编码 |
| ruleName | 是 | 规则名 |
| targetType | 是 | spu/sku/combo（无 all 全局规则，需逐SKU绑定） |
| targetId | 是 | 绑定对象 ID |
| ruleType | 是 | age_limit/id_type_limit/combo_required/channel_limit/buy_limit/user_type_limit |
| ruleParams | 是 | 参数 JSON（按 ruleType 格式见下表） |
| errorMsg | 是 | 触发提示 |
| priority | 是 | 数字越小越高 |
| status | 是 | 0/1 |

**ruleType → ruleParams 格式对照表：**

| ruleType | ruleParams 格式 | 说明 |
|----------|-----------------|------|
| age_limit | `{"min_age":18,"max_age":60}` | 年龄范围限制 |
| id_type_limit | `{"blocked_id_types":["PASSPORT"]}` | 禁止指定证件类型 |
| combo_required | `{"combo_spu_id":"SPU_XXX"}` | 必须组合购买 |
| channel_limit | `{"blocked_channels":["ota","agent"]}` | 禁止指定渠道（与 product_channel_price/product_option_slot 互斥形成矛盾须校验） |
| buy_limit | `{"max_per_user":3,"max_per_day":1,"scope":"day|total"}` | 限购 |
| user_type_limit | `{"allowed_user_types":["member"],"min_level":"SILVER"}` | 用户类型+等级限制 |

**PUT** `/api/v1/admin/rules/sale/{ruleId}` — 编辑

**DELETE** `/api/v1/admin/rules/sale/{ruleId}` — 删除

**PUT** `/api/v1/admin/rules/sale/{ruleId}/status` — 启停 `{ "status": 0|1 }`

---

### 11.2 退票规则

**GET** `/api/v1/admin/rules/refund?targetType=X&targetId=Y&status=Z&pageNo=1` — 列表

**GET** `/api/v1/admin/rules/refund/{ruleId}` — 详情

**POST** `/api/v1/admin/rules/refund` — 创建

| 字段 | 必填 | 说明 |
|------|------|------|
| ruleCode | 是 | 编码 |
| ruleName | 是 | 规则名 |
| targetType | 是 | spu / sku / combo |
| targetId | 是 | 绑定对象 ID |
| refundTimeScope | 是 | before_visit / after_visit / anytime |
| hoursBefore | 否 | 游览前 N 小时内 |
| refundRate | 是 | 退款比例（1.0000=全额）— 与 cancelFeeJson 二选一，同时传入以 cancelFeeJson 为准 |
| allowPartial | 是 | 是否允许部分退（布尔） |
| confirmationTime | 否 | 携程退改确认时限（8/16/24 小时） |
| cancelFeeJson | 否 | 携程阶梯费率，格式 `[{"dayBeforeVisitDate":1,"time":"00:00","unit":"PERCENTAGE","value":50}]`，dayBeforeVisitDate=游览前N天，time=截止时间，unit=PERCENTAGE，value=0-100 |

**PUT** `/api/v1/admin/rules/refund/{ruleId}` — 编辑

**DELETE** `/api/v1/admin/rules/refund/{ruleId}` — 删除

**PUT** `/api/v1/admin/rules/refund/{ruleId}/status` — 启停

---

### 11.3 验票规则

**GET** `/api/v1/admin/rules/verify?targetType=X&status=Y&targetId=Z&ruleType=W&keyword=Q&pageNo=1` — 列表

**GET** `/api/v1/admin/rules/verify/{ruleId}` — 详情

**POST** `/api/v1/admin/rules/verify` — 创建

| 字段 | 必填 | 说明 |
|------|------|------|
| ruleCode | 是 | 编码 |
| ruleName | 是 | 规则名 |
| targetType | 是 | spu/sku/scenic_spot/scenic_area |
| targetId | 是 | 绑定对象 ID |
| ruleType | 是 | unique_entry/verify_deadline/offline_allow/gate_whitelist/single_use/re_entry_interval |
| ruleParams | 是 | 参数 JSON（格式见下表） |
| errorMsg | 是 | 触发提示 |
| priority | 是 | 数字越小越高 |

**ruleType → ruleParams 格式对照表：**

| ruleType | ruleParams 格式 |
|----------|-----------------|
| unique_entry | `{"scenic_area_id":"..."}` |
| verify_deadline | `{"start_time":"08:00","end_time":"20:00"}` |
| offline_allow | `{"max_offline_count":10}` |
| gate_whitelist | `{"gate_ids":["G001","G002"]}` |
| single_use | `{}`（无额外参数） |
| re_entry_interval | `{"interval_minutes":240}` |

**PUT** `/api/v1/admin/rules/verify/{ruleId}` — 编辑

**DELETE** `/api/v1/admin/rules/verify/{ruleId}` — 删除

**PUT** `/api/v1/admin/rules/verify/{ruleId}/status` — 启停

---

### 11.4 规则工具

**GET** `/api/v1/admin/rules/product/{targetId}` — 查看某商品/SKU关联的所有规则（跨 rule_sale/rule_refund/rule_verify）

**POST** `/api/v1/admin/rules/simulate` — 规则模拟 `{ "userId":"...", "skuId":"...", "channel":"...", "visitDate":"...", "quantity":1 }`。返回: `{ "passed": true|false, "triggered": [{ "ruleType":"age_limit", "ruleName":"成人票年龄限制", "result":"pass" }], "blocked": [{ "ruleType":"id_type_limit", "ruleName":"禁止护照", "result":"block", "message":"外籍游客请购买全价票" }] }`

**POST** `/api/v1/admin/rules/copy/{ruleId}` — 复制规则（从SKU001复制到SKU002）`{ "targetType":"sku", "targetId":"SKU002" }`

**GET/POST** `/api/v1/admin/rules/export` — 导出/导入规则配置

---

