# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Spec Kit** is a toolkit for Spec-Driven Development (SDD) - a methodology where specifications are the primary artifact and code is generated from them. The project consists of:

1. **Specify CLI** (`src/specify_cli/__init__.py`) - A Python CLI tool that bootstraps projects with SDD templates
2. **Templates** (`templates/`) - Markdown templates for specs, plans, tasks, and slash commands
3. **Shell scripts** (`scripts/`) - Bash and PowerShell scripts for workflow automation

The project self-hosts: it uses its own SDD methodology (see `specs/` and `.specify/`).

## Development Commands

### Running the CLI locally

```bash
# Direct execution (fastest feedback)
python -m src.specify_cli --help
python -m src.specify_cli init demo-project --ai claude --ignore-agent-tools --script sh

# Editable install
uv venv && source .venv/bin/activate
uv pip install -e .
specify --help

# Run via uvx from local source
uvx --from . specify init demo --ai copilot --ignore-agent-tools
```

Use `--ignore-agent-tools` to skip CLI tool availability checks during local testing.

### Testing template changes

```bash
# Generate release packages locally (outputs to .genreleases/)
./.github/workflows/scripts/create-release-packages.sh v1.0.0

# Copy a generated package to a test project
cp -r .genreleases/sdd-copilot-package-sh/. <path-to-test-project>/
```

### Linting

```bash
# Markdown linting (also runs in CI via .github/workflows/lint.yml)
npx markdownlint-cli2 "**/*.md"
```

### Dependencies

```bash
uv sync                    # Install dependencies
uv build                   # Build wheel for distribution
```

## Architecture

### CLI (`src/specify_cli/__init__.py`)

The entire CLI lives in a single ~1,400-line file. This is intentional.

- **`AGENT_CONFIG` dict** - Single source of truth for all 18+ supported AI agents. Keys **must** match actual CLI executable names (e.g., `cursor-agent` not `cursor`). Each entry defines: `name`, `folder`, `install_url`, `requires_cli`.
- **Commands**: `init`, `check`, `version`
- **`StepTracker` class** - Renders hierarchical progress using Rich Tree
- **`select_with_arrows()`** - Cross-platform interactive selection using readchar
- **`download_template_from_github()`** / **`download_and_extract_template()`** - Fetches and extracts release ZIP from GitHub

### Template System

Templates in `templates/` are packaged into agent-specific ZIP files during release:
- `templates/commands/*.md` - Slash command definitions with YAML frontmatter containing script command variants
- `templates/*-template.md` - Document templates for specs, plans, tasks, checklists

**Placeholder substitution** during release packaging:
- `$ARGUMENTS` - User-provided arguments to the slash command
- `{SCRIPT}` / `{AGENT_SCRIPT}` - Replaced with agent-specific script paths (bash or PowerShell)
- `__AGENT__` - Replaced with the agent name

**Format conversion**: Most agents use Markdown commands, but Gemini and Qwen use TOML format. The release script handles this conversion.

### Shell Scripts (`scripts/`)

- `bash/` and `powershell/` variants with full parity
- Key scripts: `create-new-feature.sh` (branch/spec creation with smart naming), `setup-plan.sh`, `update-agent-context.sh` (generates agent files from plan.md)
- Scripts are auto-selected based on OS unless `--script sh|ps` is specified
- Supports git worktrees with fallback for `--no-git` workflows

### Release Pipeline

Triggered on push to main (paths: `memory/`, `scripts/`, `templates/`, `.github/workflows/`). Scripts live in `.github/workflows/scripts/`.

1. `get-next-version.sh` - Calculates next version
2. `check-release-exists.sh` - Prevents duplicate releases
3. `create-release-packages.sh` - Generates **36 ZIP files** (18 agents x 2 script types) in `.genreleases/`
4. `generate-release-notes.sh` - Extracts from `CHANGELOG.md`
5. `create-github-release.sh` - Uploads all ZIPs to GitHub release

Package naming: `spec-kit-template-{agent}-{script}-{version}.zip`

## Adding New Agent Support

1. Add entry to `AGENT_CONFIG` dict in `__init__.py` - use actual CLI tool name as key
2. Update `--ai` help text in `init()` command
3. Update README supported agents table
4. Update release scripts: `create-release-packages.sh` (`ALL_AGENTS` array), `create-github-release.sh` (ZIP file list)
5. Update context scripts: `scripts/bash/update-agent-context.sh` and `scripts/powershell/update-agent-context.ps1`
6. Update `AGENTS.md` supported agents table

See `AGENTS.md` for the full integration guide including format conventions and testing checklist.

## Key Workflows

### Spec-Driven Development Flow

1. `/speckit.constitution` - Establish project principles
2. `/speckit.specify` - Create feature specification from requirements
3. `/speckit.clarify` - (Optional) Structured Q&A to clarify ambiguities
4. `/speckit.plan` - Generate technical implementation plan
5. `/speckit.analyze` - (Optional) Cross-artifact consistency check
6. `/speckit.tasks` - Break plan into actionable tasks
7. `/speckit.implement` - Execute tasks

### Version Updates

Changes to `__init__.py` require:
- Version bump in `pyproject.toml`
- Entry in `CHANGELOG.md`

## Active Technologies
- Bash (POSIX-compatible) + PowerShell 7+ + Git 2.5+ (worktree support), jq (optional, for JSON parsing) (001-git-worktrees)
- JSON configuration file (`.specify/config.json`) (001-git-worktrees)
- Bash (POSIX-compatible, sh-compatible) + Markdown (command templates) + `scripts/bash/common.sh` (shared functions), `scripts/bash/check-prerequisites.sh` (JSON output layer), `templates/commands/` (AI workflow templates) (001-adaptive-execution-modes)
- `.specify/config.json` (project default mode), `specs/{feature}/.feature-config.json` (per-feature selected mode) (001-adaptive-execution-modes)

## Recent Changes
- 001-adaptive-execution-modes: Added adaptive execution modes (fast/balanced/detailed), detect_risk_triggers(), get_execution_mode(), read_feature_config_value() to common.sh, mode-aware clarify/plan templates, run-summary.md generation, .feature-config.json per-feature persistence
- 001-git-worktrees: Added Bash (POSIX-compatible) + PowerShell 7+ + Git 2.5+ (worktree support), jq (optional, for JSON parsing)
