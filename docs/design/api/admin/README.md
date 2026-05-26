# 管理端 API — `/api/v1/admin/`

> 面向景区运营管理者、财务人员、超级管理员。全部需要 JWT Token + RBAC 角色权限。

## 权限标识

| 权限标识 | 说明 |
|----------|------|
| `admin:product:*` | 商品管理全部权限 |
| `admin:order:*` | 订单管理全部权限 |
| `admin:ticket:*` | 票务管理全部权限 |
| `admin:settlement:*` | 清分结算权限 |
| `admin:member:*` | 会员管理权限 |
| `admin:merchant:*` | 商户管理权限 |
| `admin:marketing:*` | 营销管理权限 |
| `admin:report:*` | 报表查看权限 |
| `admin:system:*` | 系统配置权限（超管） |

> 各端点同时支持 `*.create`、`*.update`、`*.delete`、`*.view` 细粒度权限。

## 子模块索引

| 模块 | 文件 | 核心路由 |
|------|------|----------|
| 景区管理 | [scenic.md](scenic.md) | `/api/v1/admin/scenic` |
| 商品管理 | [product.md](product.md) | `/api/v1/admin/products` |
| 票务管理 | [ticket.md](ticket.md) | `/api/v1/admin/tickets` |
| 订单管理 | [order.md](order.md) | `/api/v1/admin/orders` |
| 清分结算 | [settlement.md](settlement.md) | `/api/v1/admin/settlements` |
| 发票管理 | [invoice.md](invoice.md) | `/api/v1/admin/invoices` |
| 商户管理 | [merchant.md](merchant.md) | `/api/v1/admin/merchants` |
| 营销管理 | [marketing.md](marketing.md) | `/api/v1/admin/marketing` |
| 会员管理 | [member.md](member.md) | `/api/v1/admin/members` |
| 进销存管理 | [inventory.md](inventory.md) | `/api/v1/admin/inventory` |
| 规则管理 | [rule.md](rule.md) | `/api/v1/admin/rules` |
| 预警监控 | [alert.md](alert.md) | `/api/v1/admin/alerts` |
| 设备管理 | [device.md](device.md) | `/api/v1/admin/devices` |
| 旅行社管理 | [travel-agency.md](travel-agency.md) | `/api/v1/admin/travel-agencies` |
| 报表 | [report.md](report.md) | `/api/v1/admin/reports` |
| 年卡管理 | [annual-card.md](annual-card.md) | `/api/v1/admin/annual-cards` |
| 租赁管理 | [rental.md](rental.md) | `/api/v1/admin/rental` |
| AI 知识库 | [ai-knowledge.md](ai-knowledge.md) | `/api/v1/admin/ai` |
| 分销提现 | [distribution-withdraw.md](distribution-withdraw.md) | `/api/v1/admin/marketing/withdrawals` |
| 电子围栏 | [geofence.md](geofence.md) | `/api/v1/admin/geofences` |
| 消息模板 | [notification.md](notification.md) | `/api/v1/admin/notifications` |
| 客服会话 | [customer-service.md](customer-service.md) | `/api/v1/admin/cs` |
| 内容管理 | [content.md](content.md) | `/api/v1/admin/content` |
| OTA 管理 | [ota-admin.md](ota-admin.md) | `/api/v1/admin/ota` |
