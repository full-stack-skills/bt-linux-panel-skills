---
name: bt-mcp-tools
description: 宝塔 Linux 面板 13.0 的 MCP 协议 98 个工具分类速查。按 19 个分类（基础/网站/网络/数据库/服务/Docker/系统/软件/防火墙/Java/Node/Python/Go/反代/HTML/SSH/安全/计划任务/通知）组织，每行标注工具名、风险等级、一句话功能、关键参数、与通用工具的映射。当 Agent 已接入 baota-mcp 通道、需要选择合适工具时使用本 skill。
metadata:
  category: 运维工具参考
  target: 宝塔面板 13.0 MCP 工具集
  count: 98
license: Apache-2.0
---

# 宝塔面板 MCP 工具速查表（98 工具）

## 通用约定

- **返回结构**：`{"status": bool, "msg": string, ...}`，失败 `status=false`
- **风险等级**：
  - `low` 只读查询（绿色）
  - `medium` 有副作用（黄色）
  - `high` 高危操作（红色，删除/覆盖/执行命令）
- 工具总数 98，按 19 类分组

---

## 一、基础工具（10 个，远程 Agent 专用）

| 工具 | 风险 | 功能 |
|------|------|------|
| `Read` | low | 读取文件（`file_path`, `offset`, `limit`） |
| `Edit` | high | 精确修改（`old_string`, `new_string`, `replace_all`） |
| `Write` | high | 写入/覆盖 |
| `Glob` | low | 按名查找文件 |
| `Grep` | low | 内容搜索（正则） |
| `LS` | low | 列目录 |
| `Bash` | high | 执行 Shell（`run_in_background`） |
| `BashStatus` | low | 查询后台任务 |
| `BashStop` | medium | 终止后台任务 |
| `Upload` | medium | 客户端上传文件 |

---

## 二、网站（15 个）

| 工具 | 风险 | 功能 |
|------|------|------|
| `SiteList` | low | 网站列表 |
| `SiteGetConfig` | low | 站点配置 |
| `SiteLogs` | low | 访问日志 |
| `SiteTraffic` | low | 站点流量 |
| `SiteCreate` | medium | 创建网站（`domain`, `port`, `php_version`） |
| `SiteDelete` | **high** | 删除网站 |
| `OneClickDeploy` | medium | 一键部署 CMS |
| `TrafficAnalysis` | low | 全站流量分析 |
| `SiteCertList` | low | 证书库列表 |
| `SiteSSLDeploy` | medium | 部署 SSL |
| `SiteSSLApply` | medium | 申请 SSL |
| `DomainManage` | **high** | 域名增删 |
| `SiteConfig` | medium | PHP/伪静态 |
| `SiteControl` | medium | 启停站点 |
| `SiteBackup` | medium | 备份站点 |

---

## 三、网络（3 个）

`WebFetch`（low）/ `ServerIP`（low）/ `Upload`（medium）

---

## 四、数据库（6 个）

| 工具 | 风险 | 功能 |
|------|------|------|
| `DatabaseList` | low | 数据库列表 |
| `DatabaseCreate` | medium | 创建（`name`, `db_user`, `password`） |
| `DatabaseDelete` | **high** | 删除 |
| `DatabaseBackup` | medium | 备份 |
| `MysqlQuery` | low | 只读 SQL |
| `MysqlExecute` | **high** | 写 SQL |

---

## 五、服务（2 个）

`ServiceStatus`（low）/ `ServiceControl`（medium）

---

## 六、Docker（10 个）

`ContainerList`（low）/ `ContainerLogs`（low）/ `ContainerInspect`（low）/ `ComposeList`（low）/ `ImageList`（low）/ `VolumeList`（low）/ `NetworkList`（low）/ `ContainerControl`（medium）/ `ContainerDelete`（**high**）/ `ImagePull`（medium）

---

## 七、系统（1 个）

`SystemInfo`（low）：CPU/内存/磁盘/负载/运行时长

---

## 八、软件商店（3 个）

`SoftwareList`（low）/ `SoftwareInstall`（medium）/ `SoftwareUninstall`（**high**）

---

## 九、防火墙（5 个）

`FirewallStatus`（low）/ `FirewallPortList`（low）/ `FirewallIpList`（low）/ `FirewallPortSet`（**high**）/ `FirewallIpSet`（**high**）

---

## 十、Java 项目（6 个）

`JavaJdk`（medium）/ `JavaProjectCreate`（medium）/ `JavaProjectInfo`（low）/ `JavaProjectControl`（medium）/ `JavaProjectModify`（medium）/ `JavaProjectDelete`（**high**）

---

## 十一、NodeJS 项目（6 个）

`NodeVersion`（medium）/ `NodeProjectCreate`（medium）/ `NodeProjectInfo`（low）/ `NodeProjectControl`（medium）/ `NodeProjectModify`（medium）/ `NodeProjectDelete`（**high**）

---

## 十二、Python 项目（8 个）

`PythonVersion`（medium）/ `PythonEnv`（medium）/ `PythonProjectCreate`（medium）/ `PythonProjectInfo`（low）/ `PythonProjectControl`（medium）/ `PythonProjectService`（medium）/ `PythonProjectModify`（medium）/ `PythonProjectDelete`（**high**）

---

## 十三、Go 项目（6 个）

`GoVersion`（medium）/ `GoProjectCreate`（medium）/ `GoProjectInfo`（low）/ `GoProjectControl`（medium）/ `GoProjectModify`（medium）/ `GoProjectDelete`（**high**）

---

## 十四、反向代理（5 个）

`ProxyProjectCreate`（medium）/ `ProxyProjectInfo`（low）/ `ProxyProjectModify`（medium）/ `ProxyWriteConfig`（medium）/ `ProxyProjectDelete`（**high**）

---

## 十五、HTML 静态项目（4 个）

`HtmlProjectInfo`（low）/ `HtmlProjectCreate`（medium）/ `HtmlProjectModify`（medium）/ `HtmlProjectDelete`（**high**）

---

## 十六、SSH（3 个）

`SSHInfo`（low）/ `SSHConfig`（**high**）/ `SSHIntrusion`（low）

---

## 十七、安全（1 个）

`SecurityCheck`（low）：安全评分、10 项检查、SSH 危险命令历史

---

## 十八、计划任务（2 个）

`GetCrontab`（low）/ `ManageCrontab`（**high**）

---

## 十九、通知（3 个）

`GetChannel`（low）/ `ManageChannel`（**high**）/ `SendMessage`（medium）

---

## 工具选择决策树

```
目标是什么？
├─ 只读查询
│  ├─ 本机 → 用通用工具（Bash + grep + read）更轻量
│  └─ 远端 → MCP low 工具
├─ 创建资源（站点/数据库/项目）
│  ├─ 本机 → 通用工具 + 手工命令
│  └─ 远端 → MCP medium 工具（先确认）
└─ 修改/删除资源
   ├─ 本机 → 通用工具（高危，需授权）
   └─ 远端 → MCP high 工具（**每次二次确认**）
```

## 风险等级速查

**只读**（low）：`SiteList` / `DatabaseList` / `SystemInfo` / `ServiceStatus` / `ContainerList` / `FirewallStatus` / `GetCrontab` / `GetChannel` / `TrafficAnalysis` / `*ProjectInfo` / `SSHInfo` / `SSHIntrusion` / `SecurityCheck`

**中风险**（medium）：`SiteCreate` / `DatabaseCreate` / `ServiceControl` / `SoftwareInstall` / `*ProjectCreate` / `*ProjectControl` / `*ProjectModify`（除 Delete）/ `SiteSSL*` / `SiteConfig` / `ContainerControl` / `ImagePull` / `BashStop` / `SendMessage` / `Upload`

**高风险**（high，每次必须用户明确确认）：`Edit` / `Write` / `Bash` / `SiteDelete` / `DomainManage` / `DatabaseDelete` / `MysqlExecute` / `ContainerDelete` / `SoftwareUninstall` / `FirewallPortSet` / `FirewallIpSet` / `*ProjectDelete` / `SSHConfig` / `ManageCrontab` / `ManageChannel`

<!-- QUALITY_CONTRACT_START -->
## 什么时候使用

✅ 适用：

1. 用户明确要按风险等级选择宝塔 MCP 工具并生成调用计划。
2. 已提供或可安全取得必要上下文，需要得到可验证的 `mcp-tool-plan`。
3. 需要按最小权限、可回滚方式执行，并保留审计证据。

⚠️ 先澄清：

1. 目标环境、授权边界或成功标准缺失时，先给出只读假设方案并列出缺失项。
2. 涉及生产环境变更时，先确认备份、维护窗口和回滚路径。
3. 输入可能含敏感信息时，只引用字段名和脱敏片段，不复制完整凭据。

❌ 不该用：

1. 绕过工具风险分级直接执行高危操作。
2. 用户只要概念解释且没有执行或交付需求。
3. 需要绕过鉴权、证书校验、人工确认或其他安全控制的请求。

## Workflow

Step 1：确认目标、环境、授权范围和不可变约束；信息不足时先产出带假设的只读版本。

Step 2：盘点现状与依赖，只读取必要数据，不记录令牌、密码、Cookie 或完整个人数据。

Step 3：选择最小影响路径，将高风险动作、外部网络调用和可逆步骤明确标注。

Step 4：生成或执行 `mcp-tool-plan`，每一步都绑定输入、预期输出与失败条件。

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
- [ ] `工具、参数、风险、授权和预期输出一一对应` 已由可复现证据验证。
- [ ] 敏感数据已脱敏，输出中没有完整凭据。
- [ ] 失败与超时路径已覆盖，未出现无限重试。
- [ ] 变更类任务具有备份或回滚说明。
- [ ] 最终结论区分事实、推断和未验证项。

## Gotchas

1. **授权不等于可达**：有权限但网络、证书或白名单不满足时，仍应停止并报告连接证据。
2. **成功码不等于业务成功**：必须检查 `工具、参数、风险、授权和预期输出一一对应`，不能只看命令退出码或 HTTP 200。
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
