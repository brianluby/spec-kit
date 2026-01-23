# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Spec Kit** is a toolkit for Spec-Driven Development (SDD) - a methodology where specifications are the primary artifact and code is generated from them. The project consists of:

1. **Specify CLI** (`src/specify_cli/__init__.py`) - A Python CLI tool that bootstraps projects with SDD templates
2. **Templates** - Markdown templates for specs, plans, tasks, and slash commands
3. **Shell scripts** - Bash and PowerShell scripts for workflow automation

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

### Testing template changes

```bash
# Generate release packages locally
./.github/workflows/scripts/create-release-packages.sh v1.0.0

# Copy generated package to test project
cp -r .genreleases/sdd-copilot-package-sh/. <path-to-test-project>/
```

### Dependencies

```bash
uv sync                    # Install dependencies
uv build                   # Build wheel for distribution
```

## Architecture

### CLI Structure (`src/specify_cli/__init__.py`)

- **AGENT_CONFIG dict** - Single source of truth for all supported AI agents (Claude, Gemini, Copilot, Cursor, etc.). Keys must match actual CLI tool names (e.g., `cursor-agent` not `cursor`)
- **Commands**: `init`, `check`, `version`
- **StepTracker class** - Renders hierarchical progress using Rich Tree
- **select_with_arrows()** - Cross-platform interactive selection using readchar

### Template System

Templates in `templates/` are packaged into agent-specific ZIP files during release:
- `templates/commands/*.md` - Slash command definitions (analyze, checklist, clarify, constitution, implement, plan, specify, tasks)
- `templates/*-template.md` - Document templates for specs, plans, tasks, checklists
- Agent directories (`.claude/`, `.gemini/`, `.cursor/`, etc.) are generated per-agent with appropriate format (Markdown or TOML)

### Shell Scripts (`scripts/`)

- `bash/` and `powershell/` variants for cross-platform support
- Key scripts: `create-new-feature.sh`, `setup-plan.sh`, `update-agent-context.sh`
- Scripts are auto-selected based on OS unless `--script sh|ps` is specified

## Adding New Agent Support

1. Add entry to `AGENT_CONFIG` dict in `__init__.py` - use actual CLI tool name as key
2. Update `--ai` help text in `init()` command
3. Update README supported agents table
4. Update release scripts in `.github/workflows/scripts/`
5. Update context scripts in `scripts/bash/` and `scripts/powershell/`

See `AGENTS.md` for detailed integration guide.

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

## Recent Changes
- 001-git-worktrees: Added Bash (POSIX-compatible) + PowerShell 7+ + Git 2.5+ (worktree support), jq (optional, for JSON parsing)
