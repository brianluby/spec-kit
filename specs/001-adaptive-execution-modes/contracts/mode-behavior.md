# Contract: Mode-Specific Workflow Behavior

**Feature**: `001-adaptive-execution-modes` | **Date**: 2026-02-21

This contract defines the behavioral obligations of each execution mode across all speckit command templates. Any deviation from this contract is a bug.

---

## Mode: `fast`

### `/speckit.specify`
- Present mode selection prompt with `balanced` as default (or project `defaultMode` from config)
- Write selection to `.feature-config.json`
- Generate spec as normal

### `/speckit.clarify`
- Read mode from `.feature-config.json`
- **Maximum 2 clarification questions** (down from 5)
- Question selection criterion: impact * uncertainty heuristic, top 2 only
- If 0–1 ambiguities found: proceed without forcing 2 questions
- Scan spec for risk triggers BEFORE starting questions; if triggers found:
  - Notify user: "Risk triggers detected: [list]. AR and SEC artifacts will be added to your plan."
  - Continue clarification within the 2-question limit

### `/speckit.plan`
- Read mode and risk trigger results from `.feature-config.json` / `check-prerequisites.sh`
- **Skip AR** unless `HAS_RISK_TRIGGERS=true`
- **Skip SEC** unless `HAS_RISK_TRIGGERS=true`
- Condense plan depth: skip PRD section, merge Technical Context and Summary
- If risk triggers present: notify user, generate AR + SEC before plan
- Always generate `run-summary.md` at end of command

### `/speckit.tasks`
- No mode-specific behavior (tasks are always derived from plan.md; depth is already baked in)

---

## Mode: `balanced`

### `/speckit.specify`
- Same as fast mode selection flow; `balanced` is the recommended default

### `/speckit.clarify`
- **Maximum 4 clarification questions**
- Full question taxonomy scan
- Risk trigger scan: same as fast mode notification behavior

### `/speckit.plan`
- **Skip AR** unless `HAS_RISK_TRIGGERS=true`
- **Skip SEC** unless `HAS_RISK_TRIGGERS=true`
- Full plan depth (Technical Context, all sections)
- If risk triggers present: notify user, generate AR + SEC before plan
- Always generate `run-summary.md`

### `/speckit.tasks`
- No mode-specific behavior

---

## Mode: `detailed`

### `/speckit.specify`
- Same selection flow

### `/speckit.clarify`
- **Maximum 5 clarification questions** (standard behavior, no change)

### `/speckit.plan`
- **Always generate AR** before plan (unconditionally)
- **Always generate SEC** before plan (unconditionally)
- Full plan depth
- Always generate `run-summary.md`

### `/speckit.tasks`
- No mode-specific behavior

---

## Cross-Mode Invariants

1. `run-summary.md` is generated at the end of every `/speckit.plan` run regardless of mode.
2. `plan.md` is always generated regardless of mode.
3. Risk trigger detection runs in every mode; only the response differs.
4. The notification shown on trigger detection is identical across modes: it lists the matched keywords and the artifact(s) being added.
5. After notification, the workflow continues automatically — no user action required.
6. An invalid mode value (not `fast`/`balanced`/`detailed`) causes an immediate error with remediation guidance.

---

## Error Cases

| Condition | Response |
|---|---|
| `.feature-config.json` missing | Treat as `balanced` (fallback), warn user to re-run `/speckit.specify` |
| `mode` key missing from config | Treat as `balanced` (fallback) |
| Invalid `mode` value | ERROR: "Unknown execution mode '{value}'. Valid values: fast, balanced, detailed. Re-run /speckit.specify to reset." |
| Spec file missing during trigger scan | Skip trigger scan, emit warning, continue |
