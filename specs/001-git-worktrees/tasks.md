# Tasks: Git Worktree Support

**Input**: Design documents from `/specs/001-git-worktrees/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/cli-interface.md

**Tests**: No automated tests requested. Manual validation per quickstart.md.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Source scripts**: `scripts/bash/`, `scripts/powershell/`
- **Installed scripts**: `.specify/scripts/bash/`
- Both locations must be updated to maintain consistency

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and shared utilities

- [x] T001 Add `.worktrees/` to .gitignore at repository root
- [x] T002 [P] Add `read_config_value()` function to scripts/bash/common.sh for JSON config reading with jq fallback
- [x] T003 [P] Add `Get-ConfigValue` function to scripts/powershell/common.ps1 for JSON config reading

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Configuration system that ALL user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T004 [P] Create configure-worktree.sh in scripts/bash/ with --mode, --strategy, --path, --show arguments per cli-interface.md
- [x] T005 [P] Create configure-worktree.ps1 in scripts/powershell/ with -Mode, -Strategy, -Path, -Show parameters per cli-interface.md
- [x] T006 Validate configuration scripts: test --show with no config, test setting each mode/strategy combination

**Checkpoint**: Configuration system ready - user story implementation can now begin

---

## Phase 3: User Story 1 - Parallel Feature Development (Priority: P1) 🎯 MVP

**Goal**: Enable worktree creation when configured, creating separate working directories for each feature branch

**Independent Test**: Run `configure-worktree.sh --mode worktree --strategy nested`, then `create-new-feature.sh --json "Test feature"` and verify a worktree is created in `.worktrees/` with the full project structure

### Implementation for User Story 1

- [x] T007 [US1] Add worktree path calculation logic to scripts/bash/create-new-feature.sh (nested, sibling, custom strategies per research.md section 3)
- [x] T008 [US1] Add config reading to scripts/bash/create-new-feature.sh to check git_mode and worktree_strategy
- [x] T009 [US1] Implement `git worktree add` command execution in scripts/bash/create-new-feature.sh for new branches
- [x] T010 [US1] Handle existing branch case in scripts/bash/create-new-feature.sh (use `git worktree add <path> <branch>` without -b flag)
- [x] T011 [P] [US1] Add worktree path calculation logic to scripts/powershell/create-new-feature.ps1 (mirror Bash implementation)
- [x] T012 [P] [US1] Add config reading to scripts/powershell/create-new-feature.ps1 to check git_mode and worktree_strategy
- [x] T013 [US1] Implement `git worktree add` command execution in scripts/powershell/create-new-feature.ps1
- [x] T014 [US1] Handle existing branch case in scripts/powershell/create-new-feature.ps1 (mirror Bash implementation)
- [x] T015 [US1] Validate nested strategy: create feature with nested config, verify worktree at `.worktrees/<branch>/`
- [x] T016 [US1] Validate sibling strategy: create feature with sibling config, verify worktree at `../<branch>/`

**Checkpoint**: Worktree creation works with all three strategies (nested, sibling, custom)

---

## Phase 4: User Story 4 - AI Agent Context Awareness (Priority: P1)

**Goal**: Extend JSON output with FEATURE_ROOT and MODE fields so AI agents know the working directory

**Independent Test**: Run `create-new-feature.sh --json "Test"` in both branch mode and worktree mode, verify JSON contains `FEATURE_ROOT` (absolute path) and `MODE` ("branch" or "worktree")

### Implementation for User Story 4

- [x] T017 [US4] Add FEATURE_ROOT field to JSON output in scripts/bash/create-new-feature.sh (repo root in branch mode, worktree path in worktree mode)
- [x] T018 [US4] Add MODE field to JSON output in scripts/bash/create-new-feature.sh ("branch" or "worktree")
- [x] T019 [P] [US4] Add FEATURE_ROOT and MODE fields to JSON output in scripts/powershell/create-new-feature.ps1
- [x] T020 [US4] Update .specify/templates/commands/specify.md to instruct AI agents to use FEATURE_ROOT for working directory
- [x] T021 [US4] Validate JSON output: verify both branch mode and worktree mode produce correct FEATURE_ROOT and MODE values

**Checkpoint**: AI agents can detect worktree mode and use correct working directory

---

## Phase 5: User Story 2 - Configure Worktree Preferences (Priority: P2)

**Goal**: Provide user-friendly configuration command with validation

**Builds on**: Phase 2 created the basic configure scripts (T004-T006). This phase adds input validation, path verification, and config persistence logic.

**Independent Test**: Run `configure-worktree.sh --mode worktree --strategy custom --path /tmp/test`, verify config.json is created with correct values and `--show` displays them

### Implementation for User Story 2

- [x] T022 [US2] Add input validation to scripts/bash/configure-worktree.sh (validate strategy enum, require --path for custom strategy)
- [x] T023 [US2] Add absolute path validation for --path argument in scripts/bash/configure-worktree.sh
- [x] T024 [US2] Implement config file creation/update logic in scripts/bash/configure-worktree.sh preserving existing fields
- [x] T025 [P] [US2] Add input validation to scripts/powershell/configure-worktree.ps1 (mirror Bash validation)
- [x] T026 [P] [US2] Add absolute path validation for -Path parameter in scripts/powershell/configure-worktree.ps1
- [x] T027 [US2] Implement config file creation/update logic in scripts/powershell/configure-worktree.ps1 preserving existing fields
- [x] T028 [US2] Validate configuration persistence: set config, restart terminal, verify `--show` displays saved values

**Checkpoint**: Configuration persists across sessions and validates user input

---

## Phase 6: User Story 3 - Graceful Fallback (Priority: P3)

**Goal**: Automatically fall back to branch mode when worktree creation fails

**Independent Test**: Configure worktree mode with an invalid/unwritable path, run create-new-feature, verify fallback to branch mode with warning

### Implementation for User Story 3

- [x] T029 [US3] Add path writability check before worktree creation in scripts/bash/create-new-feature.sh
- [x] T030 [US3] Implement fallback to branch mode on worktree failure in scripts/bash/create-new-feature.sh with `[specify] Warning:` message
- [x] T031 [US3] Add uncommitted changes detection and warning in scripts/bash/create-new-feature.sh (per FR-013)
- [x] T032 [US3] Add orphaned worktree detection and warning in scripts/bash/create-new-feature.sh (per FR-012)
- [x] T033 [P] [US3] Add path writability check before worktree creation in scripts/powershell/create-new-feature.ps1
- [x] T034 [P] [US3] Implement fallback to branch mode on worktree failure in scripts/powershell/create-new-feature.ps1
- [x] T035 [US3] Add uncommitted changes detection and warning in scripts/powershell/create-new-feature.ps1
- [x] T036 [US3] Add orphaned worktree detection and warning in scripts/powershell/create-new-feature.ps1
- [x] T037 [US3] Validate fallback: set unwritable path, create feature, verify branch mode fallback with warning matching `[specify] Warning:` format (per FR-011)

**Checkpoint**: System gracefully handles all failure scenarios

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Sync installed scripts and final validation

- [x] T038 [P] Copy updated scripts/bash/common.sh to .specify/scripts/bash/common.sh
- [x] T039 [P] Copy updated scripts/bash/create-new-feature.sh to .specify/scripts/bash/create-new-feature.sh
- [x] T040 [P] Copy scripts/bash/configure-worktree.sh to .specify/scripts/bash/configure-worktree.sh
- [x] T041 Run full quickstart.md validation: enable worktree mode, create feature, switch modes, verify all scenarios
- [x] T042 Verify cross-platform parity: test identical scenarios on Bash and PowerShell

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-6)**: All depend on Foundational phase completion
  - US1 (Phase 3): First priority, core functionality
  - US4 (Phase 4): Depends on US1 (JSON output builds on worktree creation)
  - US2 (Phase 5): Can run parallel to US1/US4 (different scripts)
  - US3 (Phase 6): Depends on US1 (fallback requires worktree logic)
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational - No dependencies on other stories
- **User Story 4 (P1)**: Depends on US1 tasks T007-T014 being complete
- **User Story 2 (P2)**: Can start after Foundational - Independent of US1/US4
- **User Story 3 (P3)**: Depends on US1 - Builds fallback logic around worktree creation

### Within Each User Story

- Bash implementation before PowerShell (establish pattern first)
- Core logic before validation
- Both platforms must be feature-complete before story is done

### Parallel Opportunities

**Setup Phase (3 parallel tracks)**:
```
Track A: T001 (.gitignore)
Track B: T002 (Bash common.sh)
Track C: T003 (PowerShell common.ps1)
```

**Foundational Phase (2 parallel tracks)**:
```
Track A: T004 (Bash configure script)
Track B: T005 (PowerShell configure script)
Then: T006 (validation)
```

**User Story 1 (2 parallel tracks after T007-T010)**:
```
Track A: T007 → T008 → T009 → T010 (Bash)
Track B: T011 → T012 → T013 → T014 (PowerShell, can start after T007 establishes pattern)
Then: T015, T016 (validation)
```

**User Story 2 (2 parallel tracks)**:
```
Track A: T022 → T023 → T024 (Bash)
Track B: T025 → T026 → T027 (PowerShell)
Then: T028 (validation)
```

---

## Parallel Example: User Story 1

```bash
# After establishing Bash pattern (T007-T010), launch PowerShell in parallel:
Task: "Add worktree path calculation logic to scripts/powershell/create-new-feature.ps1"
Task: "Add config reading to scripts/powershell/create-new-feature.ps1"
```

---

## Implementation Strategy

### MVP First (User Story 1 + 4 Only)

1. Complete Phase 1: Setup (T001-T003)
2. Complete Phase 2: Foundational (T004-T006)
3. Complete Phase 3: User Story 1 (T007-T016)
4. Complete Phase 4: User Story 4 (T017-T021)
5. **STOP and VALIDATE**: Test worktree creation with all strategies, verify JSON output
6. Deploy if ready - users can create worktrees and AI agents work correctly

### Incremental Delivery

1. Setup + Foundational → Configuration system ready
2. Add User Story 1 + 4 → Core worktree + JSON output → **MVP!**
3. Add User Story 2 → Better configuration UX
4. Add User Story 3 → Robust error handling
5. Polish → Sync scripts, final validation

---

## Task Summary

| Phase | Tasks | Parallelizable |
|-------|-------|----------------|
| Setup | 3 | 2 |
| Foundational | 3 | 2 |
| US1 (P1) | 10 | 2 |
| US4 (P1) | 5 | 1 |
| US2 (P2) | 7 | 2 |
| US3 (P3) | 9 | 2 |
| Polish | 5 | 3 |
| **Total** | **42** | **14** |

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Scripts must be copied to both `scripts/` (source) and `.specify/scripts/` (installed) locations
