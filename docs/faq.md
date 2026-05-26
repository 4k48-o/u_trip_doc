# 常见问题 (FAQ)

---

## 环境相关

### Q: 本地启动报错 "port already in use" 怎么办？
A: 端口被占用，执行 `lsof -i :<端口号>` 查看占用进程，修改 `.env.local` 中的端口配置或杀掉占用进程。

### Q: 数据库连接失败？
A: 检查 `.env.local` 中数据库配置是否正确，确认数据库服务已启动，确认 IP 白名单是否包含当前 IP。

### Q: npm install 失败？
A: 检查 Node.js 版本是否符合要求，尝试清除缓存 `npm cache clean --force` 后重试，或切换到对应的镜像源。

---

## 开发相关

### Q: 如何添加一个新的 API 接口？
A: 1) 在 `routes/` 中定义路由；2) 在 `controllers/` 中编写控制器；3) 在 `services/` 中编写业务逻辑；4) 更新 API 文档。具体参考 [开发指南](development/dev-guide.md)。

### Q: 如何添加新的数据库表？
A: 创建迁移文件 `npm run migration:create -- --name xxx`，编写 up/down 逻辑，执行 `npm run migration:run`。更新 [数据库设计文档](design/database.md)。

### Q: 如何添加新的错误码？
A: 在 [错误码文档](development/error-codes.md) 中按模块添加，并同步更新前后端的错误码定义文件。

### Q: 如何调试 API 请求？
A: 使用 Swagger 文档 (`http://localhost:8080/swagger`) 或 Postman 导入 API 集合。

---

## 部署相关

### Q: 如何部署到测试环境？
A: 将代码合并到 `develop` 分支，CI/CD 会自动构建并部署到测试环境。

### Q: 如何回滚版本？
A: 参考 [部署文档](deployment/deploy-doc.md) 的回滚方案章节。

### Q: 生产环境部署需要做什么检查？
A: 参考 [部署文档](deployment/deploy-doc.md) 的部署检查清单章节。

---

## 故障排查

### Q: 用户反映登录失败？
A: 检查：1) 账号是否被禁用/删除；2) 密码是否正确；3) 是否触发限频；4) Redis 是否正常（Token 过期校验依赖 Redis）；5) 查看对应 requestId 的日志。

### Q: 接口返回 500 错误？
A: 1) 查看服务日志中的 ERROR 信息；2) 检查数据库连接；3) 检查第三方服务是否可用；4) 查看最近是否有代码变更。

### Q: 上传文件失败？
A: 检查：1) 文件大小是否超限；2) 文件类型是否在允许列表；3) OSS 服务配置是否正确；4) 磁盘空间是否充足。

---

## 其他

### Q: 如何申请权限？
A: 联系管理员，提供需要的角色和权限范围。

### Q: 文档在哪里？
A: 项目文档站：`https://docs.example.com`（或本地 `mkdocs serve`）

### Q: 发现安全漏洞怎么办？
A: 请勿在公开渠道讨论，立即联系安全负责人。参考 [安全文档](security/security.md) 的安全事件响应章节。
