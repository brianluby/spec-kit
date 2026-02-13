# Suggested Commands

## Running the CLI
```bash
# Direct execution (fastest feedback)
python -m src.specify_cli --help
python -m src.specify_cli init demo-project --ai claude --ignore-agent-tools --script sh

# Editable install
uv venv && source .venv/bin/activate
uv pip install -e .
specify --help

# Run via uvx
uvx --from . specify init demo --ai copilot --ignore-agent-tools
```

## Dependencies
```bash
uv sync          # Install dependencies
uv build         # Build wheel
```

## Linting
```bash
npx markdownlint-cli2 "**/*.md"
```

## Testing Template Changes
```bash
# Generate release packages locally
./.github/workflows/scripts/create-release-packages.sh v1.0.0
# Output goes to .genreleases/
```

## Git
```bash
git status
git diff
git log --oneline -10
```

## System Utilities (macOS/Darwin)
```bash
ls, cd, grep, find, cat, head, tail
```
