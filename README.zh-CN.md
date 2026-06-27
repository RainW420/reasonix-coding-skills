# Reasonix Coding Skills

[English](README.md) | 中文

这个仓库保存一套可迁移的 Reasonix coding skill 体系，包括全局 agent 行为准则、核心编码工作流技能、中文项目约定、论文写作工作流技能，以及用于维护这套体系一致性的设计文档。

## 仓库内容

- `AGENTS.md` - Reasonix 全局行为准则和 coding 决策链。
- 核心 coding skills - 规划、执行、调试、审查、验证、worktree 和 agent 契约审计。
- `chinese-conventions/` - 中文项目约定入口，按需加载中文代码审查、提交规范、文档排版和国内 Git 平台参考。
- `paper-*` skills - 适配 Reasonix 的论文写作工作流。
- `docs/global-workspace/` - 设计说明、迁移说明和 skill 合并方案。

## 核心决策链

编码任务统一走一条链路：

1. 所有任务先经过 `using-superpowers` 路由。
2. 用 `task-modes` 判断任务模式。
3. 写代码前先走 task-modes 内的 ponytail 子模式。
4. 创造性或行为变更任务使用 `brainstorming` 和 `writing-plans`。
5. 执行计划时使用 `subagent-driven-development` 或 `executing-plans`。
6. 失败修复使用 `failure-repair`，未知根因调试使用 `systematic-debugging`。
7. 涉及 agent、runner、adapter、ACP/MCP/CLI、transcript、stdout/stderr、output、cancel 或 timeout 时，先使用 `agent-adapter-contracts`。
8. 审查使用 `flow-review`。
9. 完成声明前使用 `verification-before-completion`；跨文件事实对齐使用其中的 consistency mode。

## 在新 Linux 设备安装

克隆仓库：

```bash
git clone ssh://git@ssh.github.com:443/RainW420/reasonix-coding-skills.git
```

同步 skills 到 Reasonix：

```bash
mkdir -p ~/.reasonix/skills
rsync -a --delete \
  --exclude '.git' \
  --exclude 'README.md' \
  --exclude 'README.zh-CN.md' \
  --exclude 'docs/' \
  --exclude 'AGENTS.md' \
  reasonix-coding-skills/ ~/.reasonix/skills/
```

安装全局 Reasonix 指令文件：

```bash
mkdir -p ~/.config/reasonix
cp reasonix-coding-skills/AGENTS.md ~/.config/reasonix/AGENTS.md
```

可选：同步设计文档：

```bash
mkdir -p ~/.config/reasonix/global-workspace
cp reasonix-coding-skills/docs/global-workspace/*.md ~/.config/reasonix/global-workspace/
```

## 验证

检查 skill 入口文件和全局配置文件是否存在：

```bash
find ~/.reasonix/skills -maxdepth 2 -name SKILL.md | sort
test -f ~/.config/reasonix/AGENTS.md
```

检查合并前被删除的旧 skill 名是否还作为独立 skill 残留：

```bash
find ~/.reasonix/skills -name "*.md" ! -path "*/paper-*/*" -print0 \
  | xargs -0 grep -nE '(^name: (ponytail|requesting-code-review|receiving-code-review|build-fix|dispatching-parallel-agents|consistency-gate|chinese-code-review|chinese-commit-conventions|chinese-documentation|chinese-git-workflow)$|run_skill\(name="(ponytail|requesting-code-review|receiving-code-review|build-fix|dispatching-parallel-agents|consistency-gate|chinese-code-review|chinese-commit-conventions|chinese-documentation|chinese-git-workflow)")'
```

对 core coding skills 来说，这条命令应该没有输出。

## 不要提交这些内容

不要把本地运行状态或凭据提交到这个公开仓库：

- `~/.config/reasonix/credentials`
- `~/.config/reasonix/config.toml`
- `~/.config/reasonix/sessions/`
- `~/.config/reasonix/archive/`
- `~/.config/reasonix/global-workspace/.codegraph/`
- 本地缓存、日志、桌面窗口状态和 install ID

## 当前状态

核心 coding skill 已经从 20 个 core coding skill 加 4 个中文项目 skill，压缩为 14 个 core coding skill 加 1 个 `chinese-conventions` 路由 skill。

