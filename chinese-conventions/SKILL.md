---
name: chinese-conventions
description: 检测到中文项目、中文团队沟通、中文代码审查、国内 Git 平台、中文技术文档或中文提交信息时使用；按场景加载中文审查、提交、文档和 Git 工作流规范。
---

# 中文项目约定路由

这是中文项目相关约定的统一入口。先判断具体场景，再读取对应 reference 文件；不要一次性加载所有长规范。

## 触发场景

| 场景 | 使用模式 | 参考文件 |
|------|----------|----------|
| 代码审查且团队使用中文沟通 | code-review mode | `references/chinese-code-review.md` |
| 使用 Gitee、Coding.net、极狐 GitLab、CNB 或国内镜像同步 | git-workflow mode | `references/chinese-git-workflow.md` |
| 编写中文技术文档、README、说明文案 | documentation mode | `references/chinese-documentation.md` |
| 编写中文项目 commit message、changelog 或提交规范 | commit-conventions mode | `references/chinese-commit-conventions.md` |

## 使用规则

1. 先识别当前任务属于哪个模式。
2. 只读取对应 reference 文件。
3. 如果任务同时涉及多个模式，按实际需要逐个加载。
4. 用户用中文沟通时，输出使用中文；代码、命令、标识符按项目既有风格保留。

## 不要做

- 不要因为项目含少量中文就强行套用所有中文规范。
- 不要把中文表达优化和代码正确性混为一谈。
- 不要覆盖项目已有提交规范、文档模板或平台约定；先遵循本项目现状。
