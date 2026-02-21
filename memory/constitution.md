# [PROJECT_NAME] Constitution

## Core Principles

### I. [PRINCIPLE_1_NAME]
<!-- Example: I. Library-First, I. Rego-First Threat Detection (NON-NEGOTIABLE) -->

[PRINCIPLE_1_DESCRIPTION]
<!-- Example: Every feature starts as a standalone library. Libraries must be self-contained, independently testable, and documented. Clear purpose required — no organizational-only libraries. -->

- [Requirement or guideline 1]
- [Requirement or guideline 2]

Rationale: [Why this principle exists and what it prevents or enables.]

### II. [PRINCIPLE_2_NAME]
<!-- Example: II. Crate-First Architecture, II. CLI Interface -->

[PRINCIPLE_2_DESCRIPTION]
<!-- Example: New functionality must live in the appropriate module with clear boundaries. Entry points are thin; logic lives in libraries. -->

```text
[Optional: Architecture diagram or structure]
```

Guidelines:
- [Guideline 1]
- [Guideline 2]

### III. [PRINCIPLE_3_NAME]
<!-- Example: III. Contract-First Design -->

[PRINCIPLE_3_DESCRIPTION]
<!-- Example: Define contracts before implementation. Define schemas before policies. Define interfaces before handlers. -->

- [Contract requirement 1]
- [Contract requirement 2]

### IV. [PRINCIPLE_4_NAME] (NON-NEGOTIABLE)
<!-- Example: IV. Test-First -->

[PRINCIPLE_4_DESCRIPTION]
<!-- Example: TDD mandatory — tests written → approved → fail → then implement. Red-Green-Refactor cycle strictly enforced. -->

- [Testing requirement 1]
- [Testing requirement 2]

### V. [PRINCIPLE_5_NAME] (NON-NEGOTIABLE)
<!-- Example: V. Complete Implementation -->

[PRINCIPLE_5_DESCRIPTION]
<!-- Example: All tasks in tasks.md MUST be completed before feature branch merges. -->

**Verification Checklist**:
- [ ] All tests written and passing
- [ ] Code reviewed and approved
- [ ] Performance benchmarks meet requirements
- [ ] Security review completed
- [ ] Breaking changes documented in CHANGELOG.md

Rationale: [Why this principle is non-negotiable.]

### VI. [PRINCIPLE_6_NAME]
<!-- Example: VI. Security-First, VI. Observability by Default -->

[PRINCIPLE_6_DESCRIPTION]

- [Requirement 1]
- [Requirement 2]

### VII. [PRINCIPLE_7_NAME]
<!-- Example: VII. Simplicity and Clarity -->

[PRINCIPLE_7_DESCRIPTION]

- [Guideline 1]
- [Guideline 2]

## Technology Stack

### Primary Stack

| Component | Technology | Version | Rationale |
|-----------|-----------|---------|-----------|
| **[Component 1]** | [Technology] | [Version] | [Why chosen] |
| **[Component 2]** | [Technology] | [Version] | [Why chosen] |
| **[Component 3]** | [Technology] | [Version] | [Why chosen] |

## Development Workflow

Quality gates are strict and must pass before merge:

- [Quality gate command 1]
- [Quality gate command 2]
- [Quality gate command 3]

### Pull Request Requirements

- All quality gates MUST pass.
- At least one approval required from code owner.
- All comments MUST be resolved before merge.
- Branch MUST be up-to-date with main.

## Governance

### Amendment Process

Changes to this constitution require team discussion and must follow the amendment process:

1. **Propose**: Document rationale and impact in GitHub issue.
2. **Discuss**: Discuss for at least 2 business days.
3. **Document**: Migration impact on existing code/practices.
4. **Update**: Update and version the constitution.
5. **Communicate**: Communicate changes to contributors.

### Versioning Policy

Constitution version follows semantic versioning (MAJOR.MINOR.PATCH):

- **MAJOR**: Backward incompatible changes (principle redefinition, new blocking requirements)
- **MINOR**: New principle added or materially expanded guidance
- **PATCH**: Clarifications, wording fixes, typo corrections

### Compliance Review

All pull requests MUST verify constitution compliance:

- Reviewers MUST check against principles
- New modules MUST document purpose and boundaries
- Breaking API changes MUST update documentation
- Security changes MUST include threat modeling

**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
<!-- Example: Version: 1.0.0 | Ratified: 2025-01-01 | Last Amended: 2025-01-01 -->
