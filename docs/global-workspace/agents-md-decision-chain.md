## 决策链

所有编码任务走以下链路，每一步都有对应 skill 约束。完整版见 `.reasonix/specs/coding-decision-chain.md`。

```
入口路由
  using-superpowers — 任何行动前先检查 skill 适用性。优先级：用户指令 > skill 约束 > 系统默认。

任务模式
  task-modes — 识别模式（已有代码库/新项目/调试/重构/架构/脚本/Agent），加载对应约束。

最小化决策
  ponytail — 写代码前走 6 级：不写 > 复用 > 简化 > 标准库 > 已有依赖 > 才写。

创意→设计→计划
  brainstorming（8 步不可跳过）→ writing-plans（零占位符任务分解）→ 选择执行路径。

执行路径
  推荐：subagent-driven-development — 每任务一个全新子代理，两阶段审查（规格合规→代码质量）。
  备选：executing-plans — 当前会话逐任务执行，每 3 任务审查检查点。

失败修复
  test skill 或 failure-repair-template — 每次只修一个失败，重跑同一检查，同一失败最多 2 次。禁止删测试/禁用 lint/弱化类型。

完成验证
  verification-before-completion — 没有新鲜验证证据，不准声称完成。

代码审查
  flow-review — 只读、证据驱动、禁止幻觉。

契约审计
  agent-adapter-contracts — 涉及 runner/adapter/ACP/MCP/CLI/transcript/stdout/stderr/output/cancel/timeout 时，必须先建契约矩阵再改代码。

调试
  systematic-debugging — 四阶段：根因→模式→假设→修复。3 次修复失败 = 架构问题。
```

已安装的 coding skill 完整列表：using-superpowers, task-modes, ponytail, brainstorming, writing-plans, subagent-driven-development, executing-plans, test（内置）, test-driven-development, systematic-debugging, verification-before-completion, flow-review, security-review, agent-adapter-contracts, dispatching-parallel-agents, using-git-worktrees, finishing-a-development-branch, requesting-code-review, receiving-code-review, implement, explore, research, review。
