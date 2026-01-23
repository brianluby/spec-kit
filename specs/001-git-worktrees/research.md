# Research: Git Worktree Support

**Feature**: 001-git-worktrees
**Date**: 2026-01-15

## Research Summary

This document consolidates research findings for implementing git worktree support in Spec Kit. All technical context items were resolved during specification and planning phases; this research focuses on implementation best practices.

---

## 1. Git Worktree Command Patterns

### Decision: Use `git worktree add <path> -b <branch>` for new branches

**Rationale**: This is the standard pattern for creating a worktree with a new branch in a single command. For existing branches, use `git worktree add <path> <branch>` without the `-b` flag.

**Alternatives Considered**:
- `git worktree add <path>` (creates detached HEAD) - Rejected: Not useful for feature development
- Create branch first, then worktree - Rejected: Requires two commands, more error-prone

### Key Commands

```bash
# Create worktree with new branch
git worktree add <path> -b <branch-name>

# Create worktree for existing branch
git worktree add <path> <existing-branch>

# List worktrees
git worktree list

# Check for orphaned worktrees
git worktree list --porcelain | grep -A1 "^worktree" | grep -B1 "prunable"

# Prune orphaned entries (for reference, not auto-run)
git worktree prune
```

---

## 2. Configuration File Strategy

### Decision: Use `.specify/config.json` with explicit defaults

**Rationale**: JSON format is consistent with existing `--json` output mode. Both Bash (via jq or grep/sed) and PowerShell (via `ConvertFrom-Json`) can parse JSON natively.

**Alternatives Considered**:
- `.specify/config.yaml` - Rejected: Requires additional parser in Bash
- Environment variables - Rejected: Not persistent across sessions
- `.specifyrc` (dotfile) - Rejected: Inconsistent with existing `.specify/` structure

### Configuration Schema

```json
{
  "git_mode": "branch",
  "worktree_strategy": "nested",
  "worktree_custom_path": ""
}
```

| Field | Type | Default | Values |
|-------|------|---------|--------|
| `git_mode` | string | `"branch"` | `"branch"`, `"worktree"` |
| `worktree_strategy` | string | `"nested"` | `"nested"`, `"sibling"`, `"custom"` |
| `worktree_custom_path` | string | `""` | Absolute path (required if strategy is `"custom"`) |

---

## 3. Path Calculation Logic

### Decision: Calculate worktree path based on strategy before creation

**Rationale**: Pre-calculating the path allows validation (writability check) before attempting `git worktree add`, enabling graceful fallback.

### Path Formulas

| Strategy | Formula | Example |
|----------|---------|---------|
| `nested` | `<repo_root>/.worktrees/<branch>` | `/project/.worktrees/001-feature` |
| `sibling` | `../<branch>` | `/001-feature` (sibling to `/project`) |
| `custom` | `<custom_path>/<branch>` | `/tmp/worktrees/001-feature` |

### Validation Sequence

1. Calculate target path
2. Check if parent directory exists and is writable
3. Check if target path already exists (error if so)
4. Attempt `git worktree add`
5. On failure, fall back to branch mode with warning

---

## 4. JSON Output Extension

### Decision: Add `FEATURE_ROOT` and `MODE` fields to existing JSON output

**Rationale**: Extends existing JSON structure without breaking backward compatibility. AI agents can detect worktree mode and adjust working directory.

**Alternatives Considered**:
- Separate JSON schema for worktree mode - Rejected: Complicates consumer logic
- Include full worktree metadata - Rejected: Over-engineering for current needs

### Extended JSON Output

```json
{
  "BRANCH_NAME": "001-git-worktrees",
  "SPEC_FILE": "/path/to/worktree/specs/001-git-worktrees/spec.md",
  "FEATURE_NUM": "001",
  "FEATURE_ROOT": "/path/to/worktree",
  "MODE": "worktree"
}
```

| Field | Description | Present When |
|-------|-------------|--------------|
| `FEATURE_ROOT` | Absolute path to working directory | Always (repo root in branch mode) |
| `MODE` | Creation mode | Always (`"branch"` or `"worktree"`) |

---

## 5. Cross-Platform Considerations

### Decision: Implement identical logic in Bash and PowerShell

**Rationale**: Users on different platforms expect consistent behavior. Feature parity is required per FR-009.

### Platform-Specific Notes

| Aspect | Bash | PowerShell |
|--------|------|------------|
| JSON parsing | `jq` (if available) or `grep`/`sed` fallback | `ConvertFrom-Json` (native) |
| Path handling | POSIX paths | `Join-Path`, handles both separators |
| Git commands | Direct invocation | Same, via `git` CLI |
| Error output | `>&2 echo` | `Write-Warning`, `Write-Error` |

### jq Fallback Pattern (Bash)

```bash
read_config_value() {
    local key="$1"
    local config_file="$REPO_ROOT/.specify/config.json"

    if [ ! -f "$config_file" ]; then
        echo ""
        return
    fi

    if command -v jq &>/dev/null; then
        jq -r ".$key // empty" "$config_file" 2>/dev/null
    else
        # Fallback: simple grep for "key": "value" pattern
        grep -o "\"$key\"[[:space:]]*:[[:space:]]*\"[^\"]*\"" "$config_file" 2>/dev/null | \
            sed 's/.*:[[:space:]]*"\([^"]*\)".*/\1/'
    fi
}
```

---

## 6. Warning Detection Patterns

### Decision: Check for conditions before worktree creation, warn via stderr

**Rationale**: Consistent with existing `[specify] Warning:` pattern. Warnings inform without blocking.

### Conditions to Detect

| Condition | Detection Method | Warning Message |
|-----------|------------------|-----------------|
| Uncommitted changes | `git status --porcelain` non-empty | "Uncommitted changes will not appear in new worktree" |
| Orphaned worktrees | `git worktree list --porcelain` with "prunable" | "Orphaned worktree entries detected. Run `git worktree prune` to clean up." |
| Existing branch | `git rev-parse --verify <branch>` succeeds | (No warning, proceed with attach) |

---

## 7. Fallback Strategy

### Decision: Automatic fallback to branch mode on worktree failure

**Rationale**: Maintains tool usability in restricted environments without requiring user intervention.

### Fallback Triggers

1. Target path not writable
2. `git worktree add` command fails
3. Git version < 2.5 (worktree not supported)

### Fallback Behavior

```bash
# Pseudocode
if worktree_mode_enabled; then
    calculate_worktree_path
    if ! validate_path_writable; then
        warn "Cannot create worktree at $path. Falling back to branch mode."
        fallback_to_branch_mode
    elif ! git worktree add "$path" -b "$branch"; then
        warn "Worktree creation failed. Falling back to branch mode."
        fallback_to_branch_mode
    else
        success_worktree_mode
    fi
else
    standard_branch_mode
fi
```

---

## 8. .gitignore Update

### Decision: Add `.worktrees/` to project `.gitignore`

**Rationale**: Nested worktrees directory should not be tracked. Git already handles worktree metadata separately.

### Implementation

```bash
# Check if .worktrees/ already in .gitignore
if ! grep -q "^\.worktrees/$" .gitignore 2>/dev/null; then
    echo ".worktrees/" >> .gitignore
fi
```

---

## Research Conclusion

All technical decisions have been made. No outstanding NEEDS CLARIFICATION items. Ready to proceed to Phase 1: Design & Contracts.
