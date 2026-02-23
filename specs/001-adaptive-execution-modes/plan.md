# Implementation Plan: Adaptive Execution Modes

**Branch**: `001-adaptive-execution-modes` | **Date**: 2026-02-21 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/001-adaptive-execution-modes/spec.md`

## Summary

Add a three-mode execution system (`fast`, `balanced`, `detailed`) to speckit that controls workflow depth, clarification limits, and AR/SEC artifact gating across all command templates. Fast mode targets ≥40% token reduction vs. detailed mode by skipping non-essential steps and deferring architecture/security artifacts unless risk keywords are detected in the spec text. The feature is implemented as changes to bash scripts (`common.sh`, `check-prerequisites.sh`) and command templates (`specify.md`, `clarify.md`, `plan.md`) with a new per-feature config file (`.feature-config.json`) for mode persistence.

## Technical Context

**Language/Version**: Bash (POSIX-compatible, sh-compatible) + Markdown (command templates)
**Primary Dependencies**: `scripts/bash/common.sh` (shared functions), `scripts/bash/check-prerequisites.sh` (JSON output layer), `templates/commands/` (AI workflow templates)
**Storage**: `.specify/config.json` (project default mode), `specs/{feature}/.feature-config.json` (per-feature selected mode)
**Testing**: Manual end-to-end via speckit self-hosting; bash unit tests with `bats` (if available) for `detect_risk_triggers()`
**Target Platform**: macOS/Linux CLI (bash scripts), AI agent slash commands (Markdown templates)
**Project Type**: single
**Performance Goals**: Fast mode must reduce token count per feature run by ≥40% vs. detailed mode baseline (measured via run-summary.md self-reporting)
**Constraints**: Must be backward-compatible — existing features without `.feature-config.json` fall back gracefully to `balanced`; no changes to Python CLI required
**Scale/Scope**: Per-feature-run, single user; no concurrency concerns

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Constitution is unpopulated (template placeholders only). No active gates to evaluate. **Proceeding without violations.**

## Project Structure

### Documentation (this feature)

```text
specs/001-adaptive-execution-modes/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
│   ├── mode-behavior.md
│   └── risk-triggers.md
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
scripts/bash/
├── common.sh                    # Add: detect_risk_triggers(), read_feature_config_value(), get_execution_mode()
└── check-prerequisites.sh       # Add: EXECUTION_MODE, HAS_RISK_TRIGGERS, RISK_TRIGGERS to JSON output

templates/commands/
├── specify.md                   # Add: mode selection prompt + .feature-config.json write
├── clarify.md                   # Add: mode-aware question limit (fast=2, balanced=4, detailed=5)
└── plan.md                      # Add: mode+trigger gating for AR/SEC; run-summary generation

.specify/
└── config.json                  # Document: defaultMode key (added to schema, not written by code)
```

**Structure Decision**: Single project — all changes are in-place modifications to existing script and template files. No new top-level directories. The per-feature `.feature-config.json` is a new file type written to each feature's `specs/{feature}/` directory.

## Complexity Tracking

*No Constitution violations — section left blank.*

---

## Phase 0: Research Output

Research complete. See [research.md](research.md) for full decision rationale.

**All NEEDS CLARIFICATION resolved**:

| Unknown | Resolution |
|---|---|
| Mode persistence | Two-layer: `.specify/config.json` (project default) + `specs/{feature}/.feature-config.json` (per-feature) |
| Mode selection UX | AI prompt in `specify.md` template; written to `.feature-config.json` before spec generation |
| Risk trigger mechanism | `detect_risk_triggers()` in `common.sh`; grep-based, keyword catalog v1 |
| AR/SEC gating | Template-level branching on mode + `HAS_RISK_TRIGGERS` from `check-prerequisites.sh` |
| Token measurement | AI self-reporting in `run-summary.md` |
| Run summary | `specs/{feature}/run-summary.md`, overwritten per plan run |

---

## Phase 1: Design

### 1.1 Data Model

See [data-model.md](data-model.md) for full entity definitions.

**Key entities**:
- `ExecutionMode` — enum `fast|balanced|detailed` with source tracking
- `RiskTrigger` — detected keyword + context
- `ArtifactPolicy` — per-mode required/optional/skipped artifact rules
- `RunSummary` — per-run metrics record
- `FeatureConfig` — JSON sidecar file for per-feature mode persistence

### 1.2 Contracts

See [contracts/mode-behavior.md](contracts/mode-behavior.md) and [contracts/risk-triggers.md](contracts/risk-triggers.md).

**Key invariants**:
- `run-summary.md` always generated
- `plan.md` always generated
- Risk triggers always scanned; escalation behavior differs by mode
- Invalid mode value → immediate error with remediation guidance

### 1.3 Implementation Design by File

#### `scripts/bash/common.sh` — New Functions

```bash
# Read execution mode from .feature-config.json, falling back to config default then "balanced"
# Usage: get_execution_mode [feature_dir]
get_execution_mode() { ... }

# Scan spec file for risk-indicating keywords
# Usage: detect_risk_triggers <spec_file_path>
# Returns: space-separated matched keywords (empty if none)
detect_risk_triggers() { ... }

# Read a value from a feature-level config file
# Usage: read_feature_config_value <key> [feature_dir] [default]
read_feature_config_value() { ... }
```

#### `scripts/bash/check-prerequisites.sh` — Extended JSON Output

Add to the `get_feature_paths`-equivalent output:
```json
{
  "EXECUTION_MODE": "fast",
  "HAS_RISK_TRIGGERS": "true",
  "RISK_TRIGGERS": "auth payment"
}
```

#### `templates/commands/specify.md` — Mode Selection Block

Insert after feature scaffolding script, before spec generation:

```markdown
## Mode Selection (NEW)

1. Read project default: `read_config_value "defaultMode" "balanced"`
2. Present interactive prompt with current default pre-selected
3. User confirms or overrides
4. Write to `FEATURE_ROOT/.feature-config.json`:
   `{"mode": "<selected>", "mode_source": "<user-selected|config-default>", "mode_selected_at": "<date>"}`
5. Continue with spec generation
```

#### `templates/commands/clarify.md` — Mode-Aware Limits

Insert after prerequisites check:

```markdown
## Mode-Aware Question Limit (NEW)

- Read EXECUTION_MODE from prerequisites JSON
- fast → max 2 questions
- balanced → max 4 questions
- detailed → max 5 questions (default, no change)
- Scan for risk triggers; if found, notify before questioning starts
```

#### `templates/commands/plan.md` — Mode + Trigger Gating

Insert before artifact generation steps:

```markdown
## Mode + Risk Gate (NEW)

- Read EXECUTION_MODE and HAS_RISK_TRIGGERS from prerequisites JSON
- If mode=detailed:
  - Generate AR first, then SEC (unconditional, no notification needed)
- If (mode=fast OR mode=balanced) AND HAS_RISK_TRIGGERS=true:
  - Notify user of trigger keywords
  - Generate AR artifact (specs/{feature}/ar.md)
  - Generate SEC artifact (specs/{feature}/sec.md)
  - Set escalated=true
- If (mode=fast OR mode=balanced) AND HAS_RISK_TRIGGERS=false:
  - Skip AR and SEC
- Plan depth: fast→condensed, balanced→full, detailed→full with rationale
- Always: generate run-summary.md after plan complete
```

### 1.4 Run Summary Template

```markdown
# Run Summary: {FEATURE_BRANCH}

**Date**: {DATE}
**Execution Mode**: {MODE} ({MODE_SOURCE})

## Risk Assessment

**Triggers Detected**: {RISK_TRIGGERS or "None"}
**Escalated**: {Yes/No}

## Artifacts Generated

{list of filenames}

## Token Estimate

**Estimated tokens this run**: {N}

*Token count is self-reported by the AI agent based on context window usage.*
```

---

## Agent Context Update

After filling Technical Context above, run:

```bash
./scripts/bash/update-agent-context.sh claude
```

This updates `CLAUDE.md` with the new technology stack entries for this feature.

---

## Post-Design Constitution Re-Check

Constitution is unpopulated. No violations introduced by design. **Passed.**

---

## Completion

**Branch**: `001-adaptive-execution-modes`
**Plan file**: `specs/001-adaptive-execution-modes/plan.md`
**Generated artifacts**:
- `specs/001-adaptive-execution-modes/research.md`
- `specs/001-adaptive-execution-modes/data-model.md`
- `specs/001-adaptive-execution-modes/quickstart.md`
- `specs/001-adaptive-execution-modes/contracts/mode-behavior.md`
- `specs/001-adaptive-execution-modes/contracts/risk-triggers.md`

**Next step**: Run `/speckit.tasks` to generate the actionable task breakdown.
