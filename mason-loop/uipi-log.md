# UIPI Log — 双厂 builder/reviewer 开发闭环

> 北极星指标 **UIPI(每 issue 计划外人工介入次数)** 的结构化记录,取代此前"目测"(v0.2.5-R1)。
> 度量对象:走完 `issue → builder → PR → reviewer → merge` 的**产品功能 issue**。
> 运维型 loop(daily-triage 等)的 per-run 记录在根目录 `loop-run-log.md`,与本文件分开,勿混。

## UIPI 定义:什么算一次"计划外人工介入"

**两个 sanctioned(计划内)批准点不计**:① spec 确认(issue 内 @claude 放行);② 最终 merge。

**这两点之外、任何由人推动流程前进的动作,各计 1 次**,例如:
- 手动 `@codex review` 触发或催审查
- 手动 resolve reviewer 线程
- 手动开 PR(#B 取消后的"一键开 PR" —— 已知常驻介入;暂计入 UIPI。若日后认定"一键"可接受为第 3 个 sanctioned 点,再重分类)
- 替 builder 手动推修复(如撞 workflow-file 限制时)
- 手动 re-run 卡住的检查

**不计**:agent 自动完成的步骤(如 codex-request 自动 @codex)、CI 自动跑。

## Per-issue 记录

| Issue | 版本 | PR | UIPI | 介入明细 | 备注 |
|---|---|---|---|---|---|
| #2 | v0.1 | #4 | 2 | ① 手动开 PR ② 手动 @codex | v0.1 基线(retro-001) |
| #16 | v0.2 | #17 | 2 | ① 一键开 PR ② 手动 @codex(autoreview 抽风) | v0.2 端到端验收(retro-002) |
| #23* | v0.2 | #23 | 0 | (无计划外介入) | *文档/constitution PR,无 issue 前端;codex-request **自动** @codex 已验证;Codex 审 clean 无需 resolve |

## 度量方法(v0.2.5-R1)

- 当前:**半机械**——闭环走完后按上表结构补录;介入明细可从 PR 时间线核对(谁在 sanctioned 点之外发了推动性动作)。
- 后续轮次(Backlog):从 GitHub 事件**全自动**推断介入(区分 bot 自动 vs 人手动),免人工补录。

## 趋势

- v0.1 → v0.2:UIPI 持平 ≈ 2(纪律管道建成,但介入未减)。
- Path A 上线后:codex-request 自动 @codex → "手动 @codex"这个介入点消除(#23 已验证)。
- **待验**:下一个真实产品 issue(v0.2.5 之后)应实测 UIPI ≈ 1,验证是否真降。
