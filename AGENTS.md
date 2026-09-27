# ZAG dev — Full-Stack AI Assistant

This repo is a **Claude Code skill**, not a running app. The app lives in
`assets/template/` and is copied out by a scaffolder to create new projects.

## Layout

- Skill entry point: `SKILL.md`
- Template (what gets copied): `assets/template/`
  - Backend: FastAPI in `assets/template/api/` (port 8000)
  - Frontend: Next.js in `assets/template/web/` (port 3000)
- Docs to read on demand: `references/*.md`
- Scripts: `scripts/scaffold.py`, `scripts/verify.sh`, `scripts/doctor.py`
- AI: Ollama at `localhost:11434` (model installed here: `llama3.2:1b`),
  with cloud fallback to Anthropic / OpenAI / Gemini / Grok / Meta when a
  key is set in `api/.env`

## Commands

Create a new project from the template (do **not** rebuild it by hand):

```bash
python scripts/scaffold.py --dest ./my-app --app-name "My App"
```

Run all quality gates (ruff, pytest, TypeScript, Next.js build):

```bash
bash scripts/verify.sh ./my-app          # or ./assets/template
```

Tests only:

```bash
cd assets/template/api && uv run pytest
```

Run the app locally (two terminals):

```bash
cd assets/template/api && uv sync && uv run fastapi dev app/main.py
cd assets/template/web && npm install && npm run dev
```

Check a running stack:

```bash
python scripts/doctor.py --api http://localhost:8000 --web http://localhost:3000
```

## Rules

- I am a beginner: explain changes simply.
- Never commit `.env` or API keys. Only `.env.example` files (with blank
  values) belong in git.
- Run the tests after every change.
- Branding lives in one place: `assets/template/web/lib/config.ts`
  (app name + user name).

## Gotchas that have already bitten us

- `scripts/scaffold.py` asserts on exact strings in the template (app name,
  API title, README heading, package names). If you rename any of them,
  update `scaffold.py` in the same change or scaffolding breaks.
- `next start` renames its process to `next-server`, so `pkill -f "next start"`
  silently misses it and you keep serving a stale build. Use `pkill -f next-server`.
- `next.config.ts` sets `output: "standalone"`, so `npm start` warns; the real
  deployment path is `node .next/standalone/server.js` (see `web/README.md`).
- Keep `web/lib/types.ts` in sync with `api/app/schemas.py`.
