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

- **SIP** (Session Initiation Protocol) — the language the telephone network speaks. Old, reliable, everywhere.
- **WebRTC** — the language the modern internet speaks for real-time audio. What browsers and Python code use.
- **LiveKit** is the translator between them. Without it, your Python agent can't talk to the phone network.

### Why No ngrok

In the old architecture, you'd need ngrok (a tunnel tool) to expose your MacBook to the internet so Twilio could reach it. LiveKit eliminates this: your agent **connects OUT** to LiveKit Cloud — no inbound ports needed on your MacBook. Local dev and cloud deploy work identically.

### STT vs TTS

- **STT** = Speech-to-Text. Caller speaks → Deepgram converts to text → GPT can read it.
- **TTS** = Text-to-Speech. GPT responds in text → ElevenLabs converts to voice → Caller hears it.

### What is a Vector Database (Qdrant)?

When you feed documents into the system, each chunk of text is converted into a list of numbers (a "vector") that represents its meaning mathematically. When a caller asks a question, that question is also converted to a vector, and Qdrant finds the chunks whose vectors are most similar. This is called **semantic search** — it finds meaning, not just matching keywords.

### One Collection Per Customer

Each customer (RockGas, MBIE, etc.) gets their own isolated Qdrant collection:

- `rockgas_kb` — only RockGas content
- `mbie_kb` — only MBIE content

Zero risk of one customer's answers appearing in another's demo.

### Why uv Instead of pip

`uv` is a modern Python package manager that is significantly faster than `pip` and uses a lockfile (`uv.lock`) to guarantee that the exact same package versions are installed on your MacBook AND on Railway. This prevents "works on my machine" problems.

---

## Environment Variables Reference

```bash
# Project
ACTIVE_CUSTOMER=rockgas           # which customer folder to load

# LiveKit
LIVEKIT_URL=wss://voice-agent-demo-4dm5as09.livekit.cloud
LIVEKIT_API_KEY=***
LIVEKIT_API_SECRET=***

# Twilio (used in dashboard config, not in .env directly)
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

> **Rule:** `.env` is NEVER committed to git. `.env.example` (with blank values) IS committed so others know what keys are needed.

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

**Key principle:** Customer name only appears _inside_ the project (in `customers/` folder and Qdrant collection names) — never in the top-level project name. This is what makes the platform reusable.

---

## Design Decisions Locked In

### 1. Voice ID — Per Customer

```yaml
# customers/rockgas/config.yml
voice:
  elevenlabs_voice_id: "EXAVITQu4vr4xnSDxMaL" # ← different per customer
  elevenlabs_model: "eleven_turbo_v2_5"
```

Switching voice = edit one line in config. No code changes.

### 2. RAG Citations — Source URL in Payload

When ingesting web content, each Qdrant chunk will store:

```python
payload = {
    'text': chunk,
    'source': filename,       # always populated
    'source_url': page_url,   # populated for web scraping, empty string for files
    'customer': customer_id
}
```

This enables the email summary tool to include citation links back to source pages.

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

1. All 7 API keys saved — LIVEKIT_URL, LIVEKIT_API_KEY, LIVEKIT_API_SECRET, DEEPGRAM_API_KEY, ELEVEN_API_KEY, OPENAI_API_KEY, QDRANT_URL, QDRANT_API_KEY, plus Twilio SID/token saved separately for Chat 8
2. Twilio NZ number purchased — +6498869807, active in Twilio console
3. LiveKit project named `voice-agent-demo` — Qdrant cluster also named `voice-agent-demo`, Railway service will use same name in Chat 16. GitHub repo and local folder will both be `voice-agent-demo`. Customer data lives inside `customers/rockgas/` — project name never references a specific client.

---

_voice-agent-demo | Chat P Learning Notes | February 2026_
