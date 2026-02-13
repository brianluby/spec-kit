# Task Completion Checklist

When a task is completed, verify:

1. **Markdown linting**: `npx markdownlint-cli2 "**/*.md"` — fix any violations
2. **CLI still works**: `python -m src.specify_cli --help` — verify no import/syntax errors
3. **Version bump** (if `__init__.py` changed): Update `pyproject.toml` version and add `CHANGELOG.md` entry
4. **Agent parity** (if adding new agent): Update all locations listed in CLAUDE.md "Adding New Agent Support"
5. **Script parity** (if modifying bash scripts): Ensure PowerShell equivalent is also updated
