# 测试文档

---

## 1. 测试策略

| 层级 | 工具 | 覆盖目标 | 说明 |
|------|------|----------|------|
| 单元测试 | [Jest / Vitest] | ≥ 80% | 测试纯函数、工具方法、数据转换逻辑 |
| 集成测试 | [Jest + Supertest] | 核心 API 全覆盖 | 测试 API 接口、数据库交互、中间件 |
| E2E 测试 | [Playwright / Cypress] | 核心流程全覆盖 | 测试完整用户操作流程 |
| 组件测试 | [Testing Library] | 核心组件 | 测试 UI 组件渲染与交互 |

---

## 2. 运行测试

```bash
npm run test              # 运行所有测试
npm run test:watch        # 监听模式
npm run test:coverage     # 生成覆盖率报告
npm run test:e2e          # 运行 E2E 测试
npm run test:api          # 运行 API 集成测试
```

---

## 3. 测试文件组织

```
tests/
├── unit/                 # 单元测试
│   ├── utils/            # 工具函数测试
│   └── services/         # 服务层测试
├── integration/          # 集成测试
│   ├── api/              # API 接口测试
│   └── db/               # 数据库交互测试
├── e2e/                  # 端到端测试
│   └── flows/            # 用户流程测试
├── fixtures/             # 测试固定数据
├── helpers/              # 测试辅助函数
└── mocks/                # Mock 数据
```

---

## 4. 核心测试用例

### 4.1 用户模块

| 用例编号 | 用例名称 | 测试类型 | 优先级 |
|----------|----------|----------|--------|
| TC-USR-001 | 注册 - 正常注册 | 集成 | P0 |
| TC-USR-002 | 注册 - 重复用户名 | 集成 | P1 |
| TC-USR-003 | 注册 - 弱密码 | 集成 | P1 |
| TC-USR-004 | 登录 - 正常登录 | 集成 | P0 |
| TC-USR-005 | 登录 - 错误密码 | 集成 | P0 |
| TC-USR-006 | 登录 - 账号不存在 | 集成 | P1 |
| TC-USR-007 | 登录 - 限频拦截 | 集成 | P1 |
| TC-USR-008 | Token - 过期处理 | 单元 | P1 |
| TC-USR-009 | Token - 刷新 | 集成 | P0 |
| TC-USR-010 | 获取用户信息 | 集成 | P0 |
| TC-USR-011 | 修改用户信息 | 集成 | P1 |
| TC-USR-012 | 权限校验 - 普通用户访问管理接口 | 集成 | P0 |

### 4.2 通用

| 用例编号 | 用例名称 | 测试类型 | 优先级 |
|----------|----------|----------|--------|
| TC-COM-001 | 参数校验 - 必填缺失 | 单元 | P0 |
| TC-COM-002 | 参数校验 - 类型错误 | 单元 | P1 |
| TC-COM-003 | 404 路由不存在 | 集成 | P2 |
| TC-COM-004 | 全局异常捕获 | 集成 | P0 |
| TC-COM-005 | CORS 跨域配置 | 集成 | P1 |
| TC-COM-006 | 请求日志记录 | 集成 | P2 |

---

## 5. 测试数据管理

- **隔离原则**：每个测试用例使用独立的测试数据，不互相依赖
- **清理策略**：测试用例执行后清理产生的数据（`afterEach` / `afterAll`）
- **Fixture**：常用固定数据放在 `tests/fixtures/` 目录
- **Factory**：动态生成测试数据使用 Factory 函数
- **测试数据库**：使用独立的测试数据库，不与开发/生产共用

---

## 6. Mock 策略

| 被 Mock 对象 | Mock 方式 | 说明 |
|-------------|-----------|------|
| 第三方 API | jest.mock / nock | 不依赖外部服务 |
| Redis | ioredis-mock | 集成测试使用 Mock Redis |
| 文件系统 | memfs | 文件操作测试 |
| 时间 | jest.useFakeTimers | 时间相关逻辑测试 |
| 环境变量 | process.env 注入 | 不同配置场景测试 |

---

## 7. 测试编写规范

```typescript
// 命名规范：describe('[模块名]', () => { it('should [预期行为]', () => {}) })
describe('UserService', () => {
  describe('login', () => {
    it('should return token when credentials are valid', async () => {
      // Arrange - 准备数据
      // Act - 执行操作
      // Assert - 验证结果
    });

    it('should throw AUTH_002 when password is wrong', async () => {
      // ...
    });
  });
});
```

---

## 8. CI 测试门禁

| 检查项 | 要求 | 阻断级别 |
|--------|------|----------|
| 单元测试通过率 | 100% | 阻断合并 |
| 集成测试通过率 | 100% | 阻断合并 |
| 代码覆盖率 | ≥ 80% | 警告（非阻断） |
| Lint 检查 | 0 error | 阻断合并 |
| 类型检查 | 0 error | 阻断合并 |
