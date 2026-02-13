# Code Style and Conventions

## Python (`src/specify_cli/__init__.py`)
- Single-file architecture — all CLI code in one file
- Uses typer for CLI framework, rich for output formatting
- `AGENT_CONFIG` dict keys must match actual CLI executable names
- `StepTracker` class for hierarchical progress rendering
- `select_with_arrows()` for cross-platform interactive selection

## Shell Scripts (`scripts/bash/`, `scripts/powershell/`)
- POSIX-compatible bash
- Full parity between bash and PowerShell variants
- Common utilities in `common.sh` / `common.ps1`

## Templates (`templates/`)
- Markdown with YAML frontmatter for slash commands
- Placeholders: `$ARGUMENTS`, `{SCRIPT}`, `{AGENT_SCRIPT}`, `__AGENT__`
- Most agents use Markdown; Gemini and Qwen use TOML format (auto-converted during release)

## General
- No automated test suite — testing is manual via CLI execution
- Markdown linting enforced via markdownlint-cli2
- Version tracked in `pyproject.toml`
- Changes to `__init__.py` require version bump in `pyproject.toml` + `CHANGELOG.md` entry
