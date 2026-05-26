# 部署文档

---

## 1. 部署架构

```
[ 用户 ]
    │
    ▼
[ CDN ] ── 静态资源
    │
    ▼
[ Nginx / API Gateway ] ── 反向代理 / 证书 / 限流
    │
    ├── [ 前端容器 ]  (Node.js + Nginx 静态文件)
    │
    └── [ 后端容器 ]  (Node.js)
            │
            ├── [ 数据库 ] (MySQL / PostgreSQL)
            ├── [ 缓存 ]   (Redis)
            └── [ OSS ]    (文件存储)
```

---

## 2. 环境说明

| 环境 | 域名 | 分支 | 说明 |
|------|------|------|------|
| 本地开发 | localhost | 任意分支 | 本地开发调试 |
| 开发环境 (dev) | dev.example.com | develop | 日常联调 |
| 测试环境 (test) | test.example.com | release/* | 测试验收 |
| 预发布 (staging) | staging.example.com | release/* | 上线前验证 |
| 生产环境 (prod) | example.com | main | 对外服务 |

---

## 3. 构建流程

### 前端构建

```bash
npm run build
# 产出目录：dist/
```

### 后端构建

```bash
npm run build
# 产出目录：dist/
```

### Docker 构建

```bash
# 构建镜像
docker build -t project-name:latest .

# 推送镜像
docker push registry.example.com/project-name:latest
```

---

## 4. 部署方式

### 方式一：手动部署（开发/测试环境）

```bash
# 1. SSH 登录服务器
ssh user@dev-server

# 2. 拉取最新代码
cd /opt/project
git pull origin develop

# 3. 安装依赖
npm ci

# 4. 构建
npm run build

# 5. 重启服务
pm2 restart project-api
```

### 方式二：CI/CD 自动部署（推荐）

触发条件：合并到对应分支后自动触发

```yaml
# .github/workflows/deploy.yml (示例)
name: Deploy
on:
  push:
    branches: [develop, main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: |
          npm ci
          npm run test
          npm run build
      - name: Deploy to Staging
        if: github.ref == 'refs/heads/develop'
        run: |
          # 部署到预发布环境
      - name: Deploy to Production
        if: github.ref == 'refs/heads/main'
        run: |
          # 部署到生产环境
```

---

## 5. 生产环境配置

### 服务器要求

| 资源 | 最低配置 | 推荐配置 |
|------|----------|----------|
| CPU | 2 核 | 4 核 |
| 内存 | 4 GB | 8 GB |
| 磁盘 | 20 GB SSD | 50 GB SSD |
| 操作系统 | Ubuntu 20.04+ / CentOS 7+ | Ubuntu 22.04 LTS |

### 服务端口

| 服务 | 端口 | 公网暴露 |
|------|------|----------|
| 前端 (Nginx) | 80 / 443 | 是 |
| 后端 API | 8080 | 否（仅内网） |
| 数据库 | 3306 | 否（仅内网） |
| Redis | 6379 | 否（仅内网） |

---

## 6. Docker Compose 示例

```yaml
version: '3.8'
services:
  api:
    image: registry.example.com/project-api:latest
    restart: always
    ports:
      - "127.0.0.1:8080:8080"
    env_file:
      - .env.production
    depends_on:
      - redis
    volumes:
      - ./uploads:/app/uploads

  redis:
    image: redis:7-alpine
    restart: always
    volumes:
      - redis-data:/data

volumes:
  redis-data:
```

---

## 7. 数据库迁移

```bash
# 创建迁移文件
npm run migration:create -- --name add_user_table

# 执行迁移（生产环境需谨慎）
npm run migration:run

# 回滚上一次迁移
npm run migration:revert
```

> **生产环境迁移必须备份数据库！**

---

## 8. 回滚方案

| 场景 | 操作 |
|------|------|
| 代码回滚 | 部署上一个版本的 Docker 镜像或 Git tag |
| 数据库回滚 | 执行 `npm run migration:revert`（如有不可逆变更则从备份恢复） |
| 配置回滚 | 恢复到上一个正常运行的环境变量配置 |

回滚命令示例：
```bash
# Docker 版本回滚
docker pull registry.example.com/project-api:v1.2.0
docker-compose up -d

# 数据库回滚
npm run migration:revert
```

---

## 9. 健康检查

| 端点 | 说明 |
|------|------|
| `GET /api/health` | 服务存活检查 |
| `GET /api/health/ready` | 就绪检查（含数据库/Redis 连接检测） |

---

## 10. 部署检查清单

- [ ] 代码已合并到目标分支
- [ ] CI 流水线全部通过
- [ ] 数据库备份已完成（生产环境）
- [ ] 相关依赖服务已就绪
- [ ] 环境变量已正确配置
- [ ] 健康检查接口正常
- [ ] 关键 API 手动验证通过
- [ ] 告警监控已确认正常
- [ ] 回滚方案已准备
