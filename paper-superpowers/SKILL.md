---
name: paper-superpowers
description: 论文、研究、学术写作、投稿、审稿回复、文献调研、科研绘图、论文精读或论文转 PPT 任务的总路由。任何 paper-* 工作前先用。
---

# Paper Superpowers

在任何论文相关行动前使用本技能。它是 paper skill 系统入口。

## 必读

- `/home/rainw/.reasonix/skills/paper-_constitution/skill-boundaries.md`
- `/home/rainw/.reasonix/skills/paper-_constitution/checkpoint-strategy.md`

## 路由

1. 判断任务是否属于论文/研究/学术写作。
2. 调用 `paper-task-modes` 识别主导模式。
3. 对写作、规划、修改、投稿类任务前置 `paper-ponytail`。
4. 若已有 Paper Passport，检查状态和下一合法阶段。
5. 若没有 Passport，要求 `paper-brainstorm` 或 `paper-plan` 初始化。

## 优先级

1. 用户明确指令
2. Paper constitution
3. 主线 skill
4. 门控 skill
5. 工种 skill
6. 适配器和参考层

## 红线

- 不允许外部基底 skill 绕过本路由。
- 不允许未验证就宣称完成。
- 不允许把 `MATERIAL GAP` 写成事实。
- 不允许专业工种改写主线论证。

