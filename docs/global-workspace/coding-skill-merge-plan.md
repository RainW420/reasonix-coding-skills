# Coding Skill 合并压缩方案（修订版）

## 背景

Reasonix coding 决策链已完善——13 步从入口到完成门全覆盖，新增了 Consistency Gate、Claim Ledger、failure-repair 模板。skill 数量偏多：core coding 20 个 + chinese 4 个 = **24 个文件**（不含子代理、元技能和领域技能）。目标：在不破坏决策链完整性和 skill 内容的前提下压缩合并，作为可迁移的通用配置上传到 GitHub。

## 当前实际状态（合并前的精确计数）

```
Core coding 文件（ls 确认的目录）：
  agent-adapter-contracts brainstorming build-fix consistency-gate
  dispatching-parallel-agents executing-plans failure-repair
  finishing-a-development-branch flow-review ponytail
  receiving-code-review requesting-code-review
  subagent-driven-development systematic-debugging task-modes
  test-driven-development using-git-worktrees using-superpowers
  verification-before-completion writing-plans
  共 20 个 core coding 文件

Chinese 文件：4 个（chinese-code-review / chinese-commit-conventions /
  chinese-documentation / chinese-git-workflow）

纸面 skill（skill index 列出但无独立文件）：
  security-review — 已完全集成在 flow-review 的安全 mode 中，无独立文件
  test — Reasonix 内置 skill，不在文件系统中
  implement — 有独立文件，保留
  explore / research / review — 无独立文件，为 Reasonix 内置或顶层工具映射

元技能文件：
  writing-skills — 有独立文件
  init / install-capability — 无独立文件，为 Reasonix 内置命令
```

## 合并判断原则

1. **触发点相同** — 两个 skill 在决策链的同一位置被调用
2. **语义不互斥** — 合并后不产生模式切换污染
3. **不存在硬行数上限** — 目标文件以可读性为准，超长内容拆入 `references/` 子文件 lazy-load

## 合并方案（7 组，24→15）

| # | 合并 | 理由 |
|---|------|------|
| 1 | `ponytail` → `task-modes` | 同一触发点，task-modes 已前置触发 ponytail。6 级决策阶梯作为子模式 |
| 2 | `requesting-code-review` + `receiving-code-review` → `flow-review` | 同一生命周期，合并为 flow-review 的 requesting/receiving 模式 |
| 3 | `failure-repair` + `build-fix` → `failure-repair` | build-fix 是 failure-repair 的实例，合并为模板 + build/type/lint mode |
| 4 | `security-review` → `flow-review` | **现状确认：无独立 security-review 文件。** 安全内容已在 flow-review 安全 mode 中，本次只需删除 skill index 中的独立条目引用，不涉及文件操作 |
| 5 | `dispatching-parallel-agents` → `subagent-driven-development` | 并行派发是子代理执行的策略，作为子模式 |
| 6 | 4 个 chinese skill → `chinese-conventions` | 同一触发域（中文项目），四个独立子节。总内容超 1000 行，拆为 chinese-conventions（路由） + chinese-conventions-references/（详细规范 lazy-load） |
| 7 | `consistency-gate` → `verification-before-completion` | **关键缺口：当前 consistency-gate 为独立文件。** 一致性门作为 verification 的 consistency mode 并入，删除原文件 |

## 不合并（5 组，明确排除）

| # | 不合并 | 理由 |
|---|--------|------|
| 1 | `subagent-driven-development` + `executing-plans` | 执行模型完全不同（spawn 新子代理 vs 内联逐任务），合并后 mode-switching 逻辑污染两者 |
| 2 | `using-git-worktrees` + `finishing-a-development-branch` | 调用时机完全不同（链开头 vs 链末尾） |
| 3 | `test-driven-development` + `systematic-debugging` | TDD 管「怎么写」，调试管「怎么修」，触发条件互不重叠 |
| 4 | `brainstorming` + `writing-plans` | 前者 8 步创意流程，后者零占位符任务分解，已足够长 |
| 5 | `agent-adapter-contracts` | 专用契约矩阵，无同类可合 |

## 最终清单（15 个决策链文件）

```
Core coding（14 个）：
  using-superpowers
  task-modes                  ← 合并 ponytail
  brainstorming
  writing-plans
  subagent-driven-development ← 合并 dispatching-parallel-agents
  executing-plans
  test-driven-development
  systematic-debugging
  failure-repair              ← 合并 build-fix
  verification-before-completion ← 合并 consistency-gate
  flow-review                 ← 合并 requesting-code-review + receiving-code-review
  agent-adapter-contracts
  using-git-worktrees
  finishing-a-development-branch

Chinese（1 个）：
  chinese-conventions              ← 合并 4 个 chinese skill
  chinese-conventions-references/  ← 长内容 lazy-load（可选）

不变（非决策链节点，不计入 15 个）：
  implement — 子代理，有独立文件
  writing-skills — 元技能，有独立文件
  article-polisher / due-framework / long-novel-writer — 领域技能
  test — Reasonix 内置，不在文件系统中

已确认不存在的文件（不计入计数）：
  security-review / explore / research / review — 无独立文件
  init / install-capability — 无独立文件
```

## 删除文件清单（执行时移除）

```
ponytail/SKILL.md
requesting-code-review/SKILL.md
receiving-code-review/SKILL.md
build-fix/SKILL.md
dispatching-parallel-agents/SKILL.md
consistency-gate/SKILL.md
chinese-code-review/SKILL.md
chinese-commit-conventions/SKILL.md
chinese-documentation/SKILL.md
chinese-git-workflow/SKILL.md
```
以上目录一并删除。

## 旧引用重写（执行时必须完成）

合并后以下文件中的引用必须更新，否则上传 GitHub 后有断链：

| 文件 | 旧引用 | 新写法 |
|------|--------|--------|
| `subagent-driven-development/SKILL.md` | 调用 `requesting-code-review` | → 调用 `flow-review` requesting mode |
| `using-superpowers/SKILL.md` | 路由到 `chinese-code-review` 等 | → 路由到 `chinese-conventions` |
| `verification-before-completion/SKILL.md` | 引用 `consistency-gate` | → 改为「consistency mode」 |
| `task-modes/SKILL.md` | 触发 `ponytail` | → 改为「ponytail 子模式」 |
| `flow-review/SKILL.md` | 引用 `security-review` | → 改为「security mode」（注：文件本就不存在，仅文本引用修正） |
| `implement/SKILL.md` | 引用 `consistency-gate` | → 改为「按 verification-before-completion 的 consistency mode 执行」 |

## 执行步骤

```
1. 新建 git worktree 或分支（当前在 master 且无 remote）
2. 逐组执行合并（先改目标文件，再运行 grep 确认旧引用全更新，最后删原文件）
3. 合并完成后运行断链检查（使用精确匹配，避免误伤 paper 系列 skill）：
   # 第一层：检查是否还有以被删 skill 名作为独立 skill 的引用
   find ~/.reasonix/skills -name "*.md" ! -path "*/paper-*/*" -print0 \
     | xargs -0 grep -nE '(`(ponytail|requesting-code-review|receiving-code-review|build-fix|dispatching-parallel-agents|consistency-gate|chinese-code-review|chinese-commit-conventions|chinese-documentation|chinese-git-workflow)`|run_skill\(name="(ponytail|requesting-code-review|receiving-code-review|build-fix|dispatching-parallel-agents|consistency-gate|chinese-code-review|chinese-commit-conventions|chinese-documentation|chinese-git-workflow)"|^name: (ponytail|requesting-code-review|receiving-code-review|build-fix|dispatching-parallel-agents|consistency-gate|chinese-code-review|chinese-commit-conventions|chinese-documentation|chinese-git-workflow)$)' \
     | grep -v "合并\|历史\|曾用名"
   上述命令结果必须为空。
   # 第二层：paper 域交叉引用（若 paper-task-modes 等引用被删的中文 skill，后续另行处理）
   find ~/.reasonix/skills/paper-* -name "*.md" -print0 \
     | xargs -0 grep -n "chinese-documentation\|chinese-code-review\|chinese-commit-conventions\|chinese-git-workflow" \
     | grep -v "合并\|历史\|曾用名"
   如有结果，将引用改为 `chinese-conventions` 对应子节。此项不阻塞本次合并，可在合并 commit 后单独处理。
4. git add -A && git commit
```
