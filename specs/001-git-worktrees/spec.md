# Feature Specification: Git Worktree Support

**Feature Branch**: `001-git-worktrees`
**Created**: 2026-01-15
**Status**: Draft
**Input**: User description: "Add git worktree support for parallel feature development"

## Clarifications

### Session 2026-01-15

- Q: What happens when user tries to create a worktree for an existing branch? → A: Allow creating worktree for existing branch (attach to it)
- Q: How should errors and warnings be reported? → A: Follow existing pattern (stderr for errors/warnings, JSON for success data only)
- Q: How to handle orphaned worktree entries (manually deleted directories)? → A: Detect and warn user, but proceed with operation
- Q: Behavior when main repository has uncommitted changes? → A: Warn user that uncommitted changes won't appear in new worktree, then proceed

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Parallel Feature Development (Priority: P1)

As a developer working on multiple features simultaneously, I want to create separate working directories for each feature branch so that I can switch between features without stashing or committing incomplete work.

**Why this priority**: This is the core value proposition of the feature. Developers frequently need to context-switch between features (e.g., urgent bug fix while working on a new feature), and current branch-switching requires managing uncommitted changes.

**Independent Test**: Can be fully tested by creating a new feature with worktree mode enabled and verifying that a separate working directory is created with the full project structure, allowing independent work on multiple features.

**Acceptance Scenarios**:

1. **Given** a user has worktree mode enabled in configuration, **When** they run the create-new-feature script, **Then** a new worktree is created in the configured location with the feature branch checked out.
2. **Given** a user has an existing worktree for feature A, **When** they create a new feature B with worktree mode, **Then** feature B gets its own separate worktree directory without affecting feature A.
3. **Given** a user creates a feature with worktree mode, **When** the worktree is created, **Then** all tracked files (templates, .specify directory) are available in the new worktree.

---

### User Story 2 - Configure Worktree Preferences (Priority: P2)

As a developer, I want to configure where worktrees are created (nested within project, sibling directories, or custom path) so that my workflow matches my file organization preferences and environment constraints.

**Why this priority**: Different environments have different constraints (sandboxed containers may not allow sibling directories), and developers have personal preferences for project organization. This flexibility is essential for adoption.

**Independent Test**: Can be tested by setting different worktree strategies in configuration and verifying that worktrees are created in the expected locations.

**Acceptance Scenarios**:

1. **Given** a user sets worktree strategy to "nested", **When** they create a new feature, **Then** the worktree is created inside `.worktrees/<branch-name>` within the project root.
2. **Given** a user sets worktree strategy to "sibling", **When** they create a new feature, **Then** the worktree is created as a sibling directory to the project (e.g., `../<branch-name>`).
3. **Given** a user sets worktree strategy to "custom" with a specific path, **When** they create a new feature, **Then** the worktree is created at `<custom-path>/<branch-name>`.

---

### User Story 3 - Graceful Fallback (Priority: P3)

As a developer working in a restricted environment, I want the system to gracefully fall back to standard branch creation if worktree creation fails so that I can still use the tool without manual intervention.

**Why this priority**: Reliability is important for tool adoption. Sandbox environments, permission issues, or other constraints should not break the workflow entirely.

**Independent Test**: Can be tested by simulating a worktree creation failure (e.g., invalid path permissions) and verifying that the system falls back to standard branch creation with a warning.

**Acceptance Scenarios**:

1. **Given** a user has worktree mode enabled but the target path is not writable, **When** they create a new feature, **Then** the system falls back to standard branch creation and displays a warning message.
2. **Given** a user has worktree mode with sibling strategy in a sandboxed container without parent directory access, **When** they create a new feature, **Then** the system detects the constraint and either uses nested strategy or falls back to branch mode with a clear error message.

---

### User Story 4 - AI Agent Context Awareness (Priority: P1)

As a developer using AI-assisted development, I want the AI agent to automatically understand when working in a worktree context so that all file operations target the correct worktree directory.

**Why this priority**: This is critical for the integration with AI agents (Claude, Copilot, etc.). Without proper context awareness, the AI could modify files in the wrong directory, causing confusion and potential data loss.

**Independent Test**: Can be tested by creating a feature with worktree mode and verifying that the script output includes the worktree path, and that subsequent commands from the AI agent operate in the correct directory.

**Acceptance Scenarios**:

1. **Given** a feature is created in worktree mode, **When** the script outputs JSON, **Then** the output includes `FEATURE_ROOT` with the absolute path to the worktree and `MODE` indicating "worktree".
2. **Given** an AI agent receives worktree context, **When** it performs file operations for the feature, **Then** all operations target files within the `FEATURE_ROOT` path.
3. **Given** a worktree is created, **When** the AI agent needs to run commands (build, test), **Then** the agent changes to the `FEATURE_ROOT` directory before executing commands.

---

### Edge Cases

- **Existing branch**: When creating a worktree for a branch that already exists, the system attaches the worktree to the existing branch rather than blocking or creating a new branch.
- **Disk space insufficient**: Git will fail with a clear error message; system surfaces this error to the user via stderr following standard error handling conventions.
- **Orphaned worktree**: If a user deletes a worktree directory manually without using `git worktree remove`, the system detects the orphaned entry and warns the user, but proceeds with the requested operation.
- **Uncommitted changes**: When the main repository has uncommitted changes, the system warns the user that these changes will not appear in the new worktree, then proceeds with worktree creation.
- **Path length limit**: On Windows, if the worktree path exceeds file system limits, the system detects the error from Git and warns the user, suggesting they use the "sibling" or "custom" strategy with a shorter base path.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST support a configuration file (`.specify/config.json`) that stores worktree preferences.
- **FR-002**: System MUST support three worktree placement strategies: nested (inside project), sibling (next to project), and custom (user-specified path).
- **FR-003**: System MUST default to "branch" mode (current behavior) when no configuration exists or when worktree mode is not explicitly enabled.
- **FR-004**: System MUST create the worktree using `git worktree add` command when worktree mode is enabled.
- **FR-005**: System MUST output absolute paths in JSON output, including `FEATURE_ROOT` for the worktree location and `MODE` to indicate the creation mode.
- **FR-006**: System MUST fall back to standard branch creation if worktree creation fails, with a clear warning message to the user.
- **FR-007**: System MUST ensure `.gitignore` excludes the nested worktrees directory (`.worktrees/`) to prevent tracking worktree metadata.
- **FR-008**: System MUST provide a mechanism for users to configure worktree preferences (either via CLI command or script).
- **FR-009**: Both Bash and PowerShell scripts MUST implement identical worktree logic to maintain cross-platform consistency.
- **FR-010**: System MUST validate that the target worktree path is writable before attempting creation.
- **FR-011**: System MUST follow existing error handling conventions: errors and warnings to stderr with `[specify]` prefix, JSON output for success data only.
- **FR-012**: System MUST detect orphaned worktree entries (directories deleted without `git worktree remove`) and warn the user before proceeding with new worktree creation.
- **FR-013**: System MUST warn users when creating a worktree while uncommitted changes exist in the current working directory, informing them these changes will not appear in the new worktree.

### Key Entities

- **Configuration**: User preferences for git mode (branch/worktree), worktree placement strategy, and custom path settings. Stored in `.specify/config.json`.
- **Worktree**: A separate working directory linked to the main repository, containing a checkout of a specific branch. Created via `git worktree add`.
- **Feature Root**: The base directory for a feature's files. In branch mode, this is the repository root. In worktree mode, this is the worktree directory path.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can create a new feature in worktree mode within the same time as branch mode (under 5 seconds).
- **SC-002**: Users can work on two or more features simultaneously without needing to stash or commit incomplete changes.
- **SC-003**: 100% of tracked project files (templates, configuration, source code) are accessible in newly created worktrees.
- **SC-004**: System successfully falls back to branch mode in 100% of cases where worktree creation fails due to environmental constraints.
- **SC-005**: AI agents correctly identify and use the worktree path for all file operations when `MODE` is "worktree".
- **SC-006**: Configuration changes take effect immediately for the next feature creation without requiring tool restart.

## Assumptions

- Users have Git version 2.5 or later installed (git worktree was introduced in 2.5).
- The main repository is a valid git repository with at least one commit.
- Users understand the concept of git worktrees and choose to enable the feature intentionally.
- Nested worktree strategy is the safest default for sandboxed environments.
- Sibling worktree strategy is preferred by users who want cleaner project separation.
- AI agents (Claude, Copilot, Cursor, etc.) can interpret JSON output and adjust their working directory accordingly.

## Out of Scope

- Automatic cleanup of stale worktrees (users manage worktree lifecycle manually via `git worktree remove`).
- GUI or visual interface for worktree management.
- Integration with IDE-specific worktree features.
- Worktree synchronization or conflict resolution between worktrees.
- Support for bare repositories.
