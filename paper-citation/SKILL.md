---
name: paper-citation
description: 引文工种。用于声明支持文献检索、引用格式统一、参考文献导出和正文引用核查，追加 citation_registry。
---

# Paper Citation

## 工作

- 从正文提取需支持声明。
- 检索或匹配候选文献。
- 标注 support level: direct, partial, background。
- 追加 `citation_registry`。
- 返回未支持 claims。

## 禁止

- 不编造 DOI、作者、标题或年份。
- 不把 background citation 标成 direct。
- 不直接绑定 `evidence_map`。

参考 `/home/rainw/.reasonix/skills/paper-_adapters/citation-adapter.md`。

