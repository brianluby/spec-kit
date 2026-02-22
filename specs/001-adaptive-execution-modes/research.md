# Research: Adaptive Execution Modes

**Branch**: `001-adaptive-execution-modes` | **Date**: 2026-02-21

## Decision 1: Mode State Persistence

**Decision**: Two-layer storage — project default in `.specify/config.json` under `defaultMode`; per-feature selected mode in `specs/{feature}/.feature-config.json` under `mode`.

**Rationale**: `read_config_value` in `scripts/bash/common.sh` already handles JSON reads from `.specify/config.json` (with jq/grep fallback). A parallel pattern for feature-level config avoids polluting the project config with transient state and allows future per-feature metadata extension.

**Alternatives considered**:
- Store in `spec.md` YAML frontmatter — rejected: spec.md is a human-readable design doc; embedding machine-readable state couples concerns.
- Store only in `.specify/config.json` as `currentMode` — rejected: overwrites conflict across worktrees and don't survive branch switches.
- Store in environment variable — rejected: doesn't persist across command invocations.

---

## Decision 2: Mode Selection Entry Point

**Decision**: Mode selection is presented by the AI command template (`templates/commands/specify.md`) immediately after feature scaffolding. The selected value is written to `.feature-config.json` by the AI before proceeding with spec generation.

**Rationale**: Speckit is AI-agent-driven — the shell scripts handle filesystem scaffolding, but workflow logic lives in the command templates. Adding mode selection to the template keeps the concern in the right layer and requires no changes to the Python CLI.

**Alternatives considered**:
- Add `--mode` flag to `create-new-feature.sh` — rejected: CLI entry is optional (users can start from `/speckit.specify` without the shell script); inconsistent enforcement.
- Prompt in `check-prerequisites.sh` — rejected: that script is read-only/diagnostic; mixing interactive prompts with JSON output breaks consumers.

---

## Decision 3: Risk Trigger Detection Mechanism

**Decision**: Add `detect_risk_triggers()` function to `scripts/bash/common.sh`. It scans the spec file text using grep for a predefined keyword list. Returns a space-separated list of matched trigger tokens (empty string if none). Expose results in `check-prerequisites.sh` JSON output as `RISK_TRIGGERS` and `HAS_RISK_TRIGGERS`.

**Keyword catalog** (v1):
`auth`, `authentication`, `authorization`, `payment`, `billing`, `pii`, `personal data`, `personal information`, `external api`, `third-party`, `third party`, `delete`, `destroy`, `drop`, `admin`, `compliance`, `regulation`, `gdpr`, `hipaa`, `encryption`, `secret`, `credential`, `token`, `password`

**Rationale**: Grep-based scanning is cheap, predictable, and requires no external dependencies. Lowercase comparison avoids case-sensitivity mismatches. The keyword list is the v1 catalog specified in the clarifications.

**Alternatives considered**:
- External risk catalog file (`.specify/risk-triggers.json`) — deferred: good for v2 when teams want to customize; adds operational overhead for v1.
- LLM-based semantic analysis — rejected: too expensive, non-deterministic; incompatible with the fast-mode goal of reducing token usage.

---

## Decision 4: Mode-Aware Behavior in Command Templates

**Decision**: Each affected command template reads mode from `.feature-config.json` inline (AI reads the file) and branches on its value. No new tooling layer.

**Mode behavior table**:

| Behavior | Fast | Balanced | Detailed |
|---|---|---|---|
| Max clarification questions | 2 | 4 | 5 |
| AR artifact | Skipped unless triggered | Risk-based | Always |
| SEC artifact | Skipped unless triggered | Risk-based | Always |
| Plan depth | Condensed (skip PRD, merge steps) | Standard | Full (PRD + AR + SEC before plan) |
| Run summary | Token count + mode + triggers | Token count + mode + triggers | Token count + mode + triggers |

**Rationale**: Keeping the logic in templates preserves the existing architecture where AI agents own workflow decisions. Shell scripts remain pure infrastructure.

---

## Decision 5: Token Count Measurement

**Decision**: Token count is self-reported by the AI agent in the Run Summary at the end of `/speckit.plan`. The AI uses its own context window awareness to estimate total tokens consumed during the workflow run. This is recorded in `run-summary.md`.

**Rationale**: There is no programmatic way to measure LLM token consumption from bash scripts without calling a separate API. Self-reporting is the only viable approach given the architecture. Teams can compare run-summary.md values across feature runs for SC-001/SC-002 validation.

**Alternatives considered**:
- Log API usage via a proxy — rejected: requires infrastructure change outside speckit scope.
- Count spec/plan character length as proxy — considered: too imprecise; character count doesn't correlate well with token usage across different AI models.

---

## Decision 6: Run Summary Artifact

**Decision**: A `run-summary.md` file is generated by the AI at the end of each `/speckit.plan` command run. It is stored in `specs/{feature}/run-summary.md` and overwritten on each plan run.

**Contents**:
- Selected execution mode
- Risk triggers detected (list)
- Artifacts generated (list)
- Estimated token count for the run
- Timestamp

**Rationale**: Making the Run Summary a file artifact (rather than console output only) enables downstream tooling to aggregate metrics across features and satisfies the traceability requirement in FR-009.

---

## Research Summary — All NEEDS CLARIFICATION Resolved

| Item | Resolution |
|---|---|
| Mode persistence mechanism | Two-layer: project default + per-feature file |
| Mode selection UX | AI-driven prompt in `specify.md` template |
| Risk trigger detection | grep-based scan in `common.sh` |
| AR/SEC gating logic | Template-level branching on mode + triggers |
| Token measurement | AI self-reporting in run-summary.md |
| Run summary artifact | `specs/{feature}/run-summary.md` |
