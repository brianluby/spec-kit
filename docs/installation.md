# Installation Guide

## Prerequisites

- **Linux/macOS** (or Windows; PowerShell scripts now supported without WSL)
- AI coding agent: [Claude Code](https://www.anthropic.com/claude-code), [GitHub Copilot](https://code.visualstudio.com/), [Codebuddy CLI](https://www.codebuddy.ai/cli) or [Gemini CLI](https://github.com/google-gemini/gemini-cli)
- [uv](https://docs.astral.sh/uv/) for package management
- [Python 3.11+](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)

## Installation

### Initialize a New Project

The easiest way to get started is to initialize a new project:

```bash
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <PROJECT_NAME>
```

Or initialize in the current directory:

```bash
uvx --from git+https://github.com/brianluby/spec-kit.git specify init .
# or use the --here flag
uvx --from git+https://github.com/brianluby/spec-kit.git specify init --here
```

### Specify AI Agent

You can proactively specify your AI agent during initialization:

```bash
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <project_name> --ai claude
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <project_name> --ai gemini
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <project_name> --ai copilot
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <project_name> --ai codebuddy
```

### Specify Script Type (Shell vs PowerShell)

All automation scripts now have both Bash (`.sh`) and PowerShell (`.ps1`) variants.

Auto behavior:

- Windows default: `ps`
- Other OS default: `sh`
- Interactive mode: you'll be prompted unless you pass `--script`

Force a specific script type:

```bash
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <project_name> --script sh
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <project_name> --script ps
```

Only the selected script variant is copied into your project under `.specify/scripts/<bash|powershell>`.

### Git Workflow Mode (Branch vs Worktree)

By default, Spec Kit creates features on branches. You can opt into worktree mode to get a parallel working directory per feature:

```bash
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <project_name> --git-mode worktree --worktree-strategy sibling
```

Worktree placement strategies:

- `sibling`: worktrees next to your repo
- `nested`: worktrees under `.worktrees/` inside the repo
- `custom`: a custom absolute path (use `--worktree-path`)

You can change the mode later using `.specify/scripts/<bash|powershell>/configure-worktree.*`, which updates `.specify/config.json`.

### Ignore Agent Tools Check

If you prefer to get the templates without checking for the right tools:

```bash
uvx --from git+https://github.com/brianluby/spec-kit.git specify init <project_name> --ai claude --ignore-agent-tools
```

## Verification

After initialization, you should see the following commands available in your AI agent:

- `/speckit.specify` - Create specifications
- `/speckit.plan` - Generate implementation plans  
- `/speckit.tasks` - Break down into actionable tasks

The `.specify/scripts/<bash|powershell>` directory will contain the scripts for your selected shell variant.

## Troubleshooting

### Git Credential Manager on Linux

If you're having issues with Git authentication on Linux, you can install Git Credential Manager:

```bash
#!/usr/bin/env bash
set -e
echo "Downloading Git Credential Manager v2.6.1..."
wget https://github.com/git-ecosystem/git-credential-manager/releases/download/v2.6.1/gcm-linux_amd64.2.6.1.deb
echo "Installing Git Credential Manager..."
sudo dpkg -i gcm-linux_amd64.2.6.1.deb
echo "Configuring Git to use GCM..."
git config --global credential.helper manager
echo "Cleaning up..."
rm gcm-linux_amd64.2.6.1.deb
```
