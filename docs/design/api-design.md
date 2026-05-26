# API 接口设计

> 按消费端分 6 个 API 组，每个组独立成文。后端共享 Service 层，网关层按路由前缀统一拦截鉴权。

---

## API 分组概览

| API 组 | 路由前缀 | 文档 | 端点 | 鉴权 |
|--------|----------|------|------|------|
| 游客端 | `/api/v1/tourist/` | [tourist.md](api/tourist.md) | ~45 | 部分 Token |
| 管理端 | `/api/v1/admin/` | [admin.md](api/admin.md) | ~100 | JWT + RBAC |
| 商户端 | `/api/v1/merchant/` | [merchant.md](api/merchant.md) | ~15 | JWT + 隔离 |
| 旅行社端 | `/api/v1/agent/` | [agent.md](api/agent.md) | ~15 | JWT + 分组 |
| OTA 开放 | `/api/v1/ota/` | [ota.md](api/ota.md) | ~15 | AK/SK + 签名 |
| 公共 | `/api/v1/auth/`, `/api/v1/common/` | [common.md](api/common.md) | ~10 | 部分 |

> **租赁业务（rental）** 接口归属：游客端新增租赁相关端点（可租设备查询/预约/扫码取还 ~5个，计入 tourist ~50），管理端新增设备台账管理/租赁订单查询（~5个，计入 admin ~105）。不单独开设 API 组，由游客端和管理端承载。

## 通用规范

详见 [接口规范](api/conventions.md)，包含：
- 统一响应格式与分页
- 鉴权方式与 Token 刷新
- 敏感信息脱敏规则
- 幂等性要求
- 限流策略与版本管理

## 面向角色

```
游客端 ─── 小程序/H5/PC购票  ── 查询无需Token，下单/个人中心需要
           （普通游客、年卡用户、分销员）

管理端 ─── PC管理后台  ────────── JWT + RBAC权限
           （运营管理者、售票员、检票员、财务、超级管理员）

商户端 ─── 商户H5/小程序 ──────── 商户Token + 数据隔离
           （入驻商户管理员）

旅行社端 ── 旅行社PC/H5/小程序 ── 旅行社Token + 分组权限
           （旅行社操作员）

OTA开放 ── 携程/美团/去哪儿 ────── AK/SK + AES签名
           （OTA平台系统间对接）
```
