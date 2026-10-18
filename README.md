# CME | Portable local chatbot prototype

**This repository contains only the reusable app.** No actual knowledge base, credentials, private logs or runtime state belong in this repository. Work locally only; do not deploy this prototype as-is to Netlify or any public host.

## Architecture

- `app/main.py`: local FastAPI service, retrieval and server-side model call, privacy and quota checks.
- `web/index.html`: replaceable chat UI, calling only `/api/session` and `/api/chat`.
- `examples/approved.example.md`: illustrative sample knowledge; not loaded by default.
- **External private folder** (`KNOWLEDGE_PATH`): approved facts kept outside this repository.
- `state/limits.sqlite3`: persistent SQLite counters, automatically created and gitignored.

The application reads the external knowledge file afresh per request. No endpoint serves the file. `GET /api/session` provides an HttpOnly cookie; POST requests require a valid same-origin Origin header. Server must bind ONLY to `127.0.0.1`.

## Start on Windows (PowerShell)

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
```

Create a different private folder outside the cme repository, for example `C:\Users\<user>\cme_private`. Copy `examples\approved.example.md` to it as `approved.md`, then set the absolute path in `.env`, e.g. `KNOWLEDGE_PATH=C:/Users/<user>/cme_private/approved.md`.

```powershell
python -m uvicorn app.main:app --host 127.0.0.1 --port 8765
```

Open `http://127.0.0.1:8765`. Leave `OPENAI_API_KEY` empty to test extractive mode without provider calls.

## Quotas

| Scope | Default | Window |
|---|---:|---|
| Session | 8 | 24h |
| IP | 3 | Rolling 60s |
| IP | 15 | Rolling 24h |
| Globally | 50 | Rolling 60s |
| Globally | 500 | Rolling 60min |
| Model calls globally | 30 | Rolling 24h |

SQLite atomically reserves quota before calling the model. HTTP 429 when exceeded. These are **not hard monetary caps**. Use separate provider billing controls. On a local PC, all visits appear to originate from loopback.

## Security and limitations

- Listen on loopback only; validate Host and Origin. Not ready for public deployment.
- HttpOnly SameSite Strict session cookie. HTTP localhost does not use Secure; enable for HTTPS.
- No client-controlled session IDs or forwarded headers for IP identity.
- No API secrets or private knowledge in this public repo.
- Approved facts and citations reduce hallucination, but do not guarantee factual correctness.
- Cap questions to 500 characters and responses to 300 completion tokens.
- Future public deployment requires stronger identity, privacy-preserving IP quotas, abuse mitigation and a shared atomic counter store.
- Do not commit `.env`, `state/` or private knowledge.

## Migration to Portfolio

Reuse the `/api/session`, `/api/chat` contracts and retrieval/prompt logic. Replace the local FastAPI host with a Netlify Functions adapter or separately hosted service, SQLite with atomic Supabase quotas, and configure secure HTTPS cookies and server-only secrets. Keep Portfolio production unchanged until tested.
