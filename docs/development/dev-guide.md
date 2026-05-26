# 开发指南

## 1. 环境搭建

### 前置依赖

| 工具 | 版本要求 | 安装方式 |
|------|----------|----------|
| Node.js | | |
| 包管理器 (npm/pnpm/yarn) | | |
| Docker | | |
| 其他 | | |

### 本地启动

```bash
# 1. 克隆仓库
git clone <仓库地址>

# 2. 安装依赖
npm install

# 3. 配置环境变量
cp .env.example .env.local
# 编辑 .env.local 填写本地配置

# 4. 启动开发服务
npm run dev
```

### 本地访问

| 服务 | 地址 |
|------|------|
| 前端 | http://localhost:3000 |
| 后端 API | http://localhost:8080 |
| API 文档 (Swagger) | http://localhost:8080/swagger |

---

## 2. 项目结构

```
.
├── src/
│   ├── components/     # 通用组件
│   ├── pages/          # 页面
│   ├── services/       # API 请求封装
│   ├── hooks/          # 自定义 Hooks
│   ├── utils/          # 工具函数
│   ├── types/          # 类型定义
│   └── constants/      # 常量/枚举
├── server/
│   ├── controllers/    # 控制器
│   ├── services/       # 业务逻辑
│   ├── models/         # 数据模型
│   ├── routes/         # 路由
│   ├── middleware/     # 中间件
│   └── utils/          # 工具函数
├── tests/              # 测试
├── docs/               # 文档
└── scripts/            # 脚本
```

---

## 3. 编码规范

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 文件名 | kebab-case | `user-profile.ts` |
| 组件名 | PascalCase | `UserProfile` |
| 函数/变量 | camelCase | `getUserInfo` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 枚举 | PascalCase | `UserStatus` |
| 数据库列名 | snake_case | `created_at` |
| API 路径 | kebab-case | `/api/v1/user-profile` |

### Git 提交规范

```
<type>(<scope>): <subject>

# type 类型：
# feat     - 新功能
# fix      - 修复 bug
# docs     - 文档变更
# style    - 代码格式（不影响功能）
# refactor - 重构
# perf     - 性能优化
# test     - 测试相关
# chore    - 构建/工具变更
```

示例：`feat(user): 添加用户登录功能`

### 分支策略

| 分支 | 说明 |
|------|------|
| `main` | 生产环境分支，受保护 |
| `develop` | 开发主分支 |
| `feature/*` | 功能分支，从 develop 拉取 |
| `hotfix/*` | 紧急修复，从 main 拉取 |
| `release/*` | 发布分支 |

---

## 4. 常用命令

```bash
npm run dev          # 启动开发服务
npm run build        # 构建生产包
npm run test         # 运行所有测试
npm run test:watch   # 监听模式运行测试
npm run lint         # 代码检查
npm run format       # 代码格式化
```

---

## 5. 调试指南

- 浏览器 DevTools 调试
- Node.js 调试：`npm run dev:debug`
- 日志查看：参见 [运维手册](../operations/ops-guide.md)

---

## 6. 代码审查清单

- [ ] 代码风格符合规范
- [ ] 有适当的单元测试
- [ ] 错误处理完善
- [ ] 无明显性能问题
- [ ] API 文档已更新
- [ ] 数据库迁移脚本已准备
- [ ] 无硬编码的敏感信息
