---
name: paper-review
description: 全稿模拟审稿门控。运行多视角语义审查，只写 review_findings，不直接修改正文；critical finding 阻塞 paper-finalize。
---

# Paper Review

## 使用时机

- 全稿完成后。
- 投稿前。
- 大修后复查。

## 输出

- structured `review_findings`
- severity
- dimension scores
- recommended revision priorities

## 阻塞语义

- 不阻塞 `paper-response` 和 revision 阶段。
- 若存在 critical finding，阻塞 `paper-finalize`。
- 是否继续投稿必须强制确认。

参考 `/home/rainw/.reasonix/skills/paper-_adapters/review-adapter.md`。

