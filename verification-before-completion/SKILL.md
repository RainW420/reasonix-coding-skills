---
name: verification-before-completion
description: 在宣称工作完成、已修复或测试通过之前使用，在提交或创建 PR 之前——必须运行验证命令并确认输出后才能声称成功；始终用证据支撑断言
---

# 完成前验证

## 概述

在没有验证的情况下宣称工作完成，这不是高效，而是不诚实。

**核心原则：** 始终用证据支撑结论。

**对这条规则敷衍了事，就等于违背了它的精神。**

## 铁律

```
没有新鲜的验证证据，不许宣称完成
```

如果你在这条消息中没有运行验证命令，就不能声称测试通过。

## 门控函数

```
在宣称任何状态或表达满意之前：

1. 确定：什么命令能证明这个结论？
2. 运行：执行完整命令（重新运行，完整执行）
3. 阅读：完整输出，检查退出码，统计失败数
4. 验证：输出是否支持这个结论？
   - 如果否：用证据说明实际状态
   - 如果是：带证据陈述结论
5. 只有这时：才能做出结论

跳过任何一步 = 说谎，不是验证
```

## 验证 Profile

当用户没有指定验证范围时，按风险选择最小充分 profile；不要用 profile 名称替代实际证据。

| Profile | 何时使用 | 必跑内容 |
|---------|----------|----------|
| quick | 小改动、局部修复、需要快速确认 | 直接相关测试或检查 + git diff 范围 |
| full | 功能完成、重构完成、跨文件修改 | build/type/lint/test 中项目存在的检查 + git diff |
| pre-commit | 准备提交前 | full 的相关部分 + secrets/logs 审计 + git status |
| pre-pr | 准备 PR/合并前 | pre-commit + 变更摘要 + review 结果或残余风险 |

项目没有某类命令时，明确写“未发现/未运行原因”，不要伪造通过。

## 结构化验证报告

完成声明前，用下面格式报告证据。字段可以写 `N/A`，但不能删除。

```
VERIFICATION: PASS | FAIL | PARTIAL

Profile: quick | full | pre-commit | pre-pr
Build:   OK | FAIL | N/A  （命令 + 关键结果）
Types:   OK | FAIL | N/A  （命令 + 关键结果）
Lint:    OK | FAIL | N/A  （命令 + 关键结果）
Tests:   OK | FAIL | N/A  （命令 + 通过/失败数量）
Secrets: OK | FAIL | N/A  （搜索范围 + 结果）
Logs:    OK | FAIL | N/A  （console.log / 调试输出审计）
Git:     OK | WARN        （git status / diff 范围）

Ready: YES | NO | WITH RISKS
Evidence:
- （实际运行的命令和观察到的输出）

Remaining risks:
- （未运行项、已知失败、需要用户决策的风险）
```


## 一致性门（对齐/收口/漂移清理时必做）

当任务涉及配置、文档、状态摘要、报告、账号/ID、策略表述、handoff 或多文件事实对齐时，必须执行本技能的 consistency mode，并在完成报告中附上：

- `must-have` 清单及证据。
- `must-not-have` 清单及全局搜索结果。
- 搜索范围和命令原文。
- 当前状态摘要/表格是否检查过。
- 历史报告中的“修改前引用”和当前文档残留是否区分清楚。
- 每个通过结论对应的命令输出或文件证据。
- Claim Ledger 必须区分证据等级：`DIRECT`、`INDIRECT`、`MISSING`。
- sandbox、EROFS、权限、网络或工具不可用时，相关 claim 必须标为 `MISSING` 或 `INDIRECT`，不能推断为通过。

### Claim Ledger 证据等级

| 等级 | 含义 | 可否支持 PASS |
|------|------|---------------|
| `DIRECT` | 直接运行目标命令、读取目标文件或检查目标运行状态得到的证据 | 可以 |
| `INDIRECT` | 日志、旁路状态、历史报告、替代命令等弱证据 | 只能支持 WITH_RISKS / PARTIAL |
| `MISSING` | 命令未运行、被 sandbox/EROFS/权限拦截、或没有证据 | 不能支持通过 |

Claim Ledger 格式：

```markdown
| Claim | Evidence level | Evidence | Status |
|-------|----------------|----------|--------|
| 当前账号已更新 | DIRECT | `jq ...` 输出 `31...` | PASS |
| Gateway 可达 | DIRECT | `openclaw gateway status` probe ok | PASS |
| channel running | INDIRECT | journal 显示 running，但 `channels status` 被 EROFS 拦截 | WITH_RISKS |
| 真实发送已验证 | MISSING | 未经用户批准，未运行 `--send` | UNVERIFIED |
```

### Consistency Mode 完整流程

在声明 alignment、cleanup、migration、handoff、status-summary、policy、account/id、documentation-sync 或 drift-removal 完成前，按以下流程执行：

1. **重述目标状态**
   - 完成后应当为真的是哪些事实？
   - 哪些内容明确不在范围内？

2. **编辑前建立 invariant ledger**
   - `must-have`: 必须出现或为真的事实。
   - `must-not-have`: 旧事实、旧名称、旧 ID、过期策略或不得残留的短语。
   - `unknown`: 需要运行时验证或用户确认的事实。

3. **映射受影响表面**
   检查所有可能重复旧事实的位置：
   - config 和 scripts
   - README、guide、plan、docs
   - current-status 摘要和表格
   - reports、changelogs、handoff 文件
   - workspace、persona、memory、rules
   - tests、fixtures、examples、generated artifacts（相关时）

4. **窄范围编辑**
   只有在明确标为旧状态、失败证据或 before 示例时，才保留历史引用。

5. **运行一致性扫描**
   - 在映射表面内搜索每个 `must-not-have` 字符串或正则。
   - 结构化文件用结构化读取器检查，例如 JSON 用 `jq`。
   - 如果禁用值只出现在历史 before quote 中，必须记录例外。

6. **运行行为验证**
   执行能证明当前行为的命令。若命令因 sandbox、EROFS、权限、网络或工具缺失不能运行，记录原命令和错误，并把相关 claim 标为 `MISSING` 或 `INDIRECT`；不要推断通过。

7. **写 Claim Ledger**
   每条成功声明都绑定证据等级、证据和状态。

Consistency mode 报告形状：

```markdown
## Target State

## Invariant Ledger

### Must-have
- ...

### Must-not-have
- ...

### Unknown / Needs Approval
- ...

## Changed Files

## Consistency Scan
- Command:
- Result:
- Historical exceptions:

## Behavior Verification
- Command:
- Result:

## Claim Ledger

## Remaining Risks
```

### sandbox / 权限不可用处理

如果验证命令因 sandbox、EROFS、权限、网络或工具不可用失败：

1. 记录原命令、退出码和关键错误。
2. 判断该命令证明的是哪条 claim。
3. 能安全提权或换用等价只读命令时再试一次。
4. 仍不可运行时，该 claim 标为 `MISSING` 或 `INDIRECT`。
5. 不允许写“虽然跑不了但应该没问题”。

如果 `must-not-have` 仍出现在当前状态文档或配置中，不能报告 `DONE` 或“全部通过”。只能报告 `PARTIAL`、`DONE_WITH_CONCERNS` 或请求补修。

## 常见失败模式

| 结论 | 需要 | 不够格 |
|------|------|--------|
| 测试通过 | 测试命令输出：0 failures | 之前的运行结果、"应该会通过" |
| Linter 无报错 | Linter 输出：0 errors | 部分检查、推断 |
| 构建成功 | 构建命令：exit 0 | linter 通过、日志看起来没问题 |
| Bug 已修复 | 测试原始症状：通过 | 代码改了，假设已修复 |
| 回归测试有效 | 红-绿循环已验证 | 测试只通过了一次 |
| 代理已完成 | VCS diff 显示变更 | 代理报告"成功" |
| 需求已满足 | 逐项核对清单 | 测试通过 |

## 红线——停下来

- 使用"应该"、"大概"、"似乎"
- 验证前就表达满意（"太好了！"、"完美！"、"搞定！"等）
- 即将提交/推送/创建 PR 却没有验证
- 信任代理的成功报告
- 依赖部分验证
- 想着"就这一次"
- 累了想赶紧收工
- **任何暗示成功但实际未运行验证的措辞**

## 防止合理化

| 借口 | 现实 |
|------|------|
| "应该能行了" | 运行验证命令 |
| "我有信心" | 信心 ≠ 证据 |
| "就这一次" | 没有例外 |
| "Linter 通过了" | Linter ≠ 编译器 |
| "代理说成功了" | 独立验证 |
| "我累了" | 疲劳 ≠ 借口 |
| "部分检查就够了" | 部分检查什么也证明不了 |
| "搜过了应该没了" | 给出精确 `rg` 命令、范围、输出和例外；否则不算 |
| "sandbox 跑不了，推断没问题" | sandbox 失败是缺证，不是通过；标为 MISSING/INDIRECT |
| "换个说法这条规则就不适用了" | 精神大于字面 |

## 关键模式

**测试：**
```
✅ `run_command(command="go test ./...")` [看到：34/34 pass] "全部测试通过"
❌ "应该能通过了" / "看起来对了"
```

**回归测试（TDD 红-绿）：**
```
✅ 编写 → `run_command(command="go test ./...")`（通过）→ 回退修复 → `run_command(command="go test ./...")`（必须失败）→ 恢复 → `run_command(command="go test ./...")`（通过）
❌ "我写了回归测试"（没有经过红-绿验证）
```

**构建：**
```
✅ `run_command(command="go build ./...")` [看到：exit 0] "构建通过"
❌ "Linter 通过了"（linter 不检查编译）
```

**需求：**
```
✅ `read_file(path="docs/superpowers/plans/<plan-file>.md")` 重读计划 → 创建核对清单 → 逐项验证 → 报告缺口或完成
❌ "测试通过了，阶段完成"
```

**一致性/漂移清理：**
```
✅ 列出 must-have/must-not-have → `rg` 全局搜索禁用旧事实 → `jq`/结构化读取配置 → 每条 claim 绑定证据 → 再声明完成
❌ 只改了指定段落，就说“文档已统一”“零匹配”“全部通过”
```

**代理委派：**
```
✅ 代理报告成功 → `run_command(command="git diff")` 检查 VCS diff → 验证变更 → 报告实际状态
❌ 信任代理报告
```

**flag/side-effect 审计（修改现有代码时必做）：**
```
✅ 新增了副作用代码（文件写入/状态变更/网络调用/注入）→ 列出所有控制副作用的 flag（--dry-run/--no-state/--force/--send）→ 确认新代码被正确的 flag gate 住 → 测试 flag 开/关两种情况
❌ 只测了主路径，没验证 --no-state 等 flag 是否仍然有效隔离副作用
```

**Agent / Adapter / Protocol 验收门：**
```
✅ 修改 runner/adapter/workflow/bridge/controller → 先列出契约矩阵 → 验证真实 runner/protocol 路径 → 检查 output 文件、stdout/stderr、transcript、exit code、cancel/timeout、failure path → 再声明完成
❌ 只看到 npm test 通过，没证明 ACP/Codex/CLI 的真实协议和产物语义仍然成立
```

涉及 agent、workflow、runner、adapter、ACP/MCP/CLI、Codex exec、Reasonix ACP、transcript、stdout/stderr、outputFromStdout、cancel/timeout 时，必须额外确认：

- 指定 output 文件时没有被 stdout 覆盖；需要 stdout 输出时确实写入。
- transcript 保留 start/stdout/stderr/close/error 或协议事件，不退化为摘要。
- 协议 runner 仍执行真实握手，而不是被通用 subprocess wrapper 绕过。
- startup failure、early exit、non-zero exit、closed stdin/EPIPE、cancel、timeout 中与任务相关的路径至少被覆盖或明确标注未验证。
- fake agent/runner 与真实协议的差异已说明，不能把 shell echo 当作协议验收。

## 为什么这很重要

来自 24 次失败记录：
- 搭档说"我不信你"——信任被破坏
- 未定义的函数被交付——会直接崩溃
- 遗漏需求被交付——功能不完整
- 虚假完成浪费的时间 → 返工 → 重做
- 违反原则："诚实是核心价值。如果你说谎，就会被替换。"

## 何时使用

**以下情况之前必须使用：**
- 任何形式的成功/完成声明
- 任何满意的表达
- 任何关于工作状态的正面陈述
- 提交、创建 PR、标记任务完成
- 进入下一个任务
- 委派给代理

**本规则适用于：**
- 准确措辞
- 同义词和换一种说法
- 暗示成功
- 任何传达完成/正确性的沟通

## 底线

**验证没有捷径。**

运行命令。阅读输出。然后才能宣称结果。

这没有商量余地。
