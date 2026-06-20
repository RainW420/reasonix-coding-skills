---
name: paper-write
description: 论文主写作引擎。主 session 握笔，逐章生成正文，绑定 evidence 到 claim，调用工种，运行 verify lite，并管理 revision mode。
---

# Paper Write

## 必读

- `paper-_constitution/passport-schema.md`
- `paper-_constitution/skill-boundaries.md`
- `paper-_constitution/checkpoint-strategy.md`

## 逐章循环

1. 加载 Passport、blueprint、前文、相关候选证据和图表契约。
2. 主 session 生成章节草稿。
3. 绑定 `evidence_candidates` 到 `evidence_map`，只在证据真正支撑 claim 时绑定。
4. 并行调用工种：`paper-citation`、`paper-polish`、`paper-figure`。
5. 调用 `paper-verify` lite mode。
6. 失败进入 `paper-debug`；通过则默认确认。
7. 更新 `section_status` 和 `revision_history`。

## Revision Mode

消费 `paper-response` 的 `revision_plan` 和 `proposed_patch`。主 session 决定是否合并正文。

## 禁止

- 不得让工种掌握最终论证权。
- 不得跳过 verify lite。

