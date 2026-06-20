---
name: paper-task-modes
description: 识别论文任务模式：文献调研、论文写作、修改润色、审稿模拟、审稿回复、投稿准备、论文精读、科研绘图、PPT 制作。
---

# Paper Task Modes

## 模式表

| User intent | Mode | Next skill |
|-------------|------|------------|
| 想论文题目/研究方向 | ideation | paper-brainstorm |
| 写论文/写文章 | writing | paper-brainstorm or paper-plan |
| 已有大纲写正文 | drafting | paper-write |
| 查文献/综述 | research | paper-research |
| 润色/降 AI 味 | polishing | paper-polish |
| 查引用/改格式 | citation | paper-citation |
| 做图 | figure | paper-figure |
| 模拟审稿 | review | paper-review |
| 回复审稿 | response | paper-response |
| 投稿前准备 | finalize | paper-finalize |
| 精读论文 | read | paper-read |
| 论文转 slides | ppt | paper-ppt |

## 规则

- 写作、修改、投稿前先触发 `paper-ponytail`。
- 所有完成声明必须经过 `paper-verify` lite or full mode。
- 中文论文任务可启用 `article-polisher` 和 `chinese-documentation`，但仍遵守 Paper Passport。

