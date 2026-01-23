# Data Model: Git Worktree Support

**Feature**: 001-git-worktrees
**Date**: 2026-01-15

## Overview

This feature introduces a single persistent entity: the worktree configuration. All other data (worktrees themselves, branches) are managed by Git natively.

---

## Entities

### 1. Configuration

**Location**: `.specify/config.json`
**Lifecycle**: Created on first configuration, persists across sessions

#### Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "git_mode": {
      "type": "string",
      "enum": ["branch", "worktree"],
      "default": "branch",
      "description": "Feature creation mode"
    },
    "worktree_strategy": {
      "type": "string",
      "enum": ["nested", "sibling", "custom"],
      "default": "nested",
      "description": "Worktree placement strategy"
    },
    "worktree_custom_path": {
      "type": "string",
      "default": "",
      "description": "Custom base path for worktrees (required when strategy is 'custom')"
    }
  },
  "additionalProperties": true
}
```

#### Field Details

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `git_mode` | enum | No | `"branch"` | Controls whether features are created as branches or worktrees |
| `worktree_strategy` | enum | No | `"nested"` | Where worktrees are placed relative to the repository |
| `worktree_custom_path` | string | Conditional | `""` | Required when `worktree_strategy` is `"custom"` |

#### Validation Rules

1. `git_mode` must be one of: `"branch"`, `"worktree"`
2. `worktree_strategy` must be one of: `"nested"`, `"sibling"`, `"custom"`
3. If `worktree_strategy` is `"custom"`, then `worktree_custom_path` must be non-empty
4. If `worktree_custom_path` is provided, it must be an absolute path
5. File must be valid JSON

#### State Transitions

```
[No Config] --configure--> [Config Exists]
                               |
                               v
                    [git_mode: branch] <--switch--> [git_mode: worktree]
```

---

### 2. Script Output (Transient)

**Location**: stdout (JSON mode)
**Lifecycle**: Transient, generated per script invocation

#### Current Schema (Existing)

```json
{
  "BRANCH_NAME": "string",
  "SPEC_FILE": "string (absolute path)",
  "FEATURE_NUM": "string (3-digit padded)"
}
```

#### Extended Schema (This Feature)

```json
{
  "BRANCH_NAME": "string",
  "SPEC_FILE": "string (absolute path)",
  "FEATURE_NUM": "string (3-digit padded)",
  "FEATURE_ROOT": "string (absolute path)",
  "MODE": "string (branch|worktree)"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `BRANCH_NAME` | string | Feature branch name (e.g., `"001-git-worktrees"`) |
| `SPEC_FILE` | string | Absolute path to spec.md |
| `FEATURE_NUM` | string | Zero-padded feature number |
| `FEATURE_ROOT` | string | **NEW**: Absolute path to working directory |
| `MODE` | string | **NEW**: `"branch"` or `"worktree"` |

#### Backward Compatibility

- Existing fields remain unchanged
- New fields are additive
- Consumers ignoring unknown fields will continue to work

---

## Relationships

```
┌─────────────────────┐
│   Configuration     │
│ (.specify/config)   │
└──────────┬──────────┘
           │ read by
           ▼
┌─────────────────────┐
│  create-new-feature │
│      scripts        │
├─────────────────────┤
│                     │
│  ┌───────┐ ┌──────┐ │
│  │Branch │ │Worktree│ │
│  │ Mode  │ │ Mode  │ │
│  └───┬───┘ └───┬───┘ │
│      │         │     │
└──────┼─────────┼─────┘
       │         │
       ▼         ▼
┌──────────┐ ┌──────────────┐
│ Git      │ │ Git Worktree │
│ Branch   │ │ + Directory  │
└──────────┘ └──────────────┘
```

---

## File Locations

| File | Purpose | Created By |
|------|---------|------------|
| `.specify/config.json` | Persistent configuration | `configure-worktree.sh` script |
| `.worktrees/<branch>/` | Nested worktree directory | `git worktree add` (when nested strategy) |
| `../<branch>/` | Sibling worktree directory | `git worktree add` (when sibling strategy) |
| `<custom>/<branch>/` | Custom worktree directory | `git worktree add` (when custom strategy) |

---

## Example Configurations

### Default (Branch Mode)

```json
{
  "git_mode": "branch"
}
```

### Nested Worktrees

```json
{
  "git_mode": "worktree",
  "worktree_strategy": "nested"
}
```

### Sibling Worktrees

```json
{
  "git_mode": "worktree",
  "worktree_strategy": "sibling"
}
```

### Custom Path Worktrees

```json
{
  "git_mode": "worktree",
  "worktree_strategy": "custom",
  "worktree_custom_path": "/tmp/my-worktrees"
}
```
