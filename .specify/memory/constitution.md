<!--
SYNC IMPACT REPORT
==================
Version Change: [UNSPECIFIED] → 1.0.0 (Initial ratification)
Modified Principles: N/A (initial ratification)
Added Sections:
  - Core Principles (5 principles)
  - Development Standards
  - Governance
Removed Sections: N/A
Templates Updated:
  ✅ plan-template.md - Constitution Check section references
  ✅ spec-template.md - No changes needed
  ✅ tasks-template.md - No changes needed
  ℹ️ agent-file-template.md - No references to principles
  ℹ️ checklist-template.md - No references to principles
Follow-up TODOs:
  - TODO(RATIFICATION_DATE): Original ratification date unknown - needs investigation
  - Consider adding command file templates to .specify/templates/commands/ for reference
-->

# Spec Kit Constitution

## Core Principles

### Agent Agnosticism

Spec Kit MUST support multiple AI coding agents without bias toward any specific vendor or platform. All generated templates, scripts, and documentation MUST work across all supported agents (Claude, Gemini, Cursor, etc.). Agent-specific configurations MUST be isolated to the AGENT_CONFIG metadata and generated command files, with no hardcoded agent preferences in core logic.

**Rationale**: Developer teams have diverse tool preferences and requirements. Agent agnosticism ensures Spec Kit remains useful as new agents emerge and prevents vendor lock-in.

### Template-Driven Architecture

All Spec Kit functionality MUST be implemented through templated files (Markdown, TOML) that are generated at project initialization time. Core logic MUST NOT contain hardcoded project structures, boilerplate code, or agent-specific formats. Templates MUST be the single source of truth for project scaffolding.

**Rationale**: Templating ensures consistency across projects, enables easy updates by modifying templates rather than code, and allows teams to customize their project structure by editing templates.

### Consistent Project Structure

All projects initialized with Spec Kit MUST follow a uniform directory structure: `.specify/` for framework artifacts, `specs/` for feature specifications, and agent-specific directories (`.claude/`, `.gemini/`, etc.) for AI agent configurations. Deviations from this structure MUST be justified and documented.

**Rationale**: Consistency enables developers to move between Spec Kit projects seamlessly, reduces cognitive load, and supports automated tooling that expects standard paths.

### Backward Compatibility

Changes to Spec Kit MUST maintain backward compatibility with existing projects. Breaking changes to directory structures, template formats, or command invocations MUST be avoided or accompanied by migration paths. Agent additions MUST NOT require changes to existing agent configurations.

**Rationale**: Users should be able to upgrade Spec Kit without disrupting their existing projects and workflows. Backward compatibility builds trust and reduces upgrade friction.

### CLI Excellence

The Specify CLI tool MUST follow command-line interface best practices: clear help text, sensible defaults, predictable exit codes, and standard flag conventions (`--flag`, `-f`). Error messages MUST be actionable and suggest solutions. CLI behavior MUST be consistent across platforms (Linux, macOS, Windows).

**Rationale**: A well-designed CLI reduces documentation burden, improves user experience, and enables reliable automation and scripting.

## Development Standards

### Version Management

Changes to `__init__.py` for Specify CLI MUST trigger a semantic version bump in `pyproject.toml` and a corresponding entry in `CHANGELOG.md`. Version increments MUST follow: MAJOR for breaking changes, MINOR for new features, PATCH for bug fixes.

### Code Quality

All Python code MUST pass linting and type checking before being committed. Dependencies MUST be explicitly declared in `pyproject.toml` with minimum version requirements. No dependencies on development-only packages in production code.

### Documentation

All new agents added to AGENT_CONFIG MUST include:
- Complete step-by-step integration guide in AGENTS.md
- Updated README.md supported agents table
- Updated GitHub release and packaging scripts
- Documentation in both bash and PowerShell update scripts

### Testing

CLI commands MUST be tested across supported platforms. Agent-specific command files MUST be validated for correct syntax and placeholder replacement. Template generation MUST be tested with each supported agent type.

## Governance

### Amendment Process

Constitution amendments require:
1. Documented rationale and impact analysis
2. Updates to all dependent templates (plan, spec, tasks, etc.)
3. Validation that existing projects remain compatible
4. Version increment (MAJOR for principle removal/redefinition, MINOR for additions)
5. Entry in Sync Impact Report at top of constitution.md

### Versioning Policy

CONSTITUTION_VERSION follows semantic versioning:
- MAJOR: Removal or fundamental redefinition of existing principles
- MINOR: Addition of new principles or materially expanded guidance
- PATCH: Clarifications, wording improvements, non-semantic refinements

### Compliance Review

All pull requests MUST verify compliance with constitution principles:
- Agent agnosticism: No agent-specific hardcoding in core logic
- Template-driven: No boilerplate in Python code, use templates instead
- Consistency: Directory structures follow documented patterns
- Compatibility: No breaking changes without migration paths
- CLI standards: Clear help, predictable behavior, cross-platform support

Complexity violations MUST be justified in Complexity Tracking table in plan.md. Use AGENTS.md as guidance for runtime development decisions.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): Original ratification date unknown - needs project history investigation | **Last Amended**: 2026-01-15
