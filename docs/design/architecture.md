# 系统架构设计

---

## 1. 整体架构

### 1.1 架构风格

采用**微服务架构**，各业务子系统独立部署、独立升级、独立扩展。核心交易链路（售检票 + 订单 + 支付）与数据分析系统分离部署，降低分析负载对交易的影响。

### 1.2 分层结构

```
┌─────────────────────────────────────────────────────────┐
│                     接入层 (Gateway)                      │
│  API 网关 · 鉴权 · 限流 · 路由 · 日志 · 协议转换            │
├─────────────────────────────────────────────────────────┤
│                     业务服务层 (BFF + Services)            │
│  ┌─────────────── P0 核心交易 ───────────────┐           │
│  │ 票务服务 · 商品服务 · 订单服务 · 支付服务 · 清分服务 │           │
│  │ 发票服务 · 认证服务 · 系统管理服务            │           │
│  └────────────────────────────────────────────┘           │
│  ┌─────────────── P1 重要业务 ───────────────┐           │
│  │ 电商BFF · 商户服务 · 营销服务 · 会员服务 · 进销存服务 │         │
│  │ 人脸服务 · 客流服务 · 数据分析服务 · 集成对接服务  │           │
│  └────────────────────────────────────────────┘           │
│  ┌─────────────── P2 后续迭代 ───────────────┐           │
│  │ 商业管理服务 · 车辆服务 · 内容服务 · AI服务       │         │
│  └────────────────────────────────────────────┘           │
├─────────────────────────────────────────────────────────┤
│                     基础设施层                              │
│  服务注册 · 配置中心 · 消息队列 · 任务调度 · 链路追踪        │
├─────────────────────────────────────────────────────────┤
│                     数据层                                  │
│  MySQL · Redis · Elasticsearch · OSS · 时序数据库           │
└─────────────────────────────────────────────────────────┘
```

### 1.3 技术选型

| 层级 | 技术 | 说明 |
|------|------|------|
| 接入网关 | Kong | API 网关、鉴权、限流、路由 |
| 服务框架 | Spring Cloud | 后端微服务 |
| 前端 | Vue3 + Vite | 小程序、H5、PC |
| 服务注册 | Nacos | 服务发现与配置管理 |
| 消息队列 | RabbitMQ | 异步解耦、削峰填谷 |
| 数据库 | MySQL 8.0 高可用版 | 主从 + 读写分离 |
| 缓存 | Redis Cluster | 缓存 + 分布式锁 + Session |
| 搜索引擎 | Elasticsearch | 订单检索、日志分析 |
| 对象存储 | 阿里云 OSS | 图片、文件、备份 |
| 链路追踪 | SkyWalking | 分布式链路追踪 |
| 监控告警 | Prometheus + Grafana | 指标采集与可视化 |
| 日志收集 | 阿里云 SLS | 集中日志管理 |
| 容器编排 | Docker Compose | 容器化部署 |
| CI/CD | GitHub Actions | 自动构建部署 |

---

## 2. 模块划分

### 2.1 服务清单

| 服务 | 代码标识 | 优先级 | 所属域 | 职责 |
|------|----------|--------|--------|------|
| API 网关 | api-gateway | P0 | 接入层 | 路由转发、鉴权、限流、日志、跨域 |
| 认证服务 | auth-service | P0 | 系统管理 | 登录、Token 签发与刷新、RBAC 鉴权 |
| 用户服务 | user-service | P0 | 系统管理 | 用户信息、角色、权限、操作审计 |
| 票务服务 | ticket-service | P0 | 售检票 | 分时预约、窗口售票、检票核销、年卡 |
| 商品服务 | product-service | P0 | 商品中心 | SPU/SKU、组合产品引擎、价格日历、渠道定价 |
| 订单服务 | order-service | P0 | 订单中心 | 多业态订单汇聚、主单/子单、退票 |
| 支付服务 | payment-service | P0 | 订单中心 | 聚合支付、退款、支付回调 |
| 清分服务 | settlement-service | P0 | 订单中心 | 组合票分账、商户清分、用友 NC 对接 |
| 发票服务 | invoice-service | P0 | 订单中心 | 数电票申领/开具/冲红/归档 |
| 系统管理服务 | admin-service | P0 | 系统管理 | 组织架构、设备管理、监控告警、消息中心 |
| 电商 BFF | shop-bff | P1 | 电商平台 | 小程序/H5/PC 前端聚合接口 |
| 商户服务 | merchant-service | P1 | 电商平台 | 商户入驻、资质审核、店铺装修、对账 |
| 营销服务 | marketing-service | P1 | 营销体系 | 优惠券、秒杀、拼团、限时折扣、分销 |
| 会员服务 | member-service | P1 | 营销体系 | 会员等级、积分、权益、精准推送 |
| 进销存服务 | inventory-service | P1 | 进销存 | 供应商、商品条码、入库/调拨/盘点、收银 |
| 人脸服务 | face-service | P1 | 人脸识别 | 人像采集、1:N 比对、底库同步、合规 |
| 客流服务 | traffic-service | P1 | 店铺客流 | 客流统计、实时看板、周期对比 |
| 数据分析服务 | analytics-service | P1 | 综合管理 | 大屏数据、多维报表、游客画像、预警 |
| 集成对接服务 | integration-service | P1 | 外部对接 | OTA 接口、酒店/餐饮对接、公众号同步 |
| 通知服务 | notification-service | P1 | 系统管理 | 短信/邮件/微信模板、批量发送 |
| 商业管理服务 | biz-mgmt-service | P2 | 商业管理 | 商铺租约、租金、物业费、欠费预警 |
| 车辆服务 | vehicle-service | P2 | 车辆定位 | GPS/北斗定位、轨迹回放、异常告警 |
| 内容服务 | content-service | P2 | 导览/社区 | 攻略/笔记/视频 UGC、手绘地图、语音讲解 |
| AI 服务 | ai-service | P2 | AI 服务 | 智能问答、意图识别、知识库、人机转接 |

### 2.2 服务边界

#### P0 服务

| 服务 | 核心实体 | 对外接口（示例） |
|------|----------|------------------|
| auth-service | Token, Session, LoginLog | POST /auth/login, POST /auth/refresh, POST /auth/logout |
| user-service | User, Role, Permission, AuditLog | GET /users/:id, PUT /users/:id, GET /users/:id/orders |
| ticket-service | Ticket, Reservation, CheckRecord, AnnualCard | POST /tickets/reserve, POST /tickets/sell, POST /tickets/verify |
| product-service | SPU, SKU, ComboProduct, PriceCalendar | GET /products, POST /products, PUT /products/:id/stock |
| order-service | Order, SubOrder, Refund | POST /orders, GET /orders/:id, POST /orders/:id/refund |
| payment-service | Payment, Refund, PayChannel | POST /payments, POST /payments/callback/:channel |
| settlement-service | Settlement, Ledger, NCVoucher | GET /settlements, POST /settlements/trigger |
| invoice-service | Invoice, TaxMapping, InvoiceApply | POST /invoices/apply, POST /invoices/red |
| admin-service | Org, Device, MonitorRule, MessageTemplate | GET /admin/orgs, PUT /admin/devices/:id |

#### P1 服务

| 服务 | 核心实体 | 对外接口（示例） |
|------|----------|------------------|
| shop-bff | (聚合层，无独立实体) | GET /shop/home, GET /shop/products/:id |
| merchant-service | Merchant, Shop, Product, Settlement | POST /merchants/apply, GET /merchants/:id/shop |
| marketing-service | Coupon, SecKill, GroupBuy, Distribution | POST /marketing/coupons, POST /marketing/seckill |
| member-service | Member, Points, Benefits, Tag | GET /members/:id/points, POST /members/:id/claim |
| inventory-service | Supplier, Goods, Stock, CashierOrder | POST /inventory/inbound, POST /inventory/checkout |
| face-service | FaceImage, FaceLib, VerifyLog | POST /face/collect, POST /face/verify |
| traffic-service | VisitorFlow, StoreView, Compare | GET /traffic/realtime/:storeId |
| analytics-service | Dashboard, Report, Alert | GET /analytics/dashboard, POST /analytics/reports |
| integration-service | OTAProxy, HotelSync, CateringSync | (内部服务，定时同步/回调) |
| notification-service | Template, SendLog | POST /notifications/sms, POST /notifications/email |

#### P2 服务

| 服务 | 核心实体 | 对外接口（示例） |
|------|----------|------------------|
| biz-mgmt-service | ShopLease, Rent, PropertyFee | GET /biz/shops/:id/lease |
| vehicle-service | Vehicle, GPSPoint, RouteAlert | GET /vehicles/realtime |
| content-service | Article, Note, HandMap, AudioGuide | POST /contents/article, GET /contents/map |
| ai-service | FAQ, DialogLog, KnowledgeBase | POST /ai/chat, POST /ai/knowledge/sync |

---

## 3. 模块间依赖关系

### 3.1 核心交易链路依赖

```
                        ┌─────────────┐
                        │  auth-service │
                        └──────┬───────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
     ┌────────────┐  ┌─────────────┐  ┌─────────────┐
     │ticket-service│  │order-service│  │ admin-service│
     └──────┬─────┘  └──────┬──────┘  └─────────────┘
            │               │
   ┌────────┼───────┬───────┼────────┬────────┐
   ▼        ▼       ▼       ▼        ▼        ▼
┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐
│product││payment││invoice││settle││face  ││member│
│-serv ││-serv  ││-serv ││-serv ││-serv ││-serv │
└──────┘└──────┘└──────┘└──────┘└──────┘└──────┘
```

### 3.2 依赖关系矩阵

| 消费方 ↓ / 提供方 → | auth | user | ticket | product | order | payment | settle | invoice | admin | member |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **ticket-service** | R | R | - | R | R | - | - | - | - | - |
| **product-service** | R | R | - | - | - | - | - | - | - | - |
| **order-service** | R | R | R | R | - | R | R | - | - | R |
| **payment-service** | - | - | - | - | R | - | - | - | - | - |
| **settlement-service** | - | - | - | - | R | R | - | R | - | - |
| **invoice-service** | - | - | - | - | R | - | - | - | - | - |
| **admin-service** | R | R | R | R | R | - | - | - | - | - |
| **merchant-service** | R | R | - | R | R | - | R | - | - | - |
| **marketing-service** | - | - | - | R | R | - | - | - | - | R |
| **analytics-service** | - | - | R | R | R | R | R | R | - | R |

> R = 读依赖（同步调用），W = 写依赖，- = 无直接依赖

### 3.3 异步通信

| 场景 | 消息 | 生产者 | 消费者 |
|------|------|--------|--------|
| 支付成功通知 | payment.completed | payment-service | order-service, settlement-service, notification-service |
| 订单退款完成 | order.refunded | order-service | payment-service, settlement-service, invoice-service |
| 检票核销事件 | ticket.verified | ticket-service | order-service, analytics-service, traffic-service |
| 库存变更 | inventory.changed | inventory-service | product-service, analytics-service |
| OTA 订单同步 | ota.order.sync | integration-service | order-service, product-service |
| 发票开具完成 | invoice.issued | invoice-service | notification-service |
| 商家提醒通知 | notification.send | admin-service | notification-service |
| 异常预警 | alert.triggered | analytics-service | admin-service, notification-service |
| 人脸底库同步 | face.lib.sync | admin-service | face-service |
| 数据埋点 | data.track.* | shop-bff, 各前端 | analytics-service |

---

## 4. 部署架构

### 4.1 云环境

```
                      ┌──────────────────────────────────────────┐
                      │        CDN / DNS / WAF                    │
                      └──────────────────┬───────────────────────┘
                                         │
                      ┌──────────────────┴───────────────────────┐
                      │       负载均衡 (SLB)                       │
                      └──────────────────┬───────────────────────┘
                                         │
          ┌──────────────────────────────┼──────────────────────────────┐
          ▼                              ▼                              ▼
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│  Web 集群 x2     │          │  API 集群 x4     │          │  静态资源 CDN     │
│  Nginx + 前端    │          │  Node.js 微服务  │          │  OSS + CDN       │
│  (H5, PC, Admin) │          │  (容器化部署)    │          │                  │
└─────────────────┘          └────────┬────────┘          └─────────────────┘
                                      │
           ┌──────────────────────────┼──────────────────────────┐
           ▼                          ▼                          ▼
┌─────────────────┐    ┌──────────────────────┐    ┌─────────────────┐
│  MySQL 8.0      │    │  Redis Cluster       │    │  Elasticsearch   │
│  主从 (1主2从)   │    │  3 节点              │    │  3 节点          │
│  + 只读副本      │    │  (缓存/锁/Session)   │    │  (搜索/日志)     │
└─────────────────┘    └──────────────────────┘    └─────────────────┘

          ┌──────────────────────────┼──────────────────────────┐
          ▼                          ▼                          ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────┐
│  消息队列        │    │  对象存储        │    │  监控 + 日志         │
│  RabbitMQ       │    │  阿里云 OSS      │    │  Grafana +       │
│  (异步解耦)      │    │  (图片/文件/备份) │    │  阿里云 SLS      │
└─────────────────┘    └─────────────────┘    └─────────────────────┘
```

### 4.2 本地备份与离线能力

```
┌────────────────────────────────────────────────────────┐
│                    景区本地机房                           │
│  ┌───────────┐  ┌───────────┐  ┌──────────────────┐   │
│  │ 本地售检票  │  │ 闸机数据   │  │ 手持机离线缓存    │   │
│  │ 备份服务器  │  │ 预存服务器  │  │                  │   │
│  └───────────┘  └───────────┘  └──────────────────┘   │
│                                                         │
│  云断网时：                                               │
│  → 本地服务器接管售票                                      │
│  → 闸机使用预存数据离线检票                                 │
│  → 手持机离线检票                                         │
│  → 网络恢复后数据自动同步至云端                             │
└────────────────────────────────────────────────────────┘
```

### 4.3 阿里云资源配置

| 资源 | 数量 | 配置 | 用途 |
|------|------|------|------|
| 业务 ECS | 12 台 | 16C/32G/100G/100G/10Mbps | 微服务集群 |
| 负载均衡 | 1 台 | 16C/32G/100G/100G/1Mbps | 入口流量分发 |
| Redis 节点 | 3 台 | 2C/4G/20G/40G/1Mbps | 缓存 / 分布式锁 / Session |
| MySQL 高可用 | 2 台 | 16C/32G 独享型 | 主从数据库 |
| 公用服务器 | 1 台 | 16C/32G/100G/100G/30Mbps | 静态资源 / 代理 |
| 本地机房 | 4 台 | 2U / 12C / 64G / NVMe / SAS | 离线备份 / 数据预存 |

### 4.4 网络架构

| 区域 | 说明 |
|------|------|
| 公网入口 | 仅 80/443 端口通过 CDN/SLB 暴露 |
| 内网服务 | 所有微服务、数据库、Redis、MQ 仅内网通信 |
| 安全管理 | 安全组最小权限，数据库仅允许指定服务访问 |
| 本地连接 | 景区本地机房通过 VPN 专线连接云端，售检票数据预存 |

### 4.5 高可用策略

| 组件 | 高可用方案 | 故障恢复 |
|------|-----------|----------|
| 网关 | SLB + 多副本 | 自动切换 |
| 微服务 | Docker Compose 多副本 + 健康检查 | 自动重启 |
| MySQL | 主从复制 + 自动故障转移 | < 30 秒切换 |
| Redis | Cluster 哨兵模式 | 自动选主 |
| 消息队列 | 集群 + 持久化 | 消息不丢 |
| OSS | 阿里云高可用 + 跨区域备份 | 自动恢复 |
| 售检票 | 本地备份服务器 + 离线检票 | 断网即刻切换 |

---

## 5. 关键技术决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 数据分析分离 | 独立 analytics-service + 独立 DB | 降低分析负载对核心交易的影响 |
| 组合产品引擎 | 独立 product-service 内子模块 | 灵活的定价 + 清分 + 核销规则配置 |
| 人脸比对位置 | 终端本地比对（闸机/手持机） | 减少网络依赖，保证 ≤3 秒通行 |
| 支付回调 | 统一 payment-service 入口 | 回调签名验证集中处理，安全可控 |
| 多租户隔离 | 数据库级别隔离（独立 Schema） | 平台方与商户数据安全隔离 |
| 开放 API | 独立 API 版本体系 + 沙箱环境 | 保障生态扩展与第三方接入 |

---

## 6. 安全架构

```
┌─────────────────────────────────────────────────────┐
│                  WAF + DDoS 防护                      │
├─────────────────────────────────────────────────────┤
│  全站 HTTPS (TLS 1.3)                                 │
├─────────────────────────────────────────────────────┤
│  API 网关                                              │
│  ├─ Token 验证 (JWT)                                   │
│  ├─ RBAC 权限校验                                      │
│  ├─ 接口限流 (IP + User + API 多维度)                   │
│  ├─ 请求签名验证 (支付回调)                              │
│  └─ CORS 白名单                                        │
├─────────────────────────────────────────────────────┤
│  应用层                                                │
│  ├─ 参数校验 + SQL 参数化查询                           │
│  ├─ XSS/CSRF 防护                                     │
│  ├─ 敏感数据加密 (AES-256) + 脱敏                       │
│  ├─ 操作审计全量记录                                    │
│  └─ 防黄牛/防刷单/防恶意占票                             │
├─────────────────────────────────────────────────────┤
│  数据层                                                │
│  ├─ DB 内网隔离 + 白名单                                │
│  ├─ Redis 密码 + bind 内网                             │
│  ├─ 数据全量+增量备份                                   │
│  └─ 人脸/身份证等保三级加密                              │
└─────────────────────────────────────────────────────┘
```
