# 枚举 / 常量定义

> 本文档定义项目中所有枚举值、状态码、字典值，作为前后端统一的协议标准。

---

## 1. 用户相关

### 用户状态 (UserStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| ACTIVE | 1 | 正常 |
| DISABLED | 0 | 已禁用 |
| DELETED | -1 | 已删除 |

### 用户角色 (UserRole)

| 枚举值 | 说明 | 权限范围 |
|--------|------|----------|
| SUPER_ADMIN | 超级管理员 | 全部权限 |
| ADMIN | 管理员 | 管理后台权限 |
| EDITOR | 编辑者 | 内容管理权限 |
| USER | 普通用户 | 基础用户权限 |
| GUEST | 访客 | 只读权限 |

---

## 2. 业务状态

### 通用状态 (CommonStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| DRAFT | 0 | 草稿 |
| PENDING | 1 | 待审核 |
| APPROVED | 2 | 已通过 |
| REJECTED | 3 | 已驳回 |
| PUBLISHED | 4 | 已发布 |
| ARCHIVED | 5 | 已归档 |

### 订单状态 (OrderStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| UNPAID | 1 | 待支付 |
| PAID | 2 | 已支付 |
| VERIFIED | 3 | 已核销（全部） |
| REFUNDED | 4 | 已退款（全部） |
| CANCELLED | 5 | 已取消 |
| PARTIAL_REFUND | 6 | 部分退款 |
| CLOSED | 7 | 已关闭（超时未支付） |

### 预约状态 (ReservationStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| RESV_PENDING | 1 | 待支付 |
| RESV_PAID | 2 | 已支付 |
| RESV_RESCHEDULED | 3 | 已改签 |
| RESV_CANCELLED | 4 | 已取消 |
| RESV_VERIFIED | 5 | 已核销 |

### 核销状态 (VerifyStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| NOT_VERIFIED | 0 | 未核销 |
| VERIFIED | 1 | 已核销 |
| PARTIAL_VERIFIED | 2 | 部分核销 |

### 支付状态 (PayStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| PAY_PENDING | 1 | 待支付 |
| PAY_SUCCESS | 2 | 支付成功 |
| PAY_FAILED | 3 | 支付失败 |
| PAY_REFUNDED | 4 | 已退款 |

### 退款状态 (RefundStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| REFUND_PENDING | 1 | 待审核 |
| REFUND_APPROVED | 2 | 审核通过 |
| REFUND_PROCESSING | 3 | 退款中 |
| REFUND_COMPLETED | 4 | 已退款 |
| REFUND_REJECTED | 5 | 已拒绝 |

### 渠道类型 (ChannelType)

| 枚举值 | 说明 |
|--------|------|
| online | 线上（小程序等自有渠道聚合） |
| window | 窗口售票 |
| ota | OTA 分销 |
| agent | 旅行社 |
| merchant | 商户 |

### 商户状态 (MerchantStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| MER_AUDITING | 0 | 待审核 |
| MER_ACTIVE | 1 | 正常 |
| MER_DISABLED | 2 | 已禁用 |
| MER_REJECTED | 3 | 已驳回 |

### 旅行社分组 (AgencyGroupType)

| 枚举值 | 说明 |
|--------|------|
| key | 重点旅行社 |
| normal | 普通旅行社 |
| society | 社会团体 |
| restaurant | 周边餐馆 |

### OTA 凭证状态 (VoucherStatus)

| 枚举值 | 说明 |
|--------|------|
| pending | 待发送 |
| sent | 已发送 |
| consumed | 已核销 |
| expired | 已过期 |

### 证件类型 (IdType)

| 枚举值 | 说明 |
|--------|------|
| ID_CARD | 身份证 |
| PASSPORT | 护照 |
| HK_MO | 港澳通行证 |
| TAIWAN | 台湾通行证 |
| PERMANENT_RESIDENCE | 外国人永久居留证 |
| STUDENT_CARD | 学生证 |
| MILITARY_ID | 军官证 |
| DRIVERS_LICENSE | 驾驶证 |

### 年卡状态 (AnnualCardStatus)

| 枚举值 | 数值 | 说明 |
|--------|------|------|
| CARD_ACTIVE | 1 | 正常 |
| CARD_EXPIRED | 0 | 已过期 |
| CARD_REPORTED_LOSS | 2 | 已挂失 |

---

## 3. 系统常量

### 分页

| 常量名 | 值 | 说明 |
|--------|-----|------|
| DEFAULT_PAGE_SIZE | 20 | 默认每页条数 |
| MAX_PAGE_SIZE | 100 | 最大每页条数 |

### 其他限制

| 常量名 | 值 | 说明 |
|--------|-----|------|
| MAX_UPLOAD_SIZE | 10 * 1024 * 1024 | 最大上传文件大小 (10MB) |
| TOKEN_EXPIRE | 7200 | Token 过期时间 (秒) |
| VERIFICATION_CODE_EXPIRE | 300 | 验证码过期时间 (秒) |
| MAX_LOGIN_ATTEMPTS | 5 | 最大登录尝试次数 |

---

## 4. 数据库字典值

### 性别 (gender)

| 值 | 说明 |
|----|------|
| M | 男 |
| F | 女 |
| U | 未知 |

### 布尔字段约定

| 值 | 说明 |
|----|------|
| 0 | 否 / 未 / 关 |
| 1 | 是 / 已 / 开 |

---

## 5. 维护说明

- 新增枚举值时，必须同步更新本文档
- 枚举值的 **数值** 一旦分配不可更改，仅可追加
- 废弃的枚举值应标记 `@deprecated` 并注明替代方案
- 前后端枚举定义应保持严格一致
