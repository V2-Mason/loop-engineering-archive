# Retro 002 — v0.2 Discipline 收尾 + reviewer 架构反思(2026-07-09)

> 状态:**已收口**。v0.2 全部合入 main;#20(codex-gate)、#22(return-loop)均已 merged,Path A 三个 workflow(codex-request / codex-fix-relay / codex-gate)现已在 main 上生效。

- **本轮目标**:给 v0.1 双厂闭环加纪律护栏,让单-issue 闭环更 hands-off。
- **结果**:全部完成并 merged——#A smoke CI、#C Codex 自动审、#E 触发治理、#F 分支保护 ruleset(strict + 线程 resolution)、#D 修复轮数机械封顶;#B 自动开 PR **取消**(避 PAT、留一键)。Path A:codex-gate(#20)+ return-loop(#22)均已 merged。端到端验收:issue #16 → PR #17 → Codex 审 clean → merged。

## UIPI 实测

v0.2 端到端验收(#16 → #17,**测于 Path A 上线前**)稳态每 issue 计划外人工介入 ≈ **2**:① 一键开 PR(#B 取消后为设计内的一键,仍计一次);② 手动 `@codex review`(Codex Automatic review 抽风未降下来)。

- v0.2 计划验收目标为 UIPI = 0,**未达标**——原因是下述可靠性缺口。v0.2 建成了纪律"管道",但没实现 UIPI 下降。
- **前瞻**:Path A 的 codex-request 现已 live,会在 open/synchronize 上自动 `@codex review` → 干次 issue 的"手动 @codex"这次介入应被消除。**下一轮(v0.2.5)首个真实 issue 需机械实测 UIPI,验证是否真降。**
- **UIPI 至今靠目测,未机械测量(见发现 6)** —— 这本身是 v0.2.5 要补的头号项。

## 本轮关键发现

1. **workflow-file 边界**:Claude GitHub App 不能修改 `.github/workflows/`(GitHub 安全硬限,需 `workflow` scope,App/GITHUB_TOKEN 无)。首次让 builder 改 CI workflow(#22)即撞墙。普通代码修复不受影响(#4/#17 已证)。
2. **bootstrap 悖论**:造 loop 自身管道的 PR(#20/#22)本身就是 workflow 文件 PR,永远用不了它正在造的自动化。codex-fix-relay 上线第一次运行就把 #22 自己的修复误派给 builder。
3. **软纪律不可靠**:`AGENTS.md` §Review guidelines 早已写"只报 P0/P1、3 轮即停",但托管 Codex 实际重提已修项、审过期 commit、无限挑刺——**写下来的纪律约束不住托管 reviewer**。
4. **Codex 不收敛**:同一 PR 每轮冒新发现(含已修的、过期的),ruleset 硬要求线程全 resolve → 每条噪音都成 merge 阻塞 → 必须人工兜底 resolve。这是把 UIPI 卡在 ~2 的主因。
5. **状态漂移**:session 开头 `MEMORY.md` 说"v0.1 草案",真相在 vault + 活的 GitHub(#20/#22 实时态)。手写状态 drift 于地面真相(GitHub)。
6. **UIPI/质量未机械测量**:北极星指标靠目测;merged 代码是否真好、Codex 是否被奖励作弊,均无抽检。

## 上游现成解 ↔ 我们的缺口

上游 `cobusgreyling/loop-engineering` 对"agent 不可靠"早有整套机械方案,我们大多没接:

| 缺口 | 上游现成结构 | 我们为何仍中招 |
|---|---|---|
| Codex 不收敛(4) | `loop-context`(断路器:iteration/token/stagnation/no-progress → escalate)+ `loop-guard` | 只在 builder 侧做了 CI label 版 #D,**未把断路器套到 review↔fix 大循环** |
| 评审纪律不可靠(3) | `loop-verifier`(default REJECT、亲自跑测试、结构化 verdict) | **托管 `@codex review` 连接器不加载 skill、不跑工具 → 纪律无处落地** |
| UIPI/质量未测(6) | `loop-cost` + `loop-ledger.json` + `loop-run-log.md` | 手册 L26-28/L58/L77 **已设计**这些,实现时未落地 |
| 状态漂移(5) | `loop-ledger` / `STATE.md`(结构化留痕) | 用手写 MEMORY.md 代替 |

关键:**手册本已设计了 loop-guard/ledger/run-log(P2-fixer/P4/P5),是实现未落地;唯一被"托管 Codex reviewer"设计选择堵死的是 P3(reviewer 跑不了 loop-verifier)。**

## 根因层级诊断(Vision / Roadmap / Milestone / Spec)

- **Vision(种子)**:"correctness 靠 reviewer intent-check"默认了 reviewer 可靠,缺一条"agent 不可靠 → 机械 enforce + 全程测量"的原则。
- **Spec(具体入口)**:v0/v0.1 选"reviewer = 托管 Codex 连接器",正是 loop-verifier 纪律失效的入口。
- **Roadmap(结构)**:只按流水线广度/规模排(intake→orchestration→release),缺一条正交的"可靠性/可测量"轴,P2-P6 无处安放。
- **Milestone/实现**:照 roadmap 执行无误;但 loop-guard/ledger/run-log 设计有、实现无。

## reviewer 运行时决策(已定,2026-07-09)

**决定:② Codex CLI 跑 loop-verifier。** 四要素:
- **所选**:reviewer 从托管 `@codex review` 连接器,换成 CI 内的 **Codex CLI agent job**,加载 loop-verifier(default REJECT、亲自跑测试、产结构化 verdict)。
- **备选**:① 保留托管 @codex review;③ Claude Code 跑 loop-verifier。
- **否决原因**:① 托管连接器不加载 skill、不跑工具,P3 根本无解;③ builder 与 reviewer 都是 Claude → 丢失跨厂盲点检查、动"双厂"根基。
- **回头条件**:Codex CLI 在 CI 的稳定性/成本不可接受,或 loop-verifier 实测纪律无效 → 退回 ① 保留托管 + 外部机械过滤;若跨厂价值证伪 → 再评估 ③。

P3 的根解是把 reviewer 从"托管连接器"换成"skill-running agent"。这**不动 Vision 根基**;是否动"双厂"取决于换成谁(② 保留双厂,故选它):

| 选项 | maker/checker | 跨厂 | 对根基 | 备注 |
|---|---|---|---|---|
| 托管 @codex review(现状) | 分离 | 是 | — | P3 无解,只能外部机械过滤 |
| **Codex CLI 跑 loop-verifier** | 分离 | **保留** | **不动** | 甜点:双厂+跨厂+纪律皆得;运行时升级 |
| Claude Code 跑 loop-verifier | 分离 | 丢失 | 动双厂 | 退回仅 maker/checker 隔离 |

换 reviewer 会**简化** Path A 管道(codex-request/gate/relay 大幅退化;PAT/bot-mention/过期 review/不收敛的坑消失),代价是重回 workflow 触发链(Path A 已付)+ 每 PR 起 agent job 更贵。手册需改段落:§一 L7、§二 L20/L39/L45/L54、角色表+事件流、§五 L87-88/L91/L94、§七 L3/L108、变更记录。

## 下一轮改进项(进入 roadmap)

**新增 v0.2.5「Reliability / Convergence」,排在 v0.3 多-issue 扩规模之前**(理由:可靠性是扩规模的前置,否则规模化 = 把不可靠性乘以 N):

1. 接 `loop-context` + `loop-ledger`:把断路器套到 review↔fix 大循环(P2)。
2. 接 `loop-cost` + `loop-run-log`:机械测量 UIPI(P5)。
3. reviewer 运行时决策落地(默认候选:Codex CLI 跑 loop-verifier)(P3)。
4. relay 加"仅 workflow 改动不派 builder"守卫(P1 边界固化)。
5. 状态从 GitHub 派生,手写笔记只留指针(P4)。

先让单-issue UIPI 真降到 ~1 且稳定,再上 v0.3。
