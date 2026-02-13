# Spec Kit - Project Overview

## Purpose
Spec Kit is a toolkit for Spec-Driven Development (SDD) — a methodology where specifications are the primary artifact and code is generated from them.

## Components
1. **Specify CLI** (`src/specify_cli/__init__.py`) — Single-file Python CLI (~1,400 lines) that bootstraps projects with SDD templates
2. **Templates** (`templates/`) — Markdown templates for specs, plans, tasks, slash commands
3. **Shell Scripts** (`scripts/bash/`, `scripts/powershell/`) — Workflow automation with full parity between bash and PowerShell

## Tech Stack
- **Python 3.11+** with typer, rich, httpx, readchar, platformdirs, truststore
- **Bash (POSIX-compatible)** and **PowerShell 7+** scripts
- **Git 2.5+** (worktree support)
- **Build**: hatchling via `uv`
- **Linting**: markdownlint-cli2 for Markdown
- **Package manager**: uv

## Key Architecture Decisions
- The entire CLI lives in a single file (`__init__.py`) — this is intentional
- `AGENT_CONFIG` dict is the single source of truth for 18+ supported AI agents
- Templates use placeholder substitution (`$ARGUMENTS`, `{SCRIPT}`, `__AGENT__`)
- Release pipeline generates 36 ZIP files (18 agents × 2 script types)
- Project self-hosts its own SDD methodology (see `specs/`, `.specify/`)

## Version
Current: 0.0.23 (defined in `pyproject.toml`)
