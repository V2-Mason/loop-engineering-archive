# Roadmap — Mason's builder/reviewer loop

长期方向见 Vision / Principles(本文件末)。版本时间线如下。

> 注:本项目采纳 `cobusgreyling/loop-engineering` 地基,已跳过早期模板设想的 "PR workflow / reviewer loop" 阶段——它们在 v0.1 就完成了。

| 版本 | 状态 | 交付目标 |
|------|------|----------|
| v0.0 bootstrap | 完成 | fork、Claude Code App + Codex App、`CLAUDE_CODE_OAUTH_TOKEN` secret、双厂配置 |
| v0.1 双厂最小闭环 | 完成 | 单 issue：builder(Claude) → PR → reviewer(Codex) → 人 merge，真实跑通一次(issue #2 → PR #4) |
| v0.2 Discipline | 完成 | 单-issue 纪律护栏：smoke CI + Codex 自动审 + 修复轮数机械封顶 + 触发治理 + 分支保护；自动开 PR(#B)取消；Path A(codex-gate + return-loop)已 live。UIPI 未降(≈2),详见 `mason-loop/loops/retro-002.md` |
| **v0.2.5 Reliability / Convergence** | 计划（下一轮） | 补"可靠性轴"（retro-002）：reviewer 换 **Codex CLI 跑 loop-verifier**(P3)+ `loop-context`/`loop-ledger` 收敛 review↔fix(P2)+ `loop-cost`/`loop-run-log` 机械测 UIPI(P5)+ 状态从 GitHub 派生(P4)。目标：单-issue UIPI 真降到 ~1 且稳定，再上 v0.3 |
| v0.3 Triage / Intake | 计划 | 多-issue：去重、冲突/重复检测、优先级、backlog 状态 + Projects 排序 |
| v0.4 Orchestration | 计划 | 评估 GitHub Agentic Workflows(gh-aw);或更细的 Claude/Codex 分工 |
| v0.5 Release | 计划 | tag / release pipeline / release notes；多 issue 并行(worktree 隔离) |

## Vision / Principles(长期边界)

- 不自动 merge：人是最终质量闸门。
- agent 行为全程可追踪：一切经 issue / PR / comment / state 文件留痕。
- GitHub-native 优先：不自建 orchestrator，拼装现成组件。
- 一份 `AGENTS.md` 为跨 agent 共享契约;工具差异用薄适配层(`.claude/` / `.codex/`)隔开。
- 有指标地迭代：北极星 = UIPI(每 issue 计划外人工介入次数)，逐轮递减。
- 一轮只推进一个方向；先跑通再打磨。
- **治理文件(constitution)与产品工作(product)分开**：自主 agent 只在 product 上跑，constitution 由人手写从严审。
- **方向由人写的 spec 给定，agent 只执行**：正确性靠 spec 的 DoD + reviewer intent-check + 人 merge。
