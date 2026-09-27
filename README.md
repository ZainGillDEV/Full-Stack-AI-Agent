# ZAG dev — Full-Stack AI Agent

A production-ready **AI chat assistant** (ChatGPT/Claude-style) that runs
**local models through Ollama first** and falls back to **cloud AI** when no
local model is installed or a local model fails.

This repo is a reusable **template + scaffolder**, so you can spin up a fresh,
branded project in one command instead of rebuilding it every time.

```
Full-Stack-AI-Agent/
├── SKILL.md               how to use / extend this system
├── assets/template/       the app that gets copied into new projects
│   ├── api/               FastAPI · Python 3.13 · uv · ruff · pytest
│   ├── web/               Next.js 16 · React 19 · TypeScript 7 · Tailwind 4 · shadcn/ui
│   └── docker-compose.yml api + web (+ optional Ollama profile)
├── references/            deep-dive docs (architecture, providers, deployment, …)
└── scripts/               scaffold.py · verify.sh · doctor.py
```

## Requirements

- Python 3.13 with [uv](https://docs.astral.sh/uv/)
- Node.js 20.9+
- [Ollama](https://ollama.com) (optional, but recommended for free local models)

## Quick start

### Option A — create a new project (recommended)

```bash
python scripts/scaffold.py --dest ./my-app --app-name "My App"
```

This copies the template, renames both packages and lockfiles, sets the app
name, and creates `api/.env` + `web/.env.local` for you.

### Option B — run the template directly

```bash
# 1. a local model (optional)
ollama pull llama3.2

# 2. backend  → http://localhost:8000/docs
cd assets/template/api
uv sync
cp .env.example .env              # add cloud API keys here if you have them
uv run fastapi dev app/main.py

# 3. frontend → http://localhost:3000   (new terminal)
cd assets/template/web
npm install
cp .env.example .env.local
npm run dev
```

Or with Docker:

```bash
cd assets/template
cp api/.env.example api/.env
docker compose up --build
```

## How model selection works

The model dropdown offers three groups:

| Option | Behaviour |
|---|---|
| **Auto** (default) | First installed Ollama model; if none, the first cloud provider with a key |
| **Local · Ollama** | Every chat model you've pulled, with size and parameter count |
| **Cloud (fallback)** | One sub-menu per provider whose API key is set, listing its live models |

If the chosen model fails **before** replying (Ollama stopped, model deleted,
key out of quota), the API automatically tries the next option and the reply
shows a note saying which model answered instead. Once text has started
streaming it never switches mid-answer, so replies never mix two models.

Supported cloud providers: **Anthropic, OpenAI, Google Gemini, xAI Grok,
Meta Llama**. A provider turns on as soon as its API key is set. Model lists are
fetched live, so new models appear without code changes. Set
`ALLOW_CLOUD_FALLBACK=false` to keep everything on the chosen model.

## Quality checks

```bash
bash scripts/verify.sh ./assets/template        # everything
bash scripts/verify.sh ./assets/template --api  # backend only
```

Runs `uv sync`, `ruff check`, `ruff format --check`, `pytest`, `npm ci`,
`npm run typecheck` and `npm run build`. Current status: **ruff clean,
21/21 tests passing, TypeScript 7 typecheck and production build green.**

Check a running stack:

```bash
python scripts/doctor.py --api http://localhost:8000 --web http://localhost:3000
```

## Documentation

| Topic | File |
|---|---|
| How the pieces fit together | `references/architecture.md` |
| HTTP / SSE API contract | `references/api-contract.md` |
| Adding or tuning an AI provider | `references/providers.md` |
| UI, rebranding, accessibility | `references/frontend.md` |
| Auth, database history, uploads, RAG | `references/extending.md` |
| Docker, env vars, reverse proxy, go-live | `references/deployment.md` |
| Something's broken | `references/troubleshooting.md` |

## Notes

- Branding (app name + user) lives in `assets/template/web/lib/config.ts`.
- Chat history is stored in the browser today. Add a database when you add
  user accounts — see `references/extending.md`.
- Never commit `.env` files or API keys; only `.env.example` (blank values)
  belongs in git.
- Add authentication and rate limiting before exposing the API publicly —
  cloud calls cost money.
