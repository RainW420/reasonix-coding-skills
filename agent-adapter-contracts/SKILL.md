---
name: agent-adapter-contracts
description: 当任务涉及 agent、workflow、runner、adapter、ACP/MCP/CLI、Codex/Reasonix bridge、transcript、stdout/stderr、output 文件、cancel/timeout 或进程生命周期时使用；在修改代码前识别并保护真实协议契约。
---

# Agent / Adapter 契约保护

你正在修改的不是普通函数，而是多个进程、协议、工具和文件产物之间的边界。这里的错误通常不会被普通单元测试发现，但会在真实 agent workflow 中造成上下文丢失、输出覆盖、取消失效或进程崩溃。

**核心原则：生命周期控制器不能替代协议 runner。**

Controller 可以管理 spawn、cancel、timeout 和进程关闭；runner 必须继续负责协议握手、stdin/stdout/stderr 语义、transcript、output 文件、权限和错误传播。除非契约测试证明完全等价，不要用通用进程封装替换专用 runner。

## 触发条件

任务涉及以下任一内容时必须使用本技能：

- agent workflow、multi-agent、bridge、handoff、checkpoint、archive
- runner、adapter、controller、进程生命周期、spawn、detached child
- ACP、MCP、JSON-RPC、CLI 协议、Codex exec、Reasonix ACP、OMC
- transcript、stdout、stderr、outputFromStdout、`-o` 输出文件
- cancel、timeout、early exit、EPIPE、closed stdin、non-zero exit
- 权限边界、审批节点、工具调用、任务恢复

## 修改前契约清单

改代码前，先写出受影响边界的契约矩阵：

| 边界 | 必须确认的问题 |
|------|----------------|
| 输入 | prompt 从哪里进入？stdin 是文本、JSON-RPC、参数还是文件？ |
| 输出 | 最终结果来自 stdout、指定 output 文件、事件流还是 transcript？ |
| stdout/stderr | stdout 是用户可见结果、日志、事件，还是协议帧？stderr 是否进入 transcript？ |
| transcript | 需要记录哪些事件：start、stdin、stdout、stderr、protocol message、close、error？ |
| exit code | 0、非 0、signal、early exit 分别如何传播？ |
| cancel/timeout | 谁负责杀进程？下游节点如何标记？是否能命中 idle/approval 期？ |
| 文件产物 | 哪些文件不能被覆盖？哪些文件必须落盘？ |
| 权限/审批 | 谁可以调用高权限工具？是否必须进入 approval gate？ |
| 副作用 | 会写哪些目录、改哪些文件、触发哪些外部动作？ |

如果无法填完矩阵，状态应为 `NEEDS_CONTEXT`，不要先动手。

## 实现规则

- 复用现有 runner/adapter，优先把 controller 作为参数透传，而不是绕过 runner。
- 保留现有 output 语义。例如 Codex 使用 `-o` 时，stdout 不得覆盖 output 文件。
- 保留 transcript 完整性。不要把 transcript 退化成只有 close/error 的摘要。
- 协议 runner 必须继续执行真实协议握手。例如 ACP 必须保留 initialize、session/new、session/prompt、session/update 等 JSON-RPC 流程。
- cancel/timeout 只能改变生命周期，不应改变协议、输出和 transcript 语义。
- 对 stdin 写入必须处理 closed stdin、EPIPE 和 early exit，不能让主进程崩溃或挂起。
- 不要用“看起来能跑”的 shell fake 证明协议正确；fake 必须逼近真实协议的关键行为。

## 测试要求

至少覆盖：

- happy path：协议握手、输出文件、stdout/stderr、transcript 都符合旧契约。
- output 契约：指定 output 文件时不被 stdout 覆盖；需要 stdout 输出时确实写入。
- transcript 契约：start/stdout/stderr/close/error 或协议事件按预期落盘。
- failure path：startup failure、early exit、non-zero exit、closed stdin/EPIPE 中与任务相关的项。
- lifecycle path：cancel 和 timeout 命中真实 child，并且 pending/downstream 状态正确。
- fake 可信度：说明 fake 覆盖了真实协议的哪些关键语义，遗漏了什么。

对于 AgentMesh、ACP、Codex bridge、OMC bridge 这类任务，`npm test` 通过只是基础验收；还必须有至少一个测试或手动验证证明真实运行路径没有被通用封装绕过。

## 自审报告

完成时必须报告：

```
契约保留：
- 保留了哪些旧行为和边界语义

行为改变：
- 哪些行为被有意改变，为什么

真实路径验证：
- 跑了哪些测试/命令证明真实 runner/protocol 路径有效
- fake 与真实协议的差异

异常路径：
- startup failure / early exit / non-zero / cancel / timeout / EPIPE 覆盖情况

残余风险：
- 未验证内容和下一步建议
```

不能清楚说明这些内容时，不要报告 `DONE`，使用 `DONE_WITH_CONCERNS` 或 `NEEDS_CONTEXT`。
