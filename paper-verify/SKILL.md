---
name: paper-verify
description: 论文完成前验证门控。包含 lite mode（每章后机械检查）和 full mode（全稿验证），检查 claim、证据、数据、逻辑、格式、术语和 provenance。
---

# Paper Verify

## Lite Mode

每章后运行：

- 当前章 claims 是否均有 provenance 标签。
- evidence 是否真实绑定。
- 数字、图表、引用是否互相一致。
- section objective 是否完成。

Fail blocks next section.

## Full Mode

全稿后运行：

- 引用核查。
- 数据一致性。
- 逻辑链完整性。
- 术语一致性。
- 目标期刊格式。
- provenance 完整性。
- critical `known_gaps` 是否仍 open。

## 权限

- 可以更新 claim status。
- 可以更新 `known_gaps.status`。
- 不得改写 claim 文本或正文。

