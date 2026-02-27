# Chat P — Learning Notes

## AI Voice Agent Demo | Account Setup & API Keys

> **Project:** voice-agent-demo
> **Date:** February 2026
> **Phase:** Pre-Weekend — Foundation

---

## What We Decided

| Decision          | Choice                                  | Reason                                                                                                             |
| ----------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Project name      | `voice-agent-demo`                      | Generic enough to reuse across all client demos — no client name in the top-level project                          |
| Naming convention | kebab-case everywhere                   | GitHub, Railway, LiveKit, local folder all use `voice-agent-demo`. Qdrant collections use underscore: `rockgas_kb` |
| Voice ID          | Configured per customer in `config.yml` | One voice per customer — swap = edit one line, no code change                                                      |
| RAG citations     | `source_url` field in Qdrant payload    | Enables citation links in email summaries when content is scraped from websites                                    |

---

## The 7 Services — What Each One Does

Think of the voice agent as a **telephone relay race**. Each service is one runner passing the baton:

```
Caller speaks
    ↓
Twilio          — answers the phone (the front door — owns the NZ number)
    ↓
LiveKit         — converts phone call (SIP) to internet audio (WebRTC)
    ↓
Deepgram        — listens and turns speech into text (STT)
    ↓
OpenAI          — reads the text and decides what to say (GPT-4o-mini)
                — also converts knowledge base docs into searchable vectors
    ↓
Qdrant          — stores and searches the knowledge base
    ↓
ElevenLabs      — converts the text response into natural voice (TTS)
    ↓
Railway         — hosts the Python code that orchestrates all of the above
    ↓
Caller hears response
```

---

## Key Concepts Learned

### SIP vs WebRTC — Why We Need Both

- **SIP** (Session Initiation Protocol) — the language the telephone network speaks. Old, reliable, everywhere. Twilio speaks SIP.
- **WebRTC** — the language the modern internet speaks for real-time audio. What browsers and Python code use.
- **LiveKit** is the translator between them. Without it, your Python agent can't talk to the phone network.

### Why No ngrok

In older voice agent architectures, you'd need ngrok (a tunnel tool) to expose your MacBook to the internet so Twilio could reach it. LiveKit eliminates this completely: your agent **connects OUT** to LiveKit Cloud — no inbound ports needed on your MacBook. This means local dev and cloud (Railway) deploy work identically — same code, just different environment variables.

### STT vs TTS

- **STT** = Speech-to-Text. Caller speaks → Deepgram converts speech to text → GPT-4o-mini can read it.
- **TTS** = Text-to-Speech. GPT-4o-mini responds in text → ElevenLabs converts text to voice → Caller hears it.

### What is a Vector Database (Qdrant)?

When you feed documents into the system, each chunk of text is converted into a list of numbers called a **vector** — a mathematical representation of the text's meaning. When a caller asks a question, that question is also converted to a vector, and Qdrant finds the stored chunks whose vectors are closest in meaning. This is called **semantic search** — it finds meaning, not just matching keywords. Example: searching "how much does gas cost?" would still find a chunk about "LPG bottle pricing" even though no words match exactly.

### One Collection Per Customer — Data Isolation

Each customer gets their own isolated Qdrant collection:

- `rockgas_kb` — only RockGas content
- `mbie_kb` — only MBIE content

Zero risk of one customer's answers appearing in another's demo. Collection name is defined in each customer's `config.yml` — never hardcoded.

---

## Package Manager — Why uv Instead of pip

`pip` is Python's traditional package manager — it's been around since 2008 and works, but has limitations for professional projects.

`uv` is a modern replacement that solves three real problems:

| Problem with pip                   | How uv solves it                                       |
| ---------------------------------- | ------------------------------------------------------ |
| Slow installs                      | uv is 10-100x faster                                   |
| No lockfile by default             | uv generates `uv.lock` automatically                   |
| "Works on my machine" problems     | `uv.lock` guarantees identical packages everywhere     |
| Inconsistent between dev and cloud | Railway reads `uv.lock` to install exact same versions |

**The lockfile concept explained:** When you add a package, uv records the exact version in `uv.lock`. When Railway deploys your code, it reads that lockfile and installs the _exact same versions_ — not "latest compatible", but _exactly the same_. This is what makes "it works on my MacBook AND in production" reliable.

### How uv Was Installed

```bash
# Official recommended install method for Mac
curl -LsSf https://astral.sh/uv/install.sh | sh

# After install, activate in current terminal session
source $HOME/.local/bin/env

# Verify
uv --version
# Output: uv 0.10.6 (a91bcf268 2026-02-24)
```

**Why not Homebrew?** Homebrew also works (`brew install uv`), but the curl method is what the official uv docs recommend and what Railway's Docker environment is built to be compatible with. Fewer variables = fewer surprises.

### Key uv Commands You'll Use Throughout This Build

```bash
# Initialise a new project (creates pyproject.toml + .venv)
uv init --python 3.12

# Install all dependencies from pyproject.toml
uv sync

# Run any Python file (uses the project's .venv automatically)
uv run python agent.py dev

# Add a new package (updates pyproject.toml + uv.lock)
uv add some-package

# Add a dev-only package (not included in Railway deploy)
uv add --dev pytest
```

**Key rule:** Never type `pip install` in this project. Always `uv add` or `uv sync`. Cursor rules (Chat 2) will enforce this automatically.

---

## Environment Variables Reference

```bash
# Project
ACTIVE_CUSTOMER=rockgas           # which customer folder to load

# LiveKit
LIVEKIT_URL=wss://voice-agent-demo-4dm5as09.livekit.cloud
LIVEKIT_API_KEY=***
LIVEKIT_API_SECRET=***

# Twilio (used in dashboard config — not in .env directly)
TWILIO_ACCOUNT_SID=ACxxxxxxxxxx
TWILIO_AUTH_TOKEN=***
NZ_PHONE_NUMBER=+6498869807

# Speech
DEEPGRAM_API_KEY=***
ELEVEN_API_KEY=***

# LLM + Embeddings
OPENAI_API_KEY=***

# Vector DB
QDRANT_URL=https://xxxx.us-west-1-0.aws.cloud.qdrant.io
QDRANT_API_KEY=***

# Email
GMAIL_APP_PASSWORD=xxxx xxxx xxxx xxxx
```

> **Rule:** `.env` is NEVER committed to git. It contains real secrets.
> `.env.example` (with blank values) IS committed — it tells other developers (or future you) what keys are needed without exposing actual values.

---

## Naming Convention — Consistent Across All Surfaces

| Surface            | Convention | Example                 |
| ------------------ | ---------- | ----------------------- |
| GitHub repo        | kebab-case | `voice-agent-demo`      |
| Local folder       | kebab-case | `voice-agent-demo/`     |
| LiveKit project    | kebab-case | `voice-agent-demo`      |
| Railway service    | kebab-case | `voice-agent-demo`      |
| Qdrant cluster     | kebab-case | `voice-agent-demo`      |
| Qdrant collections | snake_case | `rockgas_kb`, `mbie_kb` |
| Customer folders   | snake_case | `customers/rockgas/`    |
| Python files       | snake_case | `agent.py`, `ingest.py` |

**Why two conventions?** Kebab-case (`words-with-hyphens`) is standard for URLs, service names, and folder names. Snake_case (`words_with_underscores`) is standard for Python files and database names. Following these conventions means your project looks professional and behaves predictably with every tool you use.

**Key principle:** Customer name only appears _inside_ the project — in the `customers/` folder and Qdrant collection names. Never in the top-level project name. This is what makes the platform reusable across any client.

---

## Design Decisions Locked In

### 1. Voice ID — Per Customer

```yaml
# customers/rockgas/config.yml
voice:
  elevenlabs_voice_id: "EXAVITQu4vr4xnSDxMaL" # ← different per customer
  elevenlabs_model: "eleven_turbo_v2_5"
```

Switching voice = edit one line in config. No code changes ever needed.

### 2. RAG Citations — Source URL in Qdrant Payload

When ingesting web content, each chunk stored in Qdrant will carry:

```python
payload = {
    'text': chunk,
    'source': filename,       # always populated
    'source_url': page_url,   # populated for web scraping, empty string for files
    'customer': customer_id
}
```

This enables the email summary tool to include citation links back to the original source pages — professional touch for the demo.

### 3. Collection Name — Config Driven

```yaml
# customers/rockgas/config.yml
qdrant:
  collection_name: rockgas_kb # ← fully config-driven, unique per customer
```

Nothing hardcoded. Every Qdrant call uses `cfg.qdrant.collection_name` loaded from this file.

---

## The Config-by-Design Principle

This is the most important architectural principle in this entire build. Ask yourself before hardcoding _anything_:

> **"Would this value need to change if I switched to the MBIE demo?"**

If yes → it goes in `customers/{name}/config.yml`. Never in the code.

This is what makes switching customers as simple as changing one environment variable (`ACTIVE_CUSTOMER=mbie`) with zero code changes.

---

## What Comes Next

| Chat   | What Gets Built                                                            |
| ------ | -------------------------------------------------------------------------- |
| Chat 1 | Project folder scaffolded with uv, Python 3.12, all dependencies declared  |
| Chat 2 | Cursor rules files — makes Cursor smarter about this project automatically |
| Chat 3 | LiveKit MCP server — gives Cursor live access to LiveKit API docs          |
| Chat 4 | `config.py` — loads customer config from YAML into typed Python objects    |
| Chat 5 | Customer data files — RockGas config, known users, orders                  |
| Chat 6 | `agent.py` — the voice agent itself, first working version                 |

---

## Handoff Into Chat 1

```
1. All 7 API keys saved — LIVEKIT_URL, LIVEKIT_API_KEY,
   LIVEKIT_API_SECRET, DEEPGRAM_API_KEY, ELEVEN_API_KEY,
   OPENAI_API_KEY, QDRANT_URL, QDRANT_API_KEY, plus Twilio
   SID/token saved separately for Chat 8

2. Twilio NZ number purchased — +6498869807, active in
   Twilio console

3. LiveKit project named voice-agent-demo — Qdrant cluster
   also named voice-agent-demo, Railway service will use
   same name in Chat 16. GitHub repo and local folder will
   both be voice-agent-demo. Customer data lives inside
   customers/rockgas/ — project name never references a
   specific client.
```

### Pre-Chat 1 Checklist

```
✅ uv installed — 0.10.6 (curl method, Mac aarch64)
✅ voice-agent-demo folder created at ~/code/voice-agent-demo
✅ All 7 API keys saved securely
✅ Twilio NZ number active — +6498869807
✅ LiveKit project named voice-agent-demo
✅ Qdrant cluster named voice-agent-demo
✅ Cursor open on voice-agent-demo folder
```

---

_voice-agent-demo | Chat P Learning Notes | February 2026_
