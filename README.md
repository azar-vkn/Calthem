<h1 align="center">CALTHEM</h1>
<p align="center"><b>AI that calls, asks, listens, and collects.</b><br/>
Turn phone conversations into structured data.</p>

---

Calthem is an autonomous AI voice information-collection platform. Give it a
phone number **or** a spreadsheet of contacts plus a plain-English instruction
("*ask their name, department, project and team size*"). Calthem derives the
questions, calls each person (or talks through the browser), holds a natural
conversation, extracts structured JSON, shows it on a live dashboard, and
exports it to Excel / CSV / JSON.

It is built as a clean application layer on top of the open-source
[Bolna](https://github.com/bolna-ai/bolna) voice-agent platform — reusing its
ASR + LLM + TTS orchestration, telephony, and instruction→extraction-schema
engine rather than reinventing them.

## What's in this repo

```
calthem/
  backend/        FastAPI + SQLite — campaigns, contacts, calls, extraction,
                  results, export, live SSE events, Bolna webhook ingest.
  frontend/       Next.js + Tailwind — monochrome glassmorphic dashboard:
                  overview, single call, web voice call, bulk import,
                  live campaign dashboard, results + export.
  bolna_runtime/  Native launcher for the Bolna engine (no Docker, no Redis)
                  used for real phone + web voice calls.
README.md
```

> The upstream `bolna/` engine is **not** committed here (it's a third-party
> dependency). Clone it alongside `calthem/` for real voice — see below.

## Two ways to run

### 1. Demo / mock mode — zero setup, zero credentials
The whole product works with a **simulated caller**: Excel → queue → live
dashboard → results → export, all without telephony or API keys.

```bash
# backend
cd calthem/backend
python -m venv .venv && .venv/Scripts/activate      # (source .venv/bin/activate on macOS/Linux)
pip install -r requirements.txt
cp .env.sample .env                                 # defaults to TELEPHONY_ADAPTER=mock
uvicorn app.main:app --port 8090
```
```bash
# frontend (new terminal)
cd calthem/frontend
npm install
npm run dev                                          # http://localhost:3000
```
Open http://localhost:3000 → **Run Demo**.

### 2. Real voice — web call & phone call
Needs provider API keys (OpenAI + Deepgram + ElevenLabs) and the Bolna engine.
**No Docker required** — the engine runs natively.

```bash
# from the repo root, clone the Bolna engine next to calthem/
git clone https://github.com/bolna-ai/bolna.git
cd bolna && python -m venv .venv && .venv/Scripts/activate
pip install -e . && pip install imageio-ffmpeg
```
```bash
# start the Calthem Bolna engine launcher
cd calthem/bolna_runtime
cp .env.sample .env          # then put your OPENAI / DEEPGRAM / ELEVENLABS keys in .env
../../bolna/.venv/Scripts/python.exe server.py     # serves http://localhost:5001
```
Then start the backend and frontend as above. In the dashboard:
- **Web Call** — talk to Calthem through your microphone (or type). No phone number needed.
- **Phone call** — set `TELEPHONY_ADAPTER=bolna` and configure a telephony
  provider (a free Twilio/Plivo trial gives you a number that can call your
  verified test numbers) + a public tunnel for webhooks.

## Architecture

```
Next.js dashboard ──HTTP/SSE──► Calthem backend ──► Call adapter
 (glassmorphic UI)              (SQLite, queue,     ├─ mock   (simulated, default)
                                 extraction,        └─ bolna  (real)
                                 results, export)          │
        Web Call: browser mic ⇄ Bolna engine ws  ◄────────┤
                                                            ▼
                                               Bolna engine (native)
                                               ASR (Deepgram) + LLM (OpenAI)
                                               + TTS (ElevenLabs) + extraction
                                                            │ webhook
                                                            ▼
                                                   Calthem result ingest
```

The **adapter interface** keeps the caller swappable: `mock` needs nothing and
simulates the full call lifecycle; `bolna` places real calls. Everything else —
dynamic schema generation, the queue, results, export — is identical in both.

## Responsible use

Calthem always identifies itself as an AI assistant on every call. Only call
people who have agreed to receive calls. Secrets live in environment variables,
never in the frontend, and `.env` files are gitignored.

---

Built on [Bolna](https://github.com/bolna-ai/bolna) (MIT).
