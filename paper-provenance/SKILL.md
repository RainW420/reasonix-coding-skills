---
name: paper-provenance
description: 论文来源标注协议。强制区分 USER DATA、SOURCE CLAIM、INFERENCE、MATERIAL GAP、NEEDS CONFIRMATION，并在阈值超限时升级 checkpoint。
---

# Paper Provenance

## 标签

- `USER DATA`: 用户提供的数据、实验、事实。
- `SOURCE CLAIM`: 文献明确支持的声明。
- `INFERENCE`: 从材料推导出的解释。
- `MATERIAL GAP`: 缺少材料支撑，不能写成事实。
- `NEEDS CONFIRMATION`: 需要用户确认的事实。

## 使用

- `paper-write` 创建或修改 claim 时必须标注。
- `paper-citation` 补引用时必须说明 support level。
- `paper-verify` 检查标签完整性。
- 超过 checkpoint strategy 阈值时升级为强制确认。

参考 `/home/rainw/.reasonix/skills/paper-_references/integrity-checklist.md`。

