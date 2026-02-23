# Quickstart: Adaptive Execution Modes

**Branch**: `001-adaptive-execution-modes` | **Date**: 2026-02-21

## What Changed

Speckit now asks you to select an **execution mode** at the start of every new feature. Your choice controls how much depth the workflow applies throughout the spec-to-tasks lifecycle.

## Modes at a Glance

| Mode | Use When | Clarification Limit | AR & SEC |
|---|---|---|---|
| **Fast** | Low-risk features, prototypes, spikes | 2 questions max | Only if risk detected |
| **Balanced** | Standard features (default) | 4 questions max | Only if risk detected |
| **Detailed** | Security-sensitive, regulated, high-impact | 5 questions max | Always |

## How to Use

### Starting a New Feature

Run `/speckit.specify` as normal. After the feature is scaffolded, you will see:

```
Select execution mode for this feature:

  ● balanced (recommended - project default)
  ○ fast      - Faster first pass, fewer tokens, skip AR/SEC unless risk detected
  ○ detailed  - Full depth, always includes AR and SEC

Press Enter to confirm selection.
```

Confirm or change the selection. Your choice is saved and applies to all subsequent commands for this feature.

### Setting a Project Default

Add `defaultMode` to `.specify/config.json`:

```json
{
  "git_mode": "worktree",
  "worktree_strategy": "sibling",
  "defaultMode": "fast"
}
```

Valid values: `fast`, `balanced`, `detailed`. If omitted, `balanced` is used.

### When Risk Triggers Are Detected

If your spec contains risk-indicating keywords (e.g., "auth", "payment", "PII"), you will see a notification during `/speckit.plan`:

```
⚠️  Risk triggers detected in spec: [auth, payment]
Adding Architecture Review (AR) and Security Review (SEC) to this run.
Continuing with escalated artifact set...
```

No action required — the workflow continues automatically with AR and SEC included.

### Reading the Run Summary

After each `/speckit.plan` run, check `specs/{feature}/run-summary.md` for:
- Which mode was used
- What triggers (if any) fired
- Which artifacts were generated
- Estimated token count for the run

Use this file to compare token efficiency across features and validate SC-001/SC-002 targets.

## Common Scenarios

**Prototyping a new UI component (no auth, no data risk)**:
→ Choose `fast`. You get spec + plan in minimal tokens. No AR/SEC generated.

**Adding a new API endpoint that reads user data**:
→ Choose `balanced`. The word "user data" may trigger SEC; you'll be notified and SEC added automatically.

**Implementing payment processing**:
→ Choose `detailed` proactively, or use `balanced` — "payment" is a risk keyword and will trigger AR + SEC automatically.

**Enterprise compliance feature**:
→ Choose `detailed`. AR and SEC are always generated; full clarification depth ensures nothing is missed.
