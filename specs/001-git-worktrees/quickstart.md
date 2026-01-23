# Quickstart: Git Worktree Support

**Feature**: 001-git-worktrees
**Date**: 2026-01-15

## Overview

This guide explains how to enable and use git worktree support in Spec Kit for parallel feature development.

---

## Prerequisites

- Git 2.5 or later (verify with `git --version`)
- Spec Kit initialized in your repository (`.specify/` directory exists)

---

## Quick Setup

### 1. Enable Worktree Mode

```bash
# Enable with nested strategy (worktrees inside .worktrees/ directory)
.specify/scripts/bash/configure-worktree.sh --mode worktree --strategy nested

# Or enable with sibling strategy (worktrees as sibling directories)
.specify/scripts/bash/configure-worktree.sh --mode worktree --strategy sibling
```

### 2. Create a Feature

```bash
# Create a new feature (will use worktree mode if enabled)
.specify/scripts/bash/create-new-feature.sh --json "Add user authentication"
```

**Output (worktree mode)**:
```json
{
  "BRANCH_NAME": "002-user-authentication",
  "SPEC_FILE": "/path/to/.worktrees/002-user-authentication/specs/002-user-authentication/spec.md",
  "FEATURE_NUM": "002",
  "FEATURE_ROOT": "/path/to/.worktrees/002-user-authentication",
  "MODE": "worktree"
}
```

### 3. Navigate to Worktree

```bash
# The FEATURE_ROOT in the JSON output is your working directory
cd /path/to/.worktrees/002-user-authentication
```

---

## Strategies Explained

### Nested (Default for Worktree Mode)

Worktrees are created inside your project's `.worktrees/` directory.

```
my-project/
├── .worktrees/
│   ├── 001-feature-a/    # Worktree for feature A
│   └── 002-feature-b/    # Worktree for feature B
├── src/
└── specs/
```

**Best for**: Sandboxed environments, keeping everything contained.

### Sibling

Worktrees are created as sibling directories to your project.

```
parent-directory/
├── my-project/           # Main repository
├── 001-feature-a/        # Worktree for feature A
└── 002-feature-b/        # Worktree for feature B
```

**Best for**: Clean separation, IDE project switching.

### Custom

Worktrees are created in a specified directory.

```bash
.specify/scripts/bash/configure-worktree.sh --mode worktree --strategy custom --path /tmp/worktrees
```

**Best for**: Temporary work, specific storage requirements.

---

## Common Tasks

### View Current Configuration

```bash
.specify/scripts/bash/configure-worktree.sh --show
```

### Switch Back to Branch Mode

```bash
.specify/scripts/bash/configure-worktree.sh --mode branch
```

### List Active Worktrees

```bash
git worktree list
```

### Remove a Worktree

```bash
# When you're done with a feature
git worktree remove /path/to/worktree
```

---

## Working with AI Agents

When using AI agents (Claude, Copilot, etc.) with worktree mode:

1. **Check the JSON output**: The `FEATURE_ROOT` field tells you (and the AI) where to work
2. **Use absolute paths**: AI agents should use `FEATURE_ROOT` as the base for all file operations
3. **MODE indicator**: The `MODE` field explicitly tells the AI whether worktree mode is active

### Example AI Agent Instruction

After creating a feature with worktree mode, tell your AI agent:

> "I've created a new feature in worktree mode. The working directory is at `FEATURE_ROOT`. Please change to that directory for all subsequent commands."

---

## Troubleshooting

### Worktree Creation Failed

If you see `[specify] Warning: Worktree creation failed. Falling back to branch mode.`:

1. Check that the target path is writable
2. Verify Git version: `git --version` (must be 2.5+)
3. Check for path length issues on Windows

### Orphaned Worktree Warning

If you see `[specify] Warning: Orphaned worktree entries detected.`:

```bash
# Clean up orphaned entries
git worktree prune
```

### Uncommitted Changes Warning

If you see warning about uncommitted changes:

- This is expected behavior
- Uncommitted changes stay in your current working directory
- The new worktree starts clean from the branch point

---

## Integration with Spec Kit Workflow

```bash
# 1. Enable worktree mode (one-time setup)
.specify/scripts/bash/configure-worktree.sh --mode worktree --strategy sibling

# 2. Create a feature (creates worktree)
.specify/scripts/bash/create-new-feature.sh --json "My new feature"

# 3. Navigate to worktree
cd <FEATURE_ROOT from JSON output>

# 4. Continue with normal Spec Kit workflow
# /speckit.specify, /speckit.plan, /speckit.tasks, etc.

# 5. When done, clean up worktree
git worktree remove $(pwd)
cd <back to main repo>
```
