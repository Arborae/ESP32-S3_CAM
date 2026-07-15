# Contributing Guidelines

Thanks for wanting to contribute! ✨
We value **clarity, modularity and quality** — small, well-explained changes over
large sweeping ones.

## Requirements

- Docker installed and working
- Python 3.11 (only for server-side development — not required to use the container)
- Git

## Workflow

1. **Fork** the repo and create a branch:
   ```bash
   git checkout -b feat/short-description
   ```
2. Make small, focused changes. Keep commits clear.
3. Update the documentation if needed (README / CHANGELOG).
4. Run quick manual tests:
   - `docker compose up -d` in the `server` folder
   - check `http://localhost:12345/` and `/status`
5. Open a **Pull Request** with a description, logs and screenshots (if UI-related).

## Commit style

Use one of these prefixes:

- `feat:` new feature
- `fix:` bug fix
- `docs:` documentation
- `chore:` maintenance, non-functional refactor
- `perf:` performance
- `test:` tests

Examples:
- `feat: pinch → publish MQTT events on threshold`
- `fix: handle MQTT reconnect`

## Code guidelines

- **No secrets in plain text.** Use `.env` / environment variables.
- Keep defaults **neutral**, with placeholders.
- Avoid heavy dependencies unless strictly necessary.
- Follow the structure of the existing code.

## Reporting bugs

Open an issue using the **"Bug report"** template and include:
- steps to reproduce
- relevant (container) logs
- system info (OS/architecture, Docker version)
- a screenshot if helpful

## Features / ideas

Use the **"Feature request"** template. Explain why it's useful and how you
imagine it working.

## Security

Found a vulnerability? Please read [`SECURITY.md`](./SECURITY.md) — do **not**
open a public issue.

Happy hacking! 💚
