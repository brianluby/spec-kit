# Contract: Risk Trigger Detection

**Feature**: `001-adaptive-execution-modes` | **Date**: 2026-02-21

This contract defines how risk triggers are detected, reported, and acted upon.

---

## Detection Function: `detect_risk_triggers`

**Location**: `scripts/bash/common.sh`

**Signature**:
```bash
detect_risk_triggers <spec_file_path>
# Returns: space-separated list of matched trigger tokens (empty string if none)
# Exit code: 0 always (detection failure is non-fatal)
```

**Algorithm**:
1. Read spec file content (case-insensitive)
2. For each keyword in the catalog, test if it appears as a whole word or phrase in the spec text
3. Collect matched keywords (deduplicated)
4. Return as space-separated string

**Keyword catalog v1** (case-insensitive, whole-word/phrase match):
```
auth
authentication
authorization
payment
billing
pii
personal data
personal information
external api
third-party
third party
delete
destroy
drop
admin
compliance
regulation
gdpr
hipaa
encryption
secret
credential
token
password
```

---

## Exposure via `check-prerequisites.sh`

The prerequisites script exposes trigger detection results in its JSON output:

```json
{
  "FEATURE_SPEC": "/path/to/spec.md",
  "IMPL_PLAN": "/path/to/plan.md",
  "SPECS_DIR": "/path/to/feature",
  "BRANCH": "001-adaptive-execution-modes",
  "HAS_GIT": "true",
  "EXECUTION_MODE": "fast",
  "HAS_RISK_TRIGGERS": "true",
  "RISK_TRIGGERS": "auth payment"
}
```

**`EXECUTION_MODE`**: Read from `specs/{feature}/.feature-config.json`; defaults to `balanced` if file missing.

**`HAS_RISK_TRIGGERS`**: `"true"` or `"false"` string.

**`RISK_TRIGGERS`**: Space-separated list of matched keywords, or empty string.

---

## Escalation Behavior

When `HAS_RISK_TRIGGERS=true` in fast or balanced mode:

1. **Notification format** (shown to user before AR/SEC generation):

   ```
   ⚠️  Risk triggers detected in spec: [auth, payment]
   Adding Architecture Review (AR) and Security Review (SEC) to this run.
   Continuing with escalated artifact set...
   ```

2. AR generation proceeds via `/speckit.architecture` invocation
3. SEC generation proceeds via `/speckit.security` invocation
4. Both artifacts are listed in `run-summary.md` under `artifacts_generated`

---

## Invariants

- Detection always runs before any artifact gating decision
- Detection is idempotent (running multiple times on the same spec returns the same results)
- Detection does not modify the spec file
- An empty trigger list (`""`) means no escalation — treat as `HAS_RISK_TRIGGERS=false`
- Detailed mode ignores trigger detection for gating purposes (AR/SEC always generated)
