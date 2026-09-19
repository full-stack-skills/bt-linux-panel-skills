---
name: bt-firewall-audit
description: 宝塔部署服务器的防火墙规则审计专家：检查 iptables/firewalld/ufw 规则、端口开放、IP 黑白名单、规则冲突、最小化原则。当用户希望审查防火墙配置或排查端口访问问题时使用本 skill。
metadata:
  category: 网络与防火墙
  target: Linux 防火墙（自动识别 firewalld/ufw/iptables）
license: Apache-2.0
---

# 宝塔服务器防火墙审计专家

## 自动识别防火墙类型

```bash
# 检测
systemctl is-active firewalld    # CentOS/RHEL
systemctl is-active ufw          # Ubuntu/Debian
which firewalld ufw iptables
```

不同发行版命令差异大，先识别类型再操作。

## firewalld 审计（CentOS/RHEL）

```bash
# 状态
systemctl status firewalld
firewall-cmd --state

# 全部规则
firewall-cmd --list-all

# 已开放端口
firewall-cmd --list-ports
# 输出示例：80/tcp 443/tcp 8888/tcp

# 已开放服务
firewall-cmd --list-services

# 富规则（IP 黑白名单）
firewall-cmd --list-rich-rules

# 配置
cat /etc/firewalld/zones/public.xml
ls -la /etc/firewalld/zones/

# 永久配置 vs 运行时（差异检查）
firewall-cmd --permanent --list-all
firewall-cmd --runtime-to-permanent   # 把运行时同步到永久
```

## ufw 审计（Ubuntu/Debian）

```bash
# 状态
systemctl status ufw
ufw status verbose

# 带编号的规则
ufw status numbered

# 应用配置
ufw app list
cat /etc/ufw/user.rules
cat /etc/ufw/before.rules
```

## iptables 审计（通用/老系统）

```bash
# 规则列表（详细）
iptables -L -n -v --line-numbers
iptables -S

# IPv6
ip6tables -L -n -v

# NAT 表
iptables -t nat -L -n -v

# 默认策略
iptables -L | grep "policy"

# 规则持久化
cat /etc/iptables/rules.v4
cat /etc/iptables/rules.v6
ls /etc/sysconfig/iptables*
```

## 关键审计项

### 1. 默认策略

```bash
# INPUT 链默认应该是 DROP 或 REJECT（白名单模式）
iptables -L INPUT | head -1
iptables -L INPUT | grep "policy"

# ufw 默认应该是 deny incoming
grep "DEFAULT_INPUT_POLICY" /etc/default/ufw
```

### 2. 必需端口放行检查

| 端口 | 用途 | 应放行 |
|------|------|--------|
| 22 | SSH | ✅ |
| 80 | HTTP | ✅（如对外服务）|
| 443 | HTTPS | ✅（如对外服务）|
| 8888 | 宝塔面板 | ⚠️ 仅管理员 IP |
| 8765 | 宝塔 MCP 服务 | ⚠️ 仅 Agent IP |
| 3306 | MySQL | ❌ 默认禁对外 |
| 6379 | Redis | ❌ 默认禁对外 |
| 11211 | Memcached | ❌ 默认禁对外 |

### 3. 端口监听情况

```bash
ss -tlnp
netstat -tlnp

# 危险开放（数据库/缓存对公网开放 = 严重风险）
ss -tlnp | grep -E ":3306|:6379|:11211|:27017|:9200|:9300|:5432"
```

### 4. IP 黑白名单审计

```bash
# firewalld 富规则中的 IP 限制
firewall-cmd --list-rich-rules | grep -E "rule family|source address"

# ufw 中的 IP 规则
ufw status | grep -E "ALLOW|DENY" | grep -v "Anywhere"

# iptables 中针对特定 IP 的规则
iptables -L -n | grep -v "0.0.0.0/0"
```

### 5. 规则冲突与冗余

```bash
# 统计规则数（过多 = 管理混乱）
iptables -L | wc -l

# 找重复规则
iptables-save | sort | uniq -d

# 找被后续规则覆盖的规则
iptables -L INPUT -n --line-numbers
# 检查 -A 与 -I 的顺序
```

## 端口无法访问的诊断流程

```bash
# 1. 服务是否在监听？
ss -tlnp | grep :<port>

# 2. 防火墙是否放行？
firewall-cmd --query-port=<port>/tcp
ufw status | grep <port>
iptables -L -n | grep <port>

# 3. 宝塔面板的防火墙（独立于系统防火墙）
bt firewall status 2>/dev/null

# 4. 云安全组（最常被忽略）
# 登录云厂商控制台检查

# 5. 测试连通性（从本地）
curl -v telnet://localhost:<port>
nc -zv localhost <port>

# 6. 抓包定位
tcpdump -i any port <port> -nn
```

## 高危配置警示

| 风险 | 配置 |
|------|------|
| 🔴 数据库对公网开放 | 3306/6379/5432 监听 0.0.0.0 |
| 🔴 面板端口对公网 | 8888 未限制 IP |
| 🟠 默认 ACCEPT 策略 | INPUT chain policy ACCEPT |
| 🟠 22 端口对全网开放 + 弱密码 | SSH 暴力破解入口 |
| 🟡 大段 IP 放行 | `0.0.0.0/0` 规则过多 |

## MCP 集成工具

如已接入 baota-mcp：
- `FirewallStatus`：面板防火墙状态
- `FirewallPortList`：端口规则查询
- `FirewallIpList`：IP 规则查询
- `FirewallPortSet`（**high**）：增删端口规则（需授权）
- `FirewallIpSet`（**high**）：增删 IP 规则（需授权）

## 输出模板

```
# 防火墙审计报告

## 防火墙类型
firewalld / ufw / iptables（自动识别）

## 当前状态
- 运行：✅ / ❌
- 默认策略：INPUT DROP/ACCEPT
- 规则总数：N

## 关键端口审计
| 端口 | 服务 | 监听 | 防火墙 | 公网可达 | 风险 |
|------|------|------|--------|---------|------|
| 22 | SSH | ✅ | ✅ | ⚠️ 全网 | 🟠 |
| 8888 | BT 面板 | ✅ | ✅ | ⚠️ 全网 | 🔴 |
| 3306 | MySQL | ✅ | ❌ | ⚠️ 0.0.0.0 | 🔴 |

## 风险发现
1. MySQL 3306 对全网开放（🔴紧急）
2. ...

## 修复建议
```bash
# 1. 限制 MySQL 仅本地访问
firewall-cmd --permanent --remove-port=3306/tcp
firewall-cmd --reload

# 2. 限制宝塔面板仅中国/管理 IP
firewall-cmd --permanent --remove-port=8888/tcp
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=<管理IP>/32 port port=8888 protocol=tcp accept'
firewall-cmd --reload
```
```

<!-- QUALITY_CONTRACT_START -->
## 什么时候使用

✅ 适用：

1. 用户明确要审计宝塔主机端口暴露、防火墙规则和云安全组差异。
2. 已提供或可安全取得必要上下文，需要得到可验证的 `firewall-audit-report`。
3. 需要按最小权限、可回滚方式执行，并保留审计证据。

⚠️ 先澄清：

1. 目标环境、授权边界或成功标准缺失时，先给出只读假设方案并列出缺失项。
2. 涉及生产环境变更时，先确认备份、维护窗口和回滚路径。
3. 输入可能含敏感信息时，只引用字段名和脱敏片段，不复制完整凭据。

❌ 不该用：

1. 直接修改防火墙或云安全组而没有动作级授权。
2. 用户只要概念解释且没有执行或交付需求。
3. 需要绕过鉴权、证书校验、人工确认或其他安全控制的请求。

## Workflow

Step 1：确认目标、环境、授权范围和不可变约束；信息不足时先产出带假设的只读版本。

Step 2：盘点现状与依赖，只读取必要数据，不记录令牌、密码、Cookie 或完整个人数据。

Step 3：选择最小影响路径，将高风险动作、外部网络调用和可逆步骤明确标注。

Step 4：生成或执行 `firewall-audit-report`，每一步都绑定输入、预期输出与失败条件。

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
- [ ] `端口、规则来源、风险等级与修复建议均有证据` 已由可复现证据验证。
- [ ] 敏感数据已脱敏，输出中没有完整凭据。
- [ ] 失败与超时路径已覆盖，未出现无限重试。
- [ ] 变更类任务具有备份或回滚说明。
- [ ] 最终结论区分事实、推断和未验证项。

## Gotchas

1. **授权不等于可达**：有权限但网络、证书或白名单不满足时，仍应停止并报告连接证据。
2. **成功码不等于业务成功**：必须检查 `端口、规则来源、风险等级与修复建议均有证据`，不能只看命令退出码或 HTTP 200。
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
