# Feature Specification: Adaptive Execution Modes

**Feature Branch**: `001-adaptive-execution-modes`  
**Created**: 2026-02-19  
**Status**: Draft  
**Input**: User description: "Adaptive Execution Modes (Fast vs Detailed) - Add --mode fast|balanced|detailed; fast mode merges steps, limits clarifications, and only creates AR/SEC when risk triggers are hit; Benefit: much faster first pass; Effort: Medium; KPI: time-to-first-working-feature, token/feature."

## Clarifications

### Session 2026-02-21

- Q: What do "AR" and "SEC" stand for in this context? → A: Architecture Review (AR) and Security Review (SEC) — dedicated artifact documents generated during planning.
- Q: What kinds of conditions qualify as risk triggers that escalate artifact requirements? → A: Keyword/signal detection in the feature spec text (e.g., "auth", "payment", "PII", "external API", "delete/destroy").
- Q: How does a user specify the execution mode for a feature run? → A: Interactive prompt at feature start, with a per-project config default (in `.specify/config.json`) that can be overridden per run.
- Q: When risk triggers are detected during a fast mode run, what happens? → A: Pause and notify — inform the user which triggers fired, then continue with AR/SEC artifacts included in the output.
- Q: How should "time-to-first-working-feature" be measured for SC-001 and SC-002? → A: Token count per feature run, tracked in the Run Summary artifact.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Accelerate low-risk feature setup (Priority: P1)

As a product engineer, I want a fast execution mode that compresses the spec workflow for low-risk features so I can get to a working implementation plan quickly with fewer tokens.

**Why this priority**: This delivers the core promised value of the feature: faster first pass speed and lower token usage.

**Independent Test**: Start a new low-risk feature in fast mode and verify the workflow completes with the reduced artifact set, fewer clarification steps, and a valid plan for implementation.

**Acceptance Scenarios**:

1. **Given** a new feature with no high-risk characteristics, **When** the user selects fast mode, **Then** the workflow produces only essential artifacts and completes without mandatory AR/SEC creation.
2. **Given** a new feature in fast mode, **When** clarification is needed, **Then** the system asks only high-impact questions and limits clarification rounds according to fast mode policy.

---

### User Story 2 - Keep a balanced default workflow (Priority: P2)

As a product manager, I want a balanced mode that preserves quality while reducing unnecessary depth so teams can move quickly without losing core planning confidence.

**Why this priority**: Balanced mode will likely become the default for most features and needs to protect quality while still improving efficiency.

**Independent Test**: Run the same feature in balanced mode and verify it generates standard planning artifacts with targeted depth and selective AR/SEC based on risk.

**Acceptance Scenarios**:

1. **Given** a medium-complexity feature, **When** balanced mode is selected, **Then** the system performs normal spec and planning steps with moderate clarification depth.
2. **Given** a balanced mode run that contains a security or compliance trigger, **When** risk is evaluated, **Then** AR/SEC artifacts are included and linked to the feature output.

---

### User Story 3 - Enforce full rigor when needed (Priority: P3)

As an engineering lead, I want a detailed mode that runs full-depth analysis and artifact generation so high-risk or high-importance work has maximum decision quality and traceability.

**Why this priority**: Detailed mode is essential for regulated, security-sensitive, or high-impact features where completeness is more important than speed.

**Independent Test**: Run a high-risk feature in detailed mode and verify full clarification depth, AR and SEC generation, and complete traceability into planning outputs.

**Acceptance Scenarios**:

1. **Given** a feature run in detailed mode, **When** the workflow executes, **Then** full-depth requirements, architecture, and security artifacts are generated before implementation planning.
2. **Given** a team chooses detailed mode, **When** execution finishes, **Then** the output includes explicit rationale for decisions and complete readiness for downstream task generation.

---

### Edge Cases

- What happens when an unsupported mode value is provided at feature start?
- How does the system resolve conflicts when a user-selected mode differs from project default mode? *(Resolved: the interactive prompt pre-selects the config default; the user's confirmation overrides it — no conflict, user choice always wins.)*
- What happens when fast mode is selected but high-risk triggers are detected midway through the workflow? *(Resolved: system pauses, notifies the user listing each detected trigger, then proceeds with AR/SEC artifacts added — no user action required.)*
- How does the system behave when risk signals are ambiguous or partially available? *(Resolved: any keyword match — even partial or in an unexpected context — is treated as a positive trigger (conservative approach). If the spec file is unreadable or the scan fails, the system treats this as triggered and includes AR/SEC as a safety fallback.)*
- What happens when a user switches mode after artifacts have already been generated for a feature? *(Resolved: mode is locked once plan artifacts are generated. To change mode, the user must re-run `/speckit.specify` which resets `.feature-config.json` and begins a fresh workflow. Existing artifacts are not automatically deleted — the user must manage cleanup manually.)*

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow users to select one execution mode per feature run (fast, balanced, or detailed) via an interactive prompt at feature start. If a project default mode is configured in `.specify/config.json`, it MUST be pre-selected as the default choice, with the user able to override it before confirming.
- **FR-002**: The system MUST apply mode-specific workflow depth rules consistently across specification and planning activities.
- **FR-003**: In fast mode, the system MUST minimize non-essential workflow steps and limit clarification to a maximum of 2 questions per run, selecting only the highest-impact decisions by an impact-times-uncertainty heuristic. Specifically, fast mode: (a) merges the Summary and Technical Context plan sections into a single condensed block, (b) omits the PRD cross-reference section from plan output, (c) skips AR and SEC unless risk triggers are detected (see FR-004), (d) limits clarification to 2 questions (see above).
- **FR-004**: In fast mode, the system MUST skip Architecture Review (AR) and Security Review (SEC) artifacts by default unless risk triggers are detected.
- **FR-005**: The system MUST evaluate each feature run for predefined risk triggers by scanning the feature spec text for risk-indicating keywords (e.g., "auth", "payment", "PII", "external API", "delete", "destroy") and escalate artifact requirements when triggers are detected.
- **FR-006**: In balanced mode, the system MUST produce standard planning outputs with a clarification limit of 4 questions per run and risk-based AR/SEC inclusion (same escalation logic as fast mode).
- **FR-007**: In detailed mode, the system MUST require full-depth analysis and include both architecture and security reviews.
- **FR-008**: The system MUST provide a clear run summary showing selected mode, triggered safeguards, and artifacts produced.
- **FR-009**: The system MUST preserve traceability from mode selection and risk decisions into run-summary.md, which records the selected mode, mode source, detected risk triggers, escalation status, and the list of artifacts generated during the run.
- **FR-010**: When risk triggers are detected during a fast or balanced mode run, the system MUST pause, display a notification listing each detected trigger keyword and the artifact(s) being added as a result, then continue the workflow with those artifacts included. The notification MUST appear before any escalated artifact generation begins.

### Assumptions and Dependencies

- Teams use one dominant mode per feature run rather than mixing modes within a single phase.
- The project default mode is stored in `.specify/config.json` under a `defaultMode` key; if absent, balanced is assumed.
- A shared risk trigger catalog exists and is maintained by product and engineering governance.
- Existing quality checks remain available regardless of selected mode.
- Historical baseline metrics are available to compare mode outcomes (time, token usage, and quality).
- Features created before this change have no `.feature-config.json`; the system falls back to balanced mode with a notice recommending the user re-run `/speckit.specify` to set a mode explicitly.

### Key Entities *(include if feature involves data)*

- **Execution Mode**: The selected depth profile for a feature run, including expected workflow behavior and output requirements.
- **Risk Trigger**: A condition indicating elevated business, security, or compliance risk that requires deeper review. Triggers are detected by scanning the feature spec text for risk-indicating keywords. The canonical keyword catalog is maintained in `contracts/risk-triggers.md` (single source of truth). Examples include: "auth", "payment", "PII", "external API", "delete", "compliance", "encryption".
- **Artifact Policy**: Rules that define which documents are required, optional, or skipped for each mode and risk level.
- **Run Summary**: A structured record of selected mode, triggered decisions, generated artifacts, and key metrics for auditability. MUST include total token count for the run to support SC-001 and SC-002 measurement.
- **Architecture Review (AR)**: A structured document capturing architectural decisions, component design, and integration considerations for the feature.
- **Security Review (SEC)**: A structured document assessing threat model, security controls, and compliance requirements relevant to the feature.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Fast mode reduces median token count per feature run by at least 40% compared to detailed mode, measured via the Run Summary artifact across a representative sample of at least 3 runs per mode. Baseline: run a representative feature through detailed mode first and record token count in run-summary.md; then repeat the same feature in fast mode and compare. Token counts are AI self-reported; validation is by human review of at least one run-pair.
- **SC-002**: Fast mode reduces median token count per feature run by at least 35% compared to balanced mode, measured via the same Run Summary comparison approach as SC-001. Validation method is identical: human review of at least one run-pair across modes.
- **SC-003**: At least 95% of runs generate artifact sets that match the applicable mode and risk policy.
- **SC-004**: Every execution run clearly communicates mode behavior at each decision point: (a) mode selection prompt shows all options with default highlighted, (b) risk trigger notification lists each detected keyword before escalation, (c) run-summary.md records mode, triggers, and artifacts produced. Validation: the end-to-end test (T019) confirms all three communication points are present and accurate.
- **SC-005**: Fewer than 5% of high-risk feature runs proceed without the required architecture and security artifacts.
