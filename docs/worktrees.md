# Worktrees Guide

Spec Kit can manage feature development using either branches (default) or git worktrees (parallel working directories). Worktrees are useful when you want multiple features open at once without constantly switching branches.

## Quick Start

Enable worktree mode during init:

```bash
uvx --from git+https://github.com/brianluby/spec-kit.git specify init my-project --git-mode worktree --worktree-strategy sibling
```

This sets defaults in `.specify/config.json`. You can change them later with:

```bash
.specify/scripts/bash/configure-worktree.sh --mode worktree --strategy sibling
```

On PowerShell:

```powershell
.specify/scripts/powershell/configure-worktree.ps1 -Mode worktree -Strategy sibling
```

## Worktree Placement Strategies

- `sibling`: create worktrees next to the repo (default)
- `nested`: create worktrees under `.worktrees/` inside the repo
- `custom`: use an absolute base path (requires `--worktree-path`)

Examples:

```bash
specify init my-project --git-mode worktree --worktree-strategy nested
specify init my-project --git-mode worktree --worktree-strategy custom --worktree-path /tmp/worktrees
```

## How Feature Creation Works

When you run `/speckit.specify`, Spec Kit:

1. Creates a new feature branch (e.g., `001-create-taskify`).
2. Creates a worktree directory for that branch (if worktree mode is enabled).
3. Writes the spec into `specs/<feature-branch>/` inside that worktree.

Important: if you use worktrees, you must switch your IDE/agent to the new worktree directory before running `/speckit.plan`, `/speckit.tasks`, or `/speckit.implement`.

## Output Context

The feature creation scripts output additional context that agents can use:

- `FEATURE_ROOT`: the directory where the feature lives (worktree root if enabled)
- `MODE`: `branch` or `worktree`

## Troubleshooting

- If you see orphaned worktree warnings, clean them up with:

```bash
git worktree prune
```

- If worktree creation fails due to path permissions, switch strategy:

```bash
.specify/scripts/bash/configure-worktree.sh --strategy nested
```

Or fall back to branch mode:

```bash
.specify/scripts/bash/configure-worktree.sh --mode branch
```
