# 运维手册

---

## 1. 监控指标

| 指标 | 工具 | 告警阈值 | 说明 |
|------|------|----------|------|
| 服务存活 | 健康检查端点 | 连续 3 次失败 | 服务是否运行 |
| CPU 使用率 | Prometheus / 云监控 | > 80% (持续 5 分钟) | 需扩容或排查 |
| 内存使用率 | Prometheus / 云监控 | > 85% (持续 5 分钟) | 可能内存泄漏 |
| API 响应时间 (P95) | APM / Middleware | > 2000ms | 接口性能瓶颈 |
| API 错误率 | APM / Middleware | > 5% | 异常增多 |
| 数据库连接数 | DB 监控 | > 80% 连接池上限 | 连接泄漏或流量高峰 |
| 磁盘使用率 | 云监控 / df | > 85% | 日志/文件清理 |
| 登录失败频率 | 日志统计 | > 50 次/分钟 | 可能暴力破解 |

---

## 2. 日志规范

### 日志级别

| 级别 | 说明 | 使用场景 |
|------|------|----------|
| DEBUG | 调试信息 | 开发调试，生产环境不输出 |
| INFO | 关键流程节点 | 请求日志、重要业务操作 |
| WARN | 警告信息 | 可恢复的异常、降级操作 |
| ERROR | 错误信息 | 需要人工介入的错误 |

### 日志格式

```json
{
  "timestamp": "2026-05-26T10:00:00.000Z",
  "level": "INFO",
  "requestId": "uuid-string",
  "userId": "user-id",
  "method": "POST",
  "path": "/api/v1/users",
  "status": 200,
  "duration": 42,
  "message": "Request completed"
}
```

### 日志存储与轮转

| 项目 | 配置 |
|------|------|
| 存储位置 | `/var/log/project/` |
| 轮转策略 | 按天轮转，保留 30 天 |
| 压缩 | 7 天前的日志自动 gzip 压缩 |
| 集中收集 | [ELK / Loki / 云日志服务] |

### 关键日志查法

```bash
# 搜索特定 requestId 的日志链
grep "requestId-xxx" /var/log/project/app.log

# 搜索最近 100 条 ERROR
tail -n 10000 /var/log/project/app.log | grep ERROR

# 按时间范围搜索
grep "2026-05-26 10:" /var/log/project/app.log
```

---

## 3. 告警配置

| 告警名称 | 触发条件 | 通知方式 | 处理人 |
|----------|----------|----------|--------|
| 服务宕机 | 健康检查连续失败 | 电话 + 钉钉/飞书 | 值班人员 |
| 接口错误率飙升 | 5 分钟内 > 10% | 钉钉/飞书 | 后端负责人 |
| 接口响应慢 | P95 > 3s 持续 5 分钟 | 钉钉/飞书 | 后端负责人 |
| 数据库连接池耗尽 | 使用率 > 90% | 钉钉/飞书 | DBA |
| 磁盘空间不足 | < 10% | 钉钉/飞书 | 运维 |
| 证书即将过期 | < 15 天 | 邮件 + 钉钉/飞书 | 运维 |
| 安全事件 | 异常登录/暴力破解 | 即时通知 | 安全负责人 |

---

## 4. 应急预案

### 4.1 服务宕机

1. 检查服务器状态：`docker ps` / `pm2 list`
2. 检查日志：`tail -200 /var/log/project/app.log`
3. 尝试重启：`docker-compose restart` / `pm2 restart all`
4. 查看资源使用：`top` / `df -h` / `free -m`
5. 如无法恢复，执行回滚方案

### 4.2 数据库故障

1. 检查数据库连接：`telnet <DB_HOST> <DB_PORT>`
2. 检查连接数：`SHOW PROCESSLIST;`
3. 检查慢查询：`SHOW FULL PROCESSLIST;` 中时间较长的查询
4. 必要时 Kill 慢查询：`KILL <id>;`
5. 联系 DBA 或云服务商

### 4.3 安全事件

1. 立即通知安全负责人
2. 暂时摘除受影响服务的流量
3. 保留现场日志和证据
4. 排查攻击来源和受影响范围
5. 修复漏洞后恢复服务

---

## 5. Nginx 配置参考

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate     /etc/nginx/certs/example.com.pem;
    ssl_certificate_key /etc/nginx/certs/example.com.key;

    # 前端静态文件
    location / {
        root /opt/project/frontend/dist;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # API 反向代理
    location /api/ {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 上传文件限制
    client_max_body_size 20m;
}
```

---

## 6. 定时任务

| 任务 | 频率 | 说明 |
|------|------|------|
| 日志清理 | 每日 03:00 | 清理超过保留期的日志 |
| 临时文件清理 | 每日 04:00 | 清理过期临时文件/验证码 |
| 数据库备份 | 每日 05:00 | 全量备份至异地存储 |
| Token 清理 | 每小时 | 清理过期 Token 记录 |
| SSL 证书检查 | 每周 | 检查证书有效期 |

---

## 7. 常用运维命令

```bash
# 查看运行状态
pm2 status
docker ps

# 查看实时日志
pm2 logs
docker-compose logs -f api

# 重启服务
pm2 restart all
docker-compose restart api

# 查看端口占用
lsof -i :8080
netstat -tlnp | grep 8080

# 磁盘使用
df -h

# 内存使用
free -m
```

---

## 8. 联系信息

| 角色 | 联系方式 |
|------|----------|
| 值班运维 | [填写] |
| 后端负责人 | [填写] |
| 前端负责人 | [填写] |
| DBA | [填写] |
| 安全负责人 | [填写] |
| 项目经理 | [填写] |
