# CME — Local chatbot prototype

Standalone LangGraph/LangChain proof-of-concept with a browser testing interface. **Never deploy this prototype to the public internet as-is.** The repository is public, and therefore contains no real knowledge, credentials, or visitor data.

## Architecture
- `app/main.py`: FastAPI, defensive input validation, persistent atomic quotas (SQLite), LangGraph retrieval -> answer/refusal workflow, LangChain model adapter.
- `web/index.html`: portable chat interface with sample questions and live quotas.
- `examples/approved.example.md`: public demonstration facts only. Your real knowledge document is kept **outside** this repository.
- `state/limits.sqlite3`: local runtime counters, excluded by gitignore.

## Install on Windows (PowerShell)

```powershell
git clone https://github.com/0rndm123/cme.git
cd cme
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
New-Item -ItemType Directory -Force ..\cme_private
Copy-Item examples\approved.example.md ..\cme_private\approved.md
python -m uvicorn app.main:app --host 127.0.0.1 --port 8765
```

Open **http://127.0.0.1:8765**. The demonstration document is for testing only; replace the file in `../cme_private/approved.md` with your reviewed Markdown. Every block needs a `## identifier` heading followed by one `PT:` and one `EN:` line. JSON `{"facts":[{"id":"...","pt":"...","en":"..."}]}` is also supported.

Leave `OPENAI_API_KEY` empty to test without paid model calls. For LLM answers, edit the **untracked** `.env` and set a model and your private key. The API key is only read server-side.

## Rate limits — rolling windows, atomically reserved

- 8 questions per session per 24h
- 3 per IP per 60s
- 15 per IP per 24h
- 50 total per 60s
- 500 total per 60min
- 30 model calls globally per 24h

SQLite counters survive app restarts. Since the local server runs on loopback, all local browsers share the same IP quota. Configure individual and global limits in `.env`. No financial hard cap is provided; the model provider's billing needs separate protection.

## API

- `GET /api/session`: creates HttpOnly SameSite=Strict cookie and returns counters/status.
- `POST /api/chat`: JSON `{"question":"...","language":"pt"}` with same-origin Origin and session cookie.

This is a local-only experimental security baseline; not a production security audit. The server rejects foreign Host/Origin values, and `uvicorn` must bind to `127.0.0.1`, never `0.0.0.0`. The metadata is not a robust public authentication method. The retrieval is basic keyword matching, and even approved-document prompting **cannot guarantee zero hallucinations**. Improve evidence verification, concurrent quota semantics and budget enforcement prior to launch.

## Future migration to Portfolio

Keep the frontend contract `/api/session` and `/api/chat`. Reuse LangGraph nodes and prompts, and adapt the hosting layer to Netlify Functions or an independent service. Replace SQLite with shared atomic Supabase/PostgreSQL quota handling; configure HTTPS Secure cookies, bot mitigation and budget caps. Do not move the private knowledge into the public build folder or source repository.
