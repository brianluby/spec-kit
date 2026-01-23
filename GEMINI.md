# GEMINI.md

This file provides context and guidance for the Gemini CLI agent when working with the Spec Kit repository.

## Project Overview

**Spec Kit** is a toolkit for **Spec-Driven Development (SDD)** — a methodology where specifications are the primary artifact, and code is generated from them. The project structure includes:

1.  **Specify CLI** (`src/specify_cli/__init__.py`): A Python CLI tool that bootstraps projects with SDD templates.
2.  **Templates** (`templates/`): Markdown templates for specifications, plans, tasks, and slash commands.
3.  **Shell Scripts** (`scripts/`): Bash and PowerShell scripts for workflow automation.

## Building and Running

The project uses `uv` for package management and Python 3.11+.

### Setup

```bash
uv sync  # Install dependencies
```

### Running the CLI Locally

**Direct Execution (Fastest Feedback):**
```bash
python -m src.specify_cli --help
# Example: Initialize a demo project
python -m src.specify_cli init demo-project --ai gemini --ignore-agent-tools --script sh
```

**Run via uvx (from local source):**
```bash
uvx --from . specify init demo --ai gemini --ignore-agent-tools
```

### Building for Distribution

```bash
uv build  # Build wheel
```

### Testing Template Changes

To test changes to templates or slash commands:

1.  **Generate release packages locally:**
    ```bash
    ./.github/workflows/scripts/create-release-packages.sh v1.0.0
    ```
2.  **Copy the generated package to a test project:**
    ```bash
    # Example for shell-based package
    cp -r .genreleases/sdd-gemini-package-sh/. <path-to-test-project>/
    ```

## Development Conventions

-   **Code Style:** Follow existing Python conventions in `src/`.
-   **AI Disclosure:** If AI assistance is used for contributions, it **must** be disclosed in the PR description (see `CONTRIBUTING.md`).
-   **Architecture:**
    -   `src/specify_cli/__init__.py`: Contains `AGENT_CONFIG` (source of truth for agents), `StepTracker` (UI), and main command logic.
    -   `templates/`: Contains the markdown files that get copied to user projects.
    -   `scripts/`: Contains the logic executed by the slash commands in the user's project.
-   **Adding Agents:** See `AGENTS.md` for instructions on adding support for new AI agents.

## Key Workflows (Spec-Driven Development)

The Spec Kit workflow relies on specific slash commands available to the agent in the initialized project:

1.  `/speckit.constitution`: Establish project principles.
2.  `/speckit.specify`: Create feature specifications from requirements.
3.  `/speckit.clarify`: (Optional) Structured Q&A to clarify ambiguities.
4.  `/speckit.plan`: Generate technical implementation plans.
5.  `/speckit.analyze`: (Optional) Cross-artifact consistency check.
6.  `/speckit.tasks`: Break plan into actionable tasks.
7.  `/speckit.implement`: Execute tasks to generate code.

## Key Files

-   `pyproject.toml`: Python project configuration and dependencies.
-   `src/specify_cli/__init__.py`: Main entry point for the CLI.
-   `CONTRIBUTING.md`: Detailed contribution guidelines.
-   `CLAUDE.md`: Reference context for Claude Code (useful for consistency).
-   `AGENTS.md`: Documentation on supported agents.
