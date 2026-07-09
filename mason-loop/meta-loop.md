# 元 loop 操作手册 v0.2（双厂 maker/checker 架构）

## 一、这是什么

本项目分两层：

- **运行层**：一个 GitHub repo 内的双厂开发闭环。builder（Claude Code GitHub App）实现 issue 并开 PR，reviewer（Codex GitHub App）审查并贴评论，builder 修复，人最终 merge。
- **元层（元 loop）**：一套改进运行层本身的迭代机制。对工作流的每一次改进，都作为一个 issue 交给当前版本的工作流自己去实施；每轮结束做一次 retro，retro 的产出回写 roadmap，成为下一轮的输入。

核心原理：对工作流的每一次改进，均以该工作流自身为执行工具与应用对象。完整模型由每轮 retro 的改进项逐步累积形成，每一轮只回答一个问题："这一轮里，人在哪里被迫介入了，如何让下一轮不再需要？"

## 二、运行层：v0.1 双厂最小闭环

### 角色分配

| 角色 | 承担者 | 职责 |
|------|--------|------|
| Spec 作者 | Mason + 本地 Claude Code | 写 spec 与 issue，确认后进入运行层 |
| Builder | Claude Code GitHub App | issue 内 @claude → 建分支、实现、自测、开 PR；PR 评论内 @claude → 向 PR 分支推修复 |
| Reviewer | Codex GitHub App | PR 打开即 Automatic review（或 @codex review 触发），读 AGENTS.md 的 Review guidelines，贴 P0/P1 评论；只评论，不改代码 |
| 质量闸门 | Mason | 确认 spec；最终 merge |

builder 与 reviewer 是两个不同厂的独立 agent，不共享运行时内存。它们唯一的共同记忆是仓库内文件：

- `AGENTS.md` — Codex 从这里读 Review guidelines；同时是 builder 的项目惯例与 build/test 命令来源。双厂共用的单一契约。
- `loop-constraints.md` — 机器可读的绑定约束（最多修复轮数、denylist、禁自动 merge），loop-constraints skill 每轮开跑前读取。
- `STATE.md` / `pr-babysitter-state.md` — 跨轮状态与人工覆盖记录。
- `loop-ledger.json` — loop-guard 熔断器的尝试记录。

### 事件流

```
Mason 开 issue（写清 spec，@claude 实现）          ← 人工介入点 1（spec）
      │
      ▼
Claude builder：建分支 → 实现 → 自测 → 开 PR
      │
      ▼
Codex reviewer：Automatic review 自动触发，读 AGENTS.md，贴 P0/P1 评论
      │
      ▼
Claude builder：@claude 响应评论 → 推修复 commit（最多 N 轮，loop-guard 熔断）
      │
      ▼
Codex reviewer：新 commit 上复审
      │
      ▼
CI / smoke test 通过
      │
      ▼
Mason merge                                        ← 人工介入点 2（merge）
```

两个 agent 皆为 GitHub App，靠 app 集成响应事件，不走 workflow_run 链，因此天然规避了默认 GITHUB_TOKEN 创建的 PR 不触发下游 workflow 这一坑。

### 边界规则（防循环，已由本 repo 的 loop-constraints.md 落地）

- 自动修复最多 3 轮，超过即升级给人（loop-guard + loop-ledger.json 机械执行）。
- 禁自动 merge，push 前先告知。
- denylist 路径（secrets/auth/payments/infra、核心文档、审计逻辑）改动必须人批准。
- merge 永远由人执行。

## 三、元层：元 loop 单轮定义

一轮 = 一次对工作流本身的最小改进，五步：

1. **选题**：从 roadmap 取最小的下一个能力点（一轮只做一项）。
2. **Spec**：在本地写成 3-5 行的 spec（目标、可观察判据、不做项），Mason 确认。〔人工介入点 1〕
3. **实施**：把 spec 开成 issue，交给当前版本的工作流自己跑完 build → review → fix；Mason merge。〔人工介入点 2〕
4. **Retro**：merge 后按模板（loops/retro-NNN.md）记录，核心问题只有一个——本轮在两个批准点之外，人被迫介入了几次、分别因为什么。
5. **回写**：retro 中的改进项进入 roadmap 排序，本轮关闭。

节奏：每轮时间盒子 1-2 小时（不含 agent 运行等待）；一轮只改一项，同一问题 3 轮不收敛即停止并质疑架构。

## 四、北极星指标

**每 issue 的计划外人工介入次数（UIPI）**：在 spec 确认与最终 merge 两个批准点之外，人为推动流程所做的任何操作（手动触发审查、手动改代码、手动发修复指令）计 1 次。结构化记录在 `mason-loop/uipi-log.md`（每 issue 一条，含介入明细 + 计数；定义与判定标准见该文件）。注：根目录 `loop-run-log.md` 是运维型 loop(daily-triage)的 per-run 日志，与 UIPI 分开。

- 目标方向：UIPI 递减；v0.1 基线预计 2-4 次，v0.5 目标 ≤1 次。
- 质量护栏：merge 后引发 revert/补丁的比例不升高。指标下降但护栏破损视为无效改进。
- 每轮 retro 必须记录本轮 UIPI 实测值。

## 五、路线图（每个版本 = 若干轮元 loop）

| 版本 | 交付目标 |
|------|----------|
| v0.0 bootstrap | fork 已克隆并 loop-ready（audit 分 100/L3）；安装 Claude Code App 与 Codex App；测试 issue @claude 能开 PR、测试 PR 能被 Codex 自动审 |
| v0.1 双厂最小闭环 | 一个真实 issue 走完 builder(Claude) → reviewer(Codex) → fix → 人工 merge；本手册第二节全部成立 |
| v0.2 纪律层 | smoke test CI、PR 模板、修复轮数上限用 label 计数、把"人手动触发修复/复审"这次介入自动化 |
| v0.3 spec/plan 层 | docs/specs 成为 builder 输入；issue 由 plan 拆解生成 |
| v0.4 编排框架 | 评估把双厂编排迁到 GitHub Agentic Workflows（gh-aw，可按 workflow 指定 engine=claude / engine=codex） |
| v0.5 发布与并行 | release pipeline、自动 release notes、多 issue 并行（worktree 隔离，参考 loop-worktree 与 docs/multi-loop.md） |

注：异构 reviewer 在本架构里即 v0.1 的设计（Codex 当 reviewer），非后续里程碑。

Vision / Principles 由不做清单固定：不自动 merge、agent 行为全部经 issue/PR/comment 留痕、GitHub-native 优先。

## 六、Retro 模板

见 `loops/retro-TEMPLATE.md`（每轮复制为 `retro-NNN.md`，递增编号；含必填 UIPI 实测字段）。retro-001 / retro-002 为已完成实例。

## 七、v0 搭建清单（含授权分级，详见项目根 plan）

| # | 步骤 | 级别 |
|---|------|------|
| 1 | 克隆 fork、加 upstream、迁入本文档 | L1（已完成） |
| 2 | 安装 Claude Code GitHub App（`/install-github-app`），配 ANTHROPIC_API_KEY | L2（Mason 亲自，浏览器授权+计费） |
| 3 | 安装 Codex GitHub App，开启 Automatic reviews | L2（Mason 亲自，授权+计费） |
| 4 | AGENTS.md 加 Codex Review guidelines 段 | L1（已起草，push 为 L2） |
| 5 | push 配置分支 → PR → Mason merge（吃狗粮） | L2（push） |
| 6 | 首个真实 issue 全链路试跑 | L2（消耗双厂 API 费用） |
| 7 | 首轮 retro，记 UIPI 基线 | L1 |

## 变更记录

- 2026-07-08 v0.2：reviewer 由 claude-code-action 改为 Codex GitHub App；事件流与角色表重写；异构 reviewer 前移到 v0.1；补记本 repo 已 loop-ready 故省去 loop-init 脚手架。依据 [[recon-2026-07-08]]。
- 2026-07-08 v0.1：首版草案（reviewer 曾设为 claude-code-action automation mode）。
