# 配置说明

> 项目环境变量与配置项参考。

---

## 1. 配置文件层级

| 文件 | 环境 | 说明 |
|------|------|------|
| `.env` | 通用 | 默认配置，不区分环境 |
| `.env.local` | 本地 | 本地开发覆盖，已 gitignore |
| `.env.development` | 开发 | 开发环境 |
| `.env.staging` | 预发布 | 预发布环境 |
| `.env.production` | 生产 | 生产环境 |

> 优先级：`.env.local` > `.env.[environment]` > `.env`

---

## 2. 环境变量列表

### 服务基础配置

| 变量名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| NODE_ENV | string | 否 | development | 环境标识：development / staging / production |
| PORT | number | 否 | 8080 | 服务端口 |
| HOST | string | 否 | 0.0.0.0 | 监听地址 |
| LOG_LEVEL | string | 否 | info | 日志级别：debug / info / warn / error |

### 数据库配置

| 变量名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| DB_HOST | string | 是 | - | 数据库主机 |
| DB_PORT | number | 是 | 3306 | 数据库端口 |
| DB_USER | string | 是 | - | 数据库用户名 |
| DB_PASSWORD | string | 是 | - | 数据库密码 |
| DB_NAME | string | 是 | - | 数据库名称 |
| DB_POOL_MIN | number | 否 | 2 | 连接池最小连接数 |
| DB_POOL_MAX | number | 否 | 10 | 连接池最大连接数 |

### Redis 配置

| 变量名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| REDIS_HOST | string | 否 | 127.0.0.1 | Redis 主机 |
| REDIS_PORT | number | 否 | 6379 | Redis 端口 |
| REDIS_PASSWORD | string | 否 | - | Redis 密码 |
| REDIS_DB | number | 否 | 0 | Redis 数据库编号 |

### JWT / 认证配置

| 变量名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| JWT_SECRET | string | 是 | - | JWT 签名密钥 |
| JWT_EXPIRE | number | 否 | 7200 | Token 有效期（秒） |
| JWT_REFRESH_EXPIRE | number | 否 | 604800 | Refresh Token 有效期（秒） |

### 文件 / OSS 配置

| 变量名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| UPLOAD_DIR | string | 否 | ./uploads | 本地上传目录 |
| UPLOAD_MAX_SIZE | number | 否 | 10485760 | 上传文件大小限制（字节） |
| UPLOAD_ALLOWED_TYPES | string | 否 | jpg,jpeg,png,gif,pdf | 允许的文件类型 |
| OSS_PROVIDER | string | 否 | local | 存储类型：local / s3 / oss |
| OSS_ENDPOINT | string | 否 | - | OSS 服务端点 |
| OSS_BUCKET | string | 否 | - | Bucket 名称 |
| OSS_ACCESS_KEY | string | 否 | - | Access Key |
| OSS_SECRET_KEY | string | 否 | - | Secret Key |
| OSS_CDN_DOMAIN | string | 否 | - | CDN 域名（拼接文件 URL 用） |

### 第三方服务配置

| 变量名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| SMS_PROVIDER | string | 否 | - | 短信服务商 |
| SMS_ACCESS_KEY | string | 否 | - | 短信服务 Key |
| SMS_SECRET | string | 否 | - | 短信服务 Secret |
| EMAIL_HOST | string | 否 | - | SMTP 主机 |
| EMAIL_PORT | number | 否 | 465 | SMTP 端口 |
| EMAIL_USER | string | 否 | - | SMTP 用户名 |
| EMAIL_PASSWORD | string | 否 | - | SMTP 密码 |

### 前端配置

| 变量名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| VITE_API_BASE_URL | string | 否 | /api | API 基础路径 |
| VITE_CDN_DOMAIN | string | 否 | - | 静态资源 CDN 域名 |
| VITE_SENTRY_DSN | string | 否 | - | Sentry 监控 DSN |

---

## 3. .env.example 模板

```bash
# ============ 基础配置 ============
NODE_ENV=development
PORT=8080
LOG_LEVEL=debug

# ============ 数据库 ============
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=
DB_NAME=project_db

# ============ Redis ============
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=

# ============ JWT ============
JWT_SECRET=change_me_to_a_random_string
JWT_EXPIRE=7200

# ============ 文件上传 ============
UPLOAD_DIR=./uploads
UPLOAD_MAX_SIZE=10485760

# ============ 第三方服务 ============
# 短信
SMS_PROVIDER=
SMS_ACCESS_KEY=
SMS_SECRET=

# 邮件
EMAIL_HOST=
EMAIL_PORT=465
EMAIL_USER=
EMAIL_PASSWORD=
```

---

## 4. 特性开关 (Feature Flags)

| 开关名称 | 类型 | 默认值 | 说明 |
|----------|------|--------|------|
| FEATURE_DARK_MODE | boolean | false | 深色模式 |
| FEATURE_EXPORT | boolean | false | 数据导出功能 |
| FEATURE_NEW_DASHBOARD | boolean | false | 新版仪表板 |

特性开关配置方式：[占位，如 LaunchDarkly / 本地 env / 数据库配置]

---

## 5. 维护说明

- **敏感信息**（密钥、密码）不得提交到代码仓库
- 新增环境变量必须同步更新本文档及 `.env.example`
- 配置变更需走发布流程，生产环境需审批
- 本地开发可使用 `.env.local` 覆盖默认值
