# CLI Interface Contract: Git Worktree Support

**Feature**: 001-git-worktrees
**Date**: 2026-01-15

## Overview

This document defines the CLI interface contracts for the git worktree feature. Since this is a shell script-based CLI tool (not a REST/GraphQL API), the contracts are defined as script invocation patterns and JSON output schemas.

---

## 1. create-new-feature Script

### Extended Interface

**Bash**: `.specify/scripts/bash/create-new-feature.sh`
**PowerShell**: `.specify/scripts/powershell/create-new-feature.ps1`

#### Usage

```bash
# Bash
./create-new-feature.sh [--json] [--short-name <name>] [--number N] <feature_description>

# PowerShell
./create-new-feature.ps1 [-Json] [-ShortName <name>] [-Number N] <feature_description>
```

#### Arguments (Unchanged)

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `--json` / `-Json` | flag | No | Output in JSON format |
| `--short-name` / `-ShortName` | string | No | Custom short name for branch |
| `--number` / `-Number` | integer | No | Manual branch number override |
| `<feature_description>` | string | Yes | Description of the feature |

#### JSON Output Schema (Extended)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["BRANCH_NAME", "SPEC_FILE", "FEATURE_NUM", "FEATURE_ROOT", "MODE"],
  "properties": {
    "BRANCH_NAME": {
      "type": "string",
      "pattern": "^[0-9]{3}-[a-z0-9-]+$",
      "description": "Feature branch name"
    },
    "SPEC_FILE": {
      "type": "string",
      "description": "Absolute path to spec.md"
    },
    "FEATURE_NUM": {
      "type": "string",
      "pattern": "^[0-9]{3}$",
      "description": "Zero-padded feature number"
    },
    "FEATURE_ROOT": {
      "type": "string",
      "description": "Absolute path to working directory (repo root in branch mode, worktree path in worktree mode)"
    },
    "MODE": {
      "type": "string",
      "enum": ["branch", "worktree"],
      "description": "Creation mode used"
    }
  }
}
```

#### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success (branch or worktree created) |
| 1 | Error (invalid arguments, git failure, etc.) |

#### Stderr Output

| Pattern | Meaning |
|---------|---------|
| `[specify] Warning: ...` | Non-fatal warning, operation continues |
| `Error: ...` | Fatal error, operation aborted |

---

## 2. configure-worktree Script (New)

### Interface

**Bash**: `.specify/scripts/bash/configure-worktree.sh`
**PowerShell**: `.specify/scripts/powershell/configure-worktree.ps1`

#### Usage

```bash
# Bash
./configure-worktree.sh [--mode <branch|worktree>] [--strategy <nested|sibling|custom>] [--path <custom-path>] [--show]

# PowerShell
./configure-worktree.ps1 [-Mode <branch|worktree>] [-Strategy <nested|sibling|custom>] [-Path <custom-path>] [-Show]
```

#### Arguments

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `--mode` / `-Mode` | enum | No | Set git mode (`branch` or `worktree`) |
| `--strategy` / `-Strategy` | enum | No | Set worktree strategy (`nested`, `sibling`, `custom`) |
| `--path` / `-Path` | string | Conditional | Custom base path (required if strategy is `custom`) |
| `--show` / `-Show` | flag | No | Display current configuration |

#### Behavior

- With no arguments: Shows help
- With `--show`: Displays current configuration
- With configuration flags: Updates `.specify/config.json`

#### Examples

```bash
# Enable worktree mode with nested strategy
./configure-worktree.sh --mode worktree --strategy nested

# Enable worktree mode with sibling strategy
./configure-worktree.sh --mode worktree --strategy sibling

# Enable worktree mode with custom path
./configure-worktree.sh --mode worktree --strategy custom --path /tmp/worktrees

# Disable worktree mode (back to branch mode)
./configure-worktree.sh --mode branch

# Show current configuration
./configure-worktree.sh --show
```

#### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error (invalid arguments, invalid path, etc.) |

---

## 3. Warning Messages Contract

### Standardized Warning Patterns

| Trigger | Message Pattern |
|---------|-----------------|
| Uncommitted changes | `[specify] Warning: Uncommitted changes in working directory will not appear in new worktree.` |
| Orphaned worktrees | `[specify] Warning: Orphaned worktree entries detected. Run 'git worktree prune' to clean up.` |
| Fallback to branch | `[specify] Warning: Worktree creation failed. Falling back to branch mode.` |
| Path not writable | `[specify] Warning: Cannot write to {path}. Falling back to branch mode.` |
| Branch name truncated | `[specify] Warning: Branch name exceeded GitHub's 244-byte limit. Truncated to: {name}` |

---

## 4. Configuration File Contract

### Location

`.specify/config.json`

### Schema

See [data-model.md](../data-model.md#1-configuration) for full schema definition.

### Example

```json
{
  "git_mode": "worktree",
  "worktree_strategy": "sibling"
}
```

---

## Backward Compatibility

1. **Default behavior unchanged**: Without configuration, scripts behave as before (branch mode)
2. **JSON output additive**: New fields (`FEATURE_ROOT`, `MODE`) added; existing fields unchanged
3. **No new required arguments**: All new CLI arguments are optional
4. **Graceful degradation**: Worktree failures fall back to branch mode automatically
