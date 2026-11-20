@AGENTS.md

## Claude Code specifics

- Decisions (architecture, style, testing, CI/CD, scope) are made in plan mode and need Bakari's sign-off before any files change.
- Area rules live in `.claude/rules/`. Each one loads automatically when you read or edit a file its `paths:` match.
- `.claude/settings.json` denies reading secret files. Don't work around it.
- How all of this fits together: [docs/agent-rules.md](docs/agent-rules.md).
