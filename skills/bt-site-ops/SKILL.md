---
name: bt-site-ops
description: 宝塔部署环境下的网站诊断专家：分析 Nginx/Apache 配置、PHP-FPM、SSL 证书、数据库连接、响应慢、502/504、目录权限等问题。当用户报告"网站无法访问/慢/出错"且主机是宝塔部署时使用本 skill。
metadata:
  category: 网站运维
  target: 宝塔面板部署的网站（Nginx/Apache + PHP-FPM + MySQL）
license: Apache-2.0
---

# 宝塔网站诊断专家

## 诊断流程（5 步）

### 第一步：基础健康快照

```bash
# 系统级
uptime
free -m
df -h
top -bn1 | head -20

# 网站服务
systemctl status nginx
systemctl status php-fpm
systemctl status mysqld

# 网络监听
ss -tlnp | grep -E ':80|:443|:9000|:3306'
```

### 第二步：Nginx/Apache 配置审计

```bash
# vhost 配置位置
ls /www/server/panel/vhost/nginx/        # Nginx
ls /www/server/panel/vhost/apache/       # Apache

# 检查语法
nginx -t
apachectl configtest

# 检查关键配置项
grep -E "fastcgi_pass|root\s|listen\s|ssl_certificate" /www/server/panel/vhost/nginx/<site>.conf
```

### 第三步：PHP-FPM 状态

```bash
# 进程数
ps aux | grep php-fpm | wc -l

# 慢日志
ls -la /www/server/php/<ver>/var/log/
tail -100 /www/server/php/74/var/log/php-fpm.slow.log 2>/dev/null

# 错误日志
tail -100 /www/server/php/74/var/log/php-fpm.log

# 配置
grep -E "pm.max_children|pm.start_servers|request_terminate_timeout" /www/server/php/74/etc/php-fpm.conf
```

### 第四步：MySQL 慢查询

```bash
# 慢查询日志
ls -la /www/server/mysql/mysql<ver>.log
tail -100 /www/server/data/mysql-slow.log 2>/dev/null

# 进程列表
mysqladmin -uroot -p<pwd> processlist

# 关键配置
mysql -e "SHOW VARIABLES LIKE 'slow_query_log'"
mysql -e "SHOW VARIABLES LIKE 'max_connections'"
mysql -e "SHOW STATUS LIKE 'Threads_connected'"
```

### 第五步：综合分析与修复建议

## 常见问题与修复

### 502 Bad Gateway

**症状**：网站返回 502

**根因排查**：
1. PHP-FPM 未运行：`systemctl status php-fpm`
2. 端口未监听：`ss -tlnp | grep 9000`
3. vhost 中 fastcgi_pass 路径错误
4. PHP-FPM 进程数耗尽（`pm.max_children` 不够）

**修复**：
```bash
# 重启 PHP-FPM
systemctl restart php-fpm

# 调大进程数（按内存算：每进程约 30MB）
sed -i 's/pm.max_children = .*/pm.max_children = 50/' /www/server/php/74/etc/php-fpm.conf
systemctl reload php-fpm
```

### 网站响应慢

**诊断路径**：
1. **网络层**：用 `curl -w "time_total: %{time_total}\n" -o /dev/null -s <site>` 测试响应时间
2. **数据库层**：检查慢查询、连接数、锁等待
3. **PHP 层**：检查 PHP-FPM 慢日志、OPcache 是否启用
4. **磁盘层**：`iostat -x 1 5`、`iotop`
5. **应用层**：开启 XHProf / Tideways 跟踪

**常见优化**：
```bash
# 启用 OPcache
grep "opcache.enable" /www/server/php/74/etc/php.ini
# 应为 opcache.enable=1

# 检查静态资源缓存
grep -E "expires|cache-control" /www/server/panel/vhost/nginx/<site>.conf
```

### SSL 证书问题

```bash
# 检查证书有效期
openssl x509 -text -noout -in /www/server/panel/vhost/cert/<site>/fullchain.pem | grep -E "Not Before|Not After"

# 检查证书链
openssl s_client -connect <site>:443 -servername <site> < /dev/null 2>&1 | grep -E "Verify return code|subject=|issuer="

# Let's Encrypt 续签
certbot renew --dry-run
```

### 磁盘占满

```bash
# 找出大目录
du -sh /www/* | sort -hr | head -10
du -sh /www/wwwlogs/* | sort -hr | head -10  # 访问日志
du -sh /www/server/mysql/data/* | sort -hr | head -10  # 数据库

# 清理日志（保留最近 7 天）
find /www/wwwlogs -name "*.log" -mtime +7 -delete
```

## MCP 集成工具映射

如已接入 baota-mcp：
- `SiteList` → 列出所有站点
- `SiteGetConfig` → 取单个站点完整配置
- `SiteLogs` → 取访问/错误日志
- `SiteTraffic` → 取流量数据
- `SiteSSLDeploy` → 一键部署证书
- `SiteSSLApply` → 申请 Let's Encrypt

## 输出模板

诊断报告：
1. **症状**（用户报告的 + 你观察到的）
2. **证据**（命令输出片段、日志行、配置片段）
3. **根因**（指向具体配置项或资源瓶颈）
4. **修复步骤**（按风险从低到高排序）
5. **验证方法**（修复后如何确认问题解决）
6. **预防建议**（如何避免再次发生）

<!-- QUALITY_CONTRACT_START -->
## 什么时候使用

✅ 适用：

1. 用户明确要诊断宝塔网站、Nginx、PHP-FPM、数据库与证书故障。
2. 已提供或可安全取得必要上下文，需要得到可验证的 `site-diagnosis-report`。
3. 需要按最小权限、可回滚方式执行，并保留审计证据。

⚠️ 先澄清：

1. 目标环境、授权边界或成功标准缺失时，先给出只读假设方案并列出缺失项。
2. 涉及生产环境变更时，先确认备份、维护窗口和回滚路径。
3. 输入可能含敏感信息时，只引用字段名和脱敏片段，不复制完整凭据。

❌ 不该用：

1. 在未确认影响范围时重启生产服务或改写站点配置。
2. 用户只要概念解释且没有执行或交付需求。
3. 需要绕过鉴权、证书校验、人工确认或其他安全控制的请求。

## Workflow

Step 1：确认目标、环境、授权范围和不可变约束；信息不足时先产出带假设的只读版本。

Step 2：盘点现状与依赖，只读取必要数据，不记录令牌、密码、Cookie 或完整个人数据。

Step 3：选择最小影响路径，将高风险动作、外部网络调用和可逆步骤明确标注。

Step 4：生成或执行 `site-diagnosis-report`，每一步都绑定输入、预期输出与失败条件。

Step 5：校验结构、事实来源和目标状态；禁止根据缺失证据编造成功结论。

Step 6：失败时停止扩大影响，输出已完成步骤、失败证据、恢复点和下一次安全重试条件。

Step 7：交付摘要、验证证据、剩余风险与后续动作；生产变更必须说明回滚是否已验证。

## Rules

- 默认只读；写操作、高危操作和付费调用必须获得与该动作匹配的明确授权。
- 本技能不收集、不存储、不上传用户凭据；日志和报告不得包含完整 token、密码或密钥。
- 不关闭 TLS 校验，不执行来源不明脚本，不使用管道下载后直接执行。
- 只把真实执行结果写成“已完成”；计划、示例和推断必须显式标注。
- 优先幂等操作；无法幂等时先提供预演、备份和回滚点。

## Validation checklist

- [ ] 目标、环境与授权范围均已写明。
- [ ] `故障层级、根因证据、修复建议和验证请求完整` 已由可复现证据验证。
- [ ] 敏感数据已脱敏，输出中没有完整凭据。
- [ ] 失败与超时路径已覆盖，未出现无限重试。
- [ ] 变更类任务具有备份或回滚说明。
- [ ] 最终结论区分事实、推断和未验证项。

## Gotchas

1. **授权不等于可达**：有权限但网络、证书或白名单不满足时，仍应停止并报告连接证据。
2. **成功码不等于业务成功**：必须检查 `故障层级、根因证据、修复建议和验证请求完整`，不能只看命令退出码或 HTTP 200。
3. **重试不等于恢复**：对鉴权失败、参数错误和安全拒绝不得盲目重试。
4. **示例不等于现状**：模板值与占位符不能写成真实环境数据。
5. **输出不等于交付**：还需完成结构校验、风险说明和可重复验证。
6. **跨环境不可照搬**：操作系统、版本、区域和宿主能力不同时必须重新确认参数。

## 渐进式资料

- 做路径选择前读取 `references/decisions/decision-guide.md`。
- 遇到异常、超时或部分成功时读取 `references/operations/failure-matrix.md`。
- 完成交付前读取 `references/operations/validation-checklist.md`。
- 需要可复制输入时，按顺序参考 `examples/basic.md`、`examples/failure.md`、`examples/advanced.md`。
<!-- QUALITY_CONTRACT_END -->
