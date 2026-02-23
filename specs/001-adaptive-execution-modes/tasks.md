# Tasks: Adaptive Execution Modes

**Input**: Design documents from `specs/001-adaptive-execution-modes/`
**Prerequisites**: plan.md ✓, spec.md ✓, data-model.md ✓, research.md ✓, quickstart.md ✓, contracts/ ✓

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no shared state dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)

---

## Phase 1: Setup

**Purpose**: Verify environment and file permissions before modifying scripts

- [x] T001 Verify scripts/bash/ execute permissions — run `ls -la scripts/bash/*.sh` and `chmod +x scripts/bash/*.sh` if any are missing execute bit; confirm `scripts/bash/common.sh` is present (the `.specify/scripts/bash/common.sh` symlink/copy is missing and must be investigated as part of this task)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core bash helper functions and JSON output layer that ALL three user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T002 Add `read_feature_config_value()` function to `scripts/bash/common.sh` — reads a named key from `specs/{feature}/.feature-config.json` using the same jq/grep-fallback pattern as the existing `read_config_value()` function; signature: `read_feature_config_value <key> [feature_dir] [default]`; returns value or default if file/key absent

- [x] T003 Add `detect_risk_triggers()` function to `scripts/bash/common.sh` — scans spec.md content (case-insensitive) for the v1 keyword catalog defined in `specs/001-adaptive-execution-modes/contracts/risk-triggers.md`; returns space-separated list of matched keywords (empty string if none); uses grep with `-i -w` for single-word keywords and `-i` for multi-word/hyphenated keywords; exit code always 0 (non-fatal); signature: `detect_risk_triggers <spec_file_path>`

- [x] T004 Add `get_execution_mode()` function to `scripts/bash/common.sh` — reads mode from `{feature_dir}/.feature-config.json` via `read_feature_config_value`; falls back to `read_config_value "defaultMode"` from `.specify/config.json`; final fallback is `"balanced"`; validates value is one of `fast|balanced|detailed` and emits warning on invalid; signature: `get_execution_mode [feature_dir]`

- [x] T005 Extend `scripts/bash/check-prerequisites.sh` JSON output to include three new fields — `EXECUTION_MODE` (string from `get_execution_mode`), `HAS_RISK_TRIGGERS` ("true" or "false" string from `detect_risk_triggers` result), `RISK_TRIGGERS` (space-separated matched keywords or empty string); add these fields to the `printf` JSON payload in the existing `--json` output path; must maintain backward-compatibility with existing JSON consumers

**Checkpoint**: Foundation ready — `scripts/bash/check-prerequisites.sh --json` now returns EXECUTION_MODE, HAS_RISK_TRIGGERS, RISK_TRIGGERS alongside existing fields

---

## Phase 3: User Story 1 — Accelerate low-risk feature setup (Priority: P1) 🎯 MVP

**Goal**: Fast mode compresses the speckit workflow for low-risk features, limiting clarification to 2 questions and skipping AR/SEC unless risk keywords are detected — delivering a usable plan in fewer tokens.

**Independent Test**: Create a new test feature spec containing no risk trigger keywords, select fast mode at the `/speckit.specify` prompt, run `/speckit.clarify` (verify ≤2 questions are asked), run `/speckit.plan` (verify no ar.md or sec.md are generated, run-summary.md is created with mode="fast" and token count populated).

### Implementation for User Story 1

- [x] T006 [US1] Add mode selection interactive prompt to `templates/commands/specify.md` — insert a new "Mode Selection" step after the feature scaffolding script call and before spec generation; the step must: (1) read project default via `read_config_value "defaultMode" "balanced" .specify/config.json`, (2) present an interactive prompt listing fast/balanced/detailed with the project default pre-selected, (3) after user confirmation write the selection to `FEATURE_ROOT/.feature-config.json` as `{"mode": "<value>", "mode_source": "user-selected", "mode_selected_at": "<YYYY-MM-DD>"}` (or `"mode_source": "config-default"` if user accepted the default without change), (4) continue to spec generation; implements FR-001

- [x] T007 [P] [US1] Add mode-aware question quota to `templates/commands/clarify.md` — insert a "Mode-Aware Limit" block immediately after the prerequisites JSON is parsed; read EXECUTION_MODE from the JSON output; apply quota: fast→2, balanced→4, detailed→5; replace the hardcoded "maximum 5 total questions" rule in the existing outline with this mode-dispatched value; implements FR-003

- [x] T008 [US1] Add mode+trigger AR/SEC gate to `templates/commands/plan.md` — insert a "Mode + Risk Gate" block before artifact generation steps; logic: if `EXECUTION_MODE=fast` AND `HAS_RISK_TRIGGERS=false`, skip AR (`ar.md`) and SEC (`sec.md`) generation entirely, and use condensed plan depth (merge Summary and Technical Context into a single brief block, omit PRD cross-reference section); implements FR-004

- [x] T009 [US1] Add risk trigger escalation notification block to `templates/commands/plan.md` — within the Mode + Risk Gate block (from T008), add the trigger-detected path: when `HAS_RISK_TRIGGERS=true`, display `"⚠️ Risk triggers detected in spec: [{RISK_TRIGGERS}]. Adding Architecture Review (AR) and Security Review (SEC) to this run. Continuing with escalated artifact set..."` before AR/SEC generation; notification must appear BEFORE any escalated artifact generation begins; implements FR-005, FR-010

- [x] T010 [US1] Add run-summary.md generation step to `templates/commands/plan.md` — append a final "Run Summary" step that creates `specs/{feature}/run-summary.md` using the RunSummary schema from `data-model.md`; fields: feature_branch, run_date, execution_mode (from EXECUTION_MODE), mode_source (from .feature-config.json), risk_triggers_detected (from RISK_TRIGGERS, as list), artifacts_generated (list of filenames created this run), estimated_token_count (AI self-reports approximate context window tokens consumed), escalated (true/false); implements FR-008, FR-009, SC-001, SC-002

**Checkpoint**: User Story 1 is fully functional — fast mode selects, limits clarification, skips AR/SEC, notifies on triggers, and writes run-summary.md

---

## Phase 4: User Story 2 — Keep a balanced default workflow (Priority: P2)

**Goal**: Balanced mode is the default for teams that want standard planning quality with risk-based AR/SEC inclusion and 4-question clarification depth.

**Independent Test**: Start a new feature without setting `defaultMode` in `.specify/config.json` — verify balanced is pre-selected in the mode prompt. Run `/speckit.clarify` with a spec that has no risk keywords — verify up to 4 questions are asked and no AR/SEC notification appears. Add a risk keyword ("auth") to the spec and re-run `/speckit.plan` — verify escalation notification appears and ar.md + sec.md are generated. Check run-summary.md shows mode="balanced".

### Implementation for User Story 2

- [x] T011 [US2] Verify `get_execution_mode()` fallback is `"balanced"` and document `defaultMode` key — (1) confirm the final fallback in the T004 implementation returns "balanced" with a brief warning; (2) add an inline comment to `.specify/config.json` (or an adjacent `.specify/config.schema.md` if a schema doc exists) documenting the `defaultMode` key with valid values `fast|balanced|detailed` and the default behavior if absent; implements FR-006 and the assumption from spec.md

- [x] T012 [US2] Add balanced mode full plan depth path to `templates/commands/plan.md` — extend the Mode + Risk Gate block from T008 to handle `EXECUTION_MODE=balanced`: use full plan depth (all Technical Context fields, no merging), same risk-based AR/SEC gating as fast mode (skip unless HAS_RISK_TRIGGERS=true); distinguish balanced from fast only in plan depth, not in trigger behavior; implements FR-006

**Checkpoint**: User Story 2 functional — balanced mode uses full plan depth, risk-based AR/SEC, and allows up to 4 clarification questions; run-summary.md shows "balanced"

---

## Phase 5: User Story 3 — Enforce full rigor when needed (Priority: P3)

**Goal**: Detailed mode always generates both AR and SEC artifacts unconditionally, applies full plan depth including pre-plan review order, and allows up to 5 clarification questions.

**Independent Test**: Start a feature, select detailed mode. Run `/speckit.plan` on a spec with NO risk trigger keywords — verify ar.md AND sec.md are still generated (no trigger required). Verify run-summary.md shows mode="detailed" and both artifacts listed. Confirm `/speckit.clarify` asks up to 5 questions (not capped lower).

### Implementation for User Story 3

- [x] T013 [US3] Add detailed mode unconditional AR/SEC generation to `templates/commands/plan.md` — extend Mode + Risk Gate block from T012: when `EXECUTION_MODE=detailed`, always invoke AR and SEC generation regardless of `HAS_RISK_TRIGGERS`; no notification needed (user selected detailed mode knowing this behavior); implements FR-007

- [x] T014 [US3] Add detailed mode full pre-plan review ordering to `templates/commands/plan.md` — for detailed mode, insert explicit ordering instruction: AR must be generated first, then SEC, then plan.md — establishing the rationale-before-implementation sequence; add instruction to include explicit decision rationale section in plan output; implements FR-007 acceptance scenario 2

**Checkpoint**: All three user stories functional — fast/balanced/detailed modes each behave per `contracts/mode-behavior.md`; all modes write run-summary.md

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Error handling, backward compatibility, and end-to-end validation

- [x] T015 Add invalid mode error handling to `templates/commands/specify.md`, `templates/commands/clarify.md`, and `templates/commands/plan.md` — each command must validate that the mode value read from `.feature-config.json` is one of `fast|balanced|detailed`; if invalid, emit: `"ERROR: Unknown execution mode '{value}'. Valid values: fast, balanced, detailed. Re-run /speckit.specify to reset."`; handles the unsupported-mode edge case from spec.md

- [x] T016 Add backward-compatibility fallback to `templates/commands/clarify.md` and `templates/commands/plan.md` — if `.feature-config.json` does not exist for a feature (features created before this feature ships), treat mode as "balanced" with a brief notice: `"Note: No execution mode configured for this feature. Defaulting to balanced. Run /speckit.specify to set a mode explicitly."`; must not break existing features

- [x] T017 [P] Update `CLAUDE.md` Recent Changes section to reflect 001-adaptive-execution-modes completion — add entry: `"001-adaptive-execution-modes: Added adaptive execution modes (fast/balanced/detailed), detect_risk_triggers(), get_execution_mode() to common.sh, mode-aware clarify/plan templates, run-summary.md"`; agent context script already added Active Technologies entries in Phase 1

- [x] T018 Establish token count baselines for SC-001/SC-002 — run one representative test feature through the full `/speckit.specify` → `/speckit.clarify` → `/speckit.plan` workflow in detailed mode, record run-summary.md token count as the detailed baseline; repeat the same feature description in balanced mode, record as balanced baseline; these baselines are compared against fast-mode runs in T019

- [x] T019 End-to-end validation — run a complete fast-mode workflow on the same test feature used in T018 with no risk keywords: (1) `/speckit.specify` → confirm mode prompt shown, fast selected, .feature-config.json written; (2) `/speckit.clarify` → confirm ≤2 questions asked; (3) `/speckit.plan` → confirm no ar.md/sec.md created, run-summary.md written with token count; compare token count to T018 baselines for SC-001/SC-002; then repeat with a risk keyword ("auth") in the spec and verify escalation notification + ar.md + sec.md created; validate all three SC-004 communication points (mode prompt, trigger notification, run-summary.md); validate against acceptance scenarios in spec.md US1

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 — BLOCKS all user stories
- **US1 (Phase 3)**: Depends on Phase 2 completion; T007 [P] with T008 (different files)
- **US2 (Phase 4)**: Depends on Phase 2; T011 independent of US1; T012 depends on T008 being in plan.md first
- **US3 (Phase 5)**: Depends on Phase 2; T013 and T014 depend on T012 (balanced gate already in plan.md)
- **Polish (Phase 6)**: Depends on Phases 3–5 complete; T015, T016, T017 can run in parallel; T018 depends on all phases; T019 depends on T018

### User Story Dependencies

- **US1 (P1)**: Starts after Phase 2 — no dependency on US2 or US3
- **US2 (P2)**: Starts after Phase 2 — T012 extends the gate block started by T008 (US1), so US1 should land first
- **US3 (P3)**: Starts after Phase 2 — T013/T014 extend the gate block; US2 should land first to avoid conflicts on plan.md

### Parallel Opportunities

Within Phase 2: T002 → T003 → T004 → T005 (all in common.sh or check-prerequisites.sh; sequential within each file)

Within Phase 3: T007 [P] can run alongside T008 (different files: clarify.md vs plan.md)

Within Phase 6: T015, T016, T017 can run concurrently (different files or independent content)

---

## Parallel Example: User Story 1

```bash
# T007 and T008 can start in parallel after Phase 2:
Task: "Add mode-aware question quota to templates/commands/clarify.md"  # T007
Task: "Add mode+trigger AR/SEC gate to templates/commands/plan.md"      # T008

# T009 and T010 are sequential in plan.md after T008:
Task: "Add risk trigger escalation notification to templates/commands/plan.md"  # T009
Task: "Add run-summary.md generation step to templates/commands/plan.md"        # T010
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001)
2. Complete Phase 2: Foundational (T002–T005) — CRITICAL
3. Complete Phase 3: User Story 1 (T006–T010)
4. **STOP and VALIDATE**: Run end-to-end fast mode test per US1 Independent Test criteria
5. Proceed to US2/US3 once US1 is confirmed working

### Incremental Delivery

1. Setup + Foundational → bash layer ready
2. US1 → fast mode works end-to-end → validate → ship
3. US2 → balanced mode works end-to-end → validate → ship
4. US3 → detailed mode works end-to-end → validate → ship
5. Polish → error handling, backward compat, full E2E validation

### Single-Developer Strategy

Work sequentially: T001 → T002 → T003 → T004 → T005 → T006 → T007 (or T008 simultaneously if editing different files) → T008 → T009 → T010 → T011 → T012 → T013 → T014 → T015 → T016 → T017 → T018 → T019

---

## Notes

- [P] tasks = different files, no dependency on each other's incomplete output
- Each user story phase has an Independent Test — stop and validate before moving to next story
- plan.md template has sequential modifications across US1/US2/US3 — coordinate carefully to avoid merge conflicts if working in parallel
- T002–T004 all modify `scripts/bash/common.sh` — work sequentially to avoid conflicts
- run-summary.md is always generated (T010) — do not gate it behind mode checks
- `.feature-config.json` is written by the AI agent at runtime, not pre-created; tasks T002–T004 prepare the read-side; T006 prepares the write-side
