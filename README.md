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

## 许可证

Apache License 2.0。
