# 预警与监控 — `/api/v1/admin/alerts`

### 12.1 预警规则

**GET/POST** `/api/v1/admin/alerts/rules`

| 字段 | 必填 | 说明 |
|------|------|------|
| alertCode | 是 | 编码 |
| alertName | 是 | "游客量骤降预警" |
| alertType | 是 | visitor_drop / sales_drop / device_fault / capacity / ... |
| targetType | 是 | 监控对象类型 |
| targetId | 是 | 监控对象 ID |
| metricField | 是 | 监控字段 |
| compareType | 是 | absolute / yoy / mom |
| threshold | 是 | 阈值 |
| thresholdDirection | 是 | below / above |
| notifyChannels | 是 | sms,email,wechat,led,weibo |
| notifyRoles | 是 | 通知角色 |

**POST** `/api/v1/admin/alerts/rules/{id}/test` — 测试发送（向当前用户发送模拟预警通知，验证通知渠道配置）

### 12.2 预警日志

**GET** `/api/v1/admin/alerts/logs?alertId=X&startTime=Y&endTime=Z`

---

