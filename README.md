# Reasonix Coding Skills

English | [中文](README.zh-CN.md)

This repository contains a portable Reasonix coding-skill system: global agent instructions, core coding workflow skills, Chinese-project conventions, paper-writing workflow skills, and design notes used to keep the system coherent across Linux devices.

## What Is Included

- `AGENTS.md` - global Reasonix behavior and coding decision-chain instructions.
- Core coding skills - planning, execution, debugging, review, verification, worktree use, and agent-contract checks.
- `chinese-conventions/` - a routing skill for Chinese code review, commit conventions, documentation, and domestic Git workflows, with detailed references loaded on demand.
- `paper-*` skills - a paper-writing workflow system adapted for Reasonix.
- `docs/global-workspace/` - design notes, migration notes, and merge plans used to build this setup.

## Core Decision Chain

The coding workflow is organized as a single decision chain:

1. Route every task through `using-superpowers`.
2. Classify the task with `task-modes`.
3. Apply the ponytail sub-mode before writing code.
4. Use `brainstorming` and `writing-plans` for creative or behavior-changing work.
5. Execute plans through `subagent-driven-development` or `executing-plans`.
6. Repair failures through `failure-repair` and debug unknown causes with `systematic-debugging`.
7. Use `agent-adapter-contracts` before touching agent, runner, adapter, ACP/MCP/CLI, transcript, stdout/stderr, output, cancel, or timeout behavior.
8. Review with `flow-review`.
9. Verify completion with `verification-before-completion`, including consistency mode for cross-file alignment work.

## Install On A New Linux Device

Clone the repository:

```bash
git clone ssh://git@ssh.github.com:443/RainW420/reasonix-coding-skills.git
```

Copy or sync the skills into Reasonix:

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

Install the global Reasonix instruction file:

```bash
mkdir -p ~/.config/reasonix
cp reasonix-coding-skills/AGENTS.md ~/.config/reasonix/AGENTS.md
```

Optional: keep the design notes available locally:

```bash
mkdir -p ~/.config/reasonix/global-workspace
cp reasonix-coding-skills/docs/global-workspace/*.md ~/.config/reasonix/global-workspace/
```

## Verify

Check that the expected skill entry files exist:

```bash
find ~/.reasonix/skills -maxdepth 2 -name SKILL.md | sort
test -f ~/.config/reasonix/AGENTS.md
```

Check that removed pre-merge skill names are not present as standalone skill names:

```bash
find ~/.reasonix/skills -name "*.md" ! -path "*/paper-*/*" -print0 \
  | xargs -0 grep -nE '(^name: (ponytail|requesting-code-review|receiving-code-review|build-fix|dispatching-parallel-agents|consistency-gate|chinese-code-review|chinese-commit-conventions|chinese-documentation|chinese-git-workflow)$|run_skill\(name="(ponytail|requesting-code-review|receiving-code-review|build-fix|dispatching-parallel-agents|consistency-gate|chinese-code-review|chinese-commit-conventions|chinese-documentation|chinese-git-workflow)")'
```

The command should print no matches for the core coding skills.

## Do Not Commit

Do not add local runtime state or credentials to this repository:

- `~/.config/reasonix/credentials`
- `~/.config/reasonix/config.toml`
- `~/.config/reasonix/sessions/`
- `~/.config/reasonix/archive/`
- `~/.config/reasonix/global-workspace/.codegraph/`
- local caches, logs, desktop window state, or install IDs

## Status

The core coding skills have been compressed from 20 core coding skills plus 4 Chinese-project skills into 14 core coding skills plus one `chinese-conventions` routing skill.

