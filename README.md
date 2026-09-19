# bt-linux-panel-skills

面向 Codex、ZCode、Kimi 及其他 Agent Skills 客户端的宝塔 Linux 面板可复用技能包。

本仓包含 6 个与宿主无关的技能，覆盖网站诊断、防火墙审计、安全审计、面板运维、MCP 接入和工具路由。插件专属的 `bt-harness` 不在本仓维护。

## 安装

```bash
npx skills add full-stack-skills/bt-linux-panel-skills
npx skills add full-stack-skills/bt-linux-panel-skills --skill bt-mcp-setup
```

## 维护门禁

```bash
python3 scripts/lint_skills.py
python3 scripts/trace_gate.py \
  --evaluator ../agent-skills/skills/skill-trace-evaluation/scripts/trace_evaluate.py \
  --threshold 4.5
```

正式 `v*` Release 发布后，仓库会把不可变 tag 与 peeled commit SHA 发送给 `full-stack-plugins/bt-linux-panel-plugin`，由插件自动创建技能升级 PR。

<!-- FULL_STACK_DOC_START -->
## 项目定位与边界

`bt-linux-panel-skills` 是包含 **6 个可独立安装 Agent Skill** 的源代码仓库，当前清单版本为 `0.1.0`。本仓负责技能的触发说明、工作流、references、examples 与质量门禁；宿主插件的 Hook、MCP、凭据注入和运行时脚本不属于本仓职责。

| 已确认事实 | 值 | 证据 |
|---|---|---|
| 安装包 | `full-stack-skills/bt-linux-panel-skills` | `.claude-plugin/plugin.json`、仓库远端 |
| 可安装技能 | 6 | `skills/*/SKILL.md` |
| 当前版本 | `0.1.0` | `.claude-plugin/plugin.json` |
| 规格事实源 | OpenSpec | `openspec/config.yaml` |
| 许可证 | Apache-2.0 | `LICENSE` |

### 不负责

- 不替代消费插件中的可执行 Harness、MCP 服务、Hook 或供应商客户端；
- 不把 `SKILL.md` 被复制到目录视为宿主已经发现、触发或成功执行；
- 不自动授权网络调用、付费生成、文件覆盖、上传或发布；
- 不允许消费插件直接修改受 `skills.lock.json` 管理的副本。

## 一眼看懂

```text
用户任务
  │
  ▼
name / description 发现技能
  │
  ▼
读取完整 SKILL.md ──► 按需加载 references / examples / scripts
  │
  ▼
执行领域工作流 ──► 收集验证证据 ──► PASS / FAIL / UNVERIFIED
```

## 已验证的安装与发现

```bash
npx skills add full-stack-skills/bt-linux-panel-skills
npx skills add full-stack-skills/bt-linux-panel-skills --skill bt-firewall-audit
npx skills list --json
```

固定发布版本时使用 GitHub Release/tag，不要把移动的 `main` 当成不可变版本。安装完成后应核对技能数量、名称、资源文件和目标 Agent 列表；Codex、ZCode、Kimi 的真实插件加载仍需分别验证。

### 可安装技能

| 技能 | 触发范围摘要 |
|---|---|
| `bt-firewall-audit` | 宝塔部署服务器的防火墙规则审计专家：检查 iptables/firewalld/ufw 规则、端口开放、IP 黑白名单、规则冲突、最小化原则。当用户希望审查防火墙配置或排查端口访问问题时使用本 skill。 |
| `bt-mcp-setup` | 宝塔 Linux 面板 13.0 的 MCP 协议接入指南。覆盖安装前置、8765 端口放行、IP 白名单、Codex/ZCode/Kimi/Claude Code 的 MCP 客户端配置示例、Bearer Token 鉴权、HTTPS 证书要求、错误排查表。当用户希望把 AI 编码助手接入宝塔面板、用自然语言远程操作 98 个面板工具时使用本 skill。 |
| `bt-mcp-tools` | 宝塔 Linux 面板 13.0 的 MCP 协议 98 个工具分类速查。按 19 个分类（基础/网站/网络/数据库/服务/Docker/系统/软件/防火墙/Java/Node/Python/Go/反代/HTML/SSH/安全/计划任务/通知）组织，每行标注工具名、风险等级、一句话功能、关键参数、与通用工具的映射。当 Agent 已接入 baota-mcp  |
| `bt-panel-ops` | 宝塔 Linux 面板综合运维：网站/数据库/服务/Docker/防火墙/SSL/计划任务/通知全场景综合诊断。当用户需要排查宝塔面板部署环境的运维问题时，本 skill 整合 14 个宝塔内置 skill agents 的诊断要点，给出结构化的诊断流程、命令清单与跨域关联分析。读这个 skill 来了解宝塔部署约定、必走路径与跨域关联排查思路。 |
| `bt-security-audit` | 宝塔部署服务器的安全审计专家：检查异常进程、SUID 文件、Webshell、SSH 安全、登录日志、可疑定时任务、入侵迹象、密码策略。当用户希望对宝塔服务器做安全合规检查时使用本 skill。 |
| `bt-site-ops` | 宝塔部署环境下的网站诊断专家：分析 Nginx/Apache 配置、PHP-FPM、SSL 证书、数据库连接、响应慢、502/504、目录权限等问题。当用户报告"网站无法访问/慢/出错"且主机是宝塔部署时使用本 skill。 |

## 包结构与加载规则

```text
bt-linux-panel-skills/
├── .claude-plugin/plugin.json   # 包名、版本与技能清单
├── skills/<name>/SKILL.md       # 触发条件与主工作流
├── skills/<name>/references/    # 按任务加载的领域知识
├── skills/<name>/examples/      # 请求、验收与恢复示例
├── scripts/                     # 仓库级生成和质量门禁（若存在）
├── openspec/                    # 规格与归档变更
└── LICENSE
```

跨技能协作必须使用技能名和安装命令，不得依赖 `../sibling-skill/` 相对链接，因为用户可能只安装一个技能。

## 质量、发布与安全

```bash
python3 scripts/lint_skills.py
```

发布前还必须检查 frontmatter、相对链接、资源完整性、TRACE 阈值、版本清单以及干净环境安装。正式 tag 不得移动；内容变化应发布新版本，并让消费插件通过 tag、peeled SHA 和摘要更新锁文件。

安全边界：不得提交真实密钥、账号、本机绝对路径或私有仓库地址；脚本应默认最小权限，付费、上传、删除和覆盖动作必须保留显式授权门。

## 故障排查

| 现象 | 检查 | 处理 |
|---|---|---|
| 安装后未发现技能 | frontmatter、Agent 发现目录、是否需要刷新 | 用 `skills list --json` 核对实际发现结果 |
| 只安装单个技能后引用缺失 | 是否存在跨技能相对路径 | 把必需资源移入当前技能，或按名称安装依赖技能 |
| 插件完整性检查失败 | tag、peeled SHA、摘要和本地技能清单 | 在源技能仓发布新版本，再由同步 PR 更新插件 |
| 工具或凭据缺失 | `compatibility`、运行时前置条件 | 报告 `UNVERIFIED`，不要猜测成功 |
| 自动化第二次运行仍产生差异 | 生成器非幂等或清单漂移 | 阻止发布并修复生成/排序规则 |
<!-- FULL_STACK_DOC_END -->

## 许可证

Apache License 2.0。
