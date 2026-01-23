# Implementation Plan: Git Worktree Support

**Branch**: `001-git-worktrees` | **Date**: 2026-01-15 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-git-worktrees/spec.md`

## Summary

Add git worktree support to Spec Kit, enabling developers to work on multiple features simultaneously in separate working directories. The implementation modifies the existing `create-new-feature` scripts (Bash and PowerShell) to optionally create git worktrees instead of branches, with configurable placement strategies (nested, sibling, custom path) and graceful fallback to standard branch mode.

## Technical Context

**Language/Version**: Bash (POSIX-compatible) + PowerShell 7+
**Primary Dependencies**: Git 2.5+ (worktree support), jq (optional, for JSON parsing)
**Storage**: JSON configuration file (`.specify/config.json`)
**Testing**: Manual integration tests, shell script validation (shellcheck for Bash)
**Target Platform**: Cross-platform (macOS, Linux, Windows)
**Project Type**: CLI tooling / Shell scripts
**Performance Goals**: Feature creation under 5 seconds (matching current branch mode)
**Constraints**: Must maintain backward compatibility with existing branch mode as default
**Scale/Scope**: Single-user CLI tool, no concurrency concerns

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

**Note**: Project constitution (`.specify/memory/constitution.md`) contains template placeholders only. No specific gates defined. Proceeding with standard software engineering best practices:

- [x] **Backward Compatibility**: Default behavior unchanged (branch mode)
- [x] **Cross-Platform Parity**: Both Bash and PowerShell implement identical logic
- [x] **Error Handling**: Follows existing conventions (stderr for errors/warnings)
- [x] **Testability**: All features can be manually tested via script invocation

## Project Structure

### Documentation (this feature)

```text
specs/001-git-worktrees/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output (N/A - no API contracts)
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
.specify/
├── config.json                              # NEW: Worktree configuration
├── scripts/
│   └── bash/
│       ├── common.sh                        # MODIFY: Add config reading helpers
│       ├── create-new-feature.sh            # MODIFY: Add worktree logic
│       └── configure-worktree.sh            # NEW: Configuration script
└── templates/
    └── commands/
        └── specify.md                       # MODIFY: Update for worktree context

scripts/
├── bash/
│   ├── common.sh                            # MODIFY: Add config reading helpers
│   ├── create-new-feature.sh                # MODIFY: Add worktree logic
│   └── configure-worktree.sh                # NEW: Configuration script
└── powershell/
    ├── common.ps1                           # MODIFY: Add config reading helpers
    ├── create-new-feature.ps1               # MODIFY: Add worktree logic
    └── configure-worktree.ps1               # NEW: Configuration script

.gitignore                                   # MODIFY: Add .worktrees/ exclusion
```

**Structure Decision**: Single project structure maintained. Changes are additive modifications to existing scripts plus one new configuration script per platform. No new directories except the optional `.worktrees/` for nested worktree storage.

## Complexity Tracking

> No constitution violations to justify. Implementation follows existing patterns.

| Aspect | Decision | Rationale |
|--------|----------|-----------|
| Configuration format | JSON | Consistent with existing JSON output mode; easily parsed in both Bash (jq) and PowerShell |
| Worktree creation | `git worktree add` | Native Git command, well-supported across platforms |
| Fallback strategy | Automatic | Reduces user friction; warnings provide visibility without blocking |
