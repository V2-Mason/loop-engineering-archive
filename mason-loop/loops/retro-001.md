# Retro 001 — v0.1 双厂最小闭环收尾(2026-07-08)

- **本轮目标**：单 issue 走完 builder(Claude) → PR → reviewer(Codex) → 人 merge。
- **结果**：已完成。issue #2(给 loop-cost 加 `--version`)→ PR #4 merged,`--version` 已在 main;Codex 审后"无重大问题"。

## UIPI 实测

稳态(排除一次性搭建摩擦)每 issue 的计划外人工介入 = **2**,分别是:
1. builder 推分支后不自动开 PR,需人手动开(`gh pr create` / 点 Create PR 链接)。
2. Codex Automatic reviews 未开,需人手动评论 `@codex review` 触发审查。

两者均在"@claude 前置放行"与"最终 merge"两个 sanctioned 点之外,故计入 UIPI。目标 v0.2 降到 ~0。

## 一次性搭建摩擦(非稳态 UIPI,记录以便下次新 repo 一步到位)

1. fork 默认关闭 Issues → `gh repo edit <repo> --enable-issues`。
2. builder 首次 run 未产出:第一次误判为"allowedTools 缺 Edit/Write"(确实缺,已补),真实根因是 **OAuth token 401 Invalid bearer token**——`show_full_output` 关着时看不到,打开后才定位。教训:排错先开 `show_full_output`。
3. `CLAUDE_CODE_OAUTH_TOKEN` 一开始无效(疑似 `gh secret set` 误把值当第二参数 / 粘贴问题),重新 `claude setup-token` 并用交互提示重设后通过。

## 质量护栏

merge 未引发 revert / 补丁;CI(audit/validate)全绿;Codex 无意见。

## Codex 触发点观测

- Automatic reviews **未开**,需手动 `@codex review`。→ v0.2 #C 处理。
- 新 commit 复审：本轮 Codex 无意见、无修复轮,未验证复审触发。→ v0.2 #C/#D 一并验证。

## 下一轮改进项(进入 roadmap)

v0.2 Discipline：自动开 PR(#B)、Codex 自动审(#C)、smoke-test CI(#A)、修复轮数上限(#D)、触发治理(#E)。builder 还反馈 `npm test`/`node --test` 不在 allowedTools 无法自验 → 并入 #A。
