# Chat 3 — Learning Notes
## AI Voice Agent Demo | LiveKit MCP Server Setup

> **Project:** voice-agent-demo
> **Date:** February 2026
> **Phase:** Day 1 — Foundation
> **Status:** ✅ Complete

---

## Table of Contents
1. [What We Built](#what-we-built)
2. [The Big Picture — What MCP Is and Why It Matters](#the-big-picture)
3. [MCP Deep Dive — How It Actually Works](#mcp-deep-dive)
4. [What We Installed](#what-we-installed)
5. [AGENTS.md Update — Why It Matters](#agentsmd-update)
6. [The Verification Test — What It Proved](#the-verification-test)
7. [Key LiveKit 1.x Patterns — Learned From Live Docs](#key-livekit-patterns)
8. [How To Do It Manually](#how-to-do-it-manually)
9. [Verify It Worked](#verify-it-worked)
10. [Cursor Upskilling — MCP and the @ System](#cursor-upskilling)
11. [Handoff Into Chat 4](#handoff-into-chat-4)

---

## What We Built

No code files this chat. Instead we wired Cursor to a live data source — the official LiveKit documentation server. From now on, every time Cursor writes livekit-agents code, it reads the actual current API docs in real time rather than relying on training data that may be months out of date.

**Changes made:**
- LiveKit Docs MCP server installed in Cursor (7 tools, 3 resources enabled)
- `AGENTS.md` updated with LiveKit documentation instructions
- Git committed and pushed

**End state after Chat 3:**
```
voice-agent-demo/
├── .cursor/
│   └── rules/
│       ├── core.mdc        ✅ always loaded
│       ├── livekit.mdc     ✅ loads for agent.py, tools.py
│       └── config.mdc      ✅ loads for customers/**
├── AGENTS.md               ✅ updated with LiveKit MCP instructions
├── customers/rockgas/docs/
├── tests/
├── pyproject.toml
└── uv.lock
```

**Cursor now has three layers of intelligence working together:**
1. **Rules files** — standing conventions (Chat 2)
2. **AGENTS.md** — project briefing loaded every session (Chat 1, updated Chat 3)
3. **MCP server** — live API documentation fetched on demand (Chat 3)

---

## The Big Picture — What MCP Is and Why It Matters

### The Problem MCP Solves

Every AI model has a training cutoff date. Cursor's Claude Sonnet was trained on data up to a certain point in time. The livekit-agents library has been evolving rapidly — version 0.x used completely different patterns from 1.x. Without MCP, when Cursor writes livekit-agents code, it's working from memory that could be many months out of date.

**The risk without MCP:**
- Cursor writes `VoicePipelineAgent` (old 0.x) instead of `AgentSession + Agent` (1.x)
- Cursor uses `@llm.ai_callable` instead of `@function_tool`
- Cursor references methods that no longer exist or have been renamed
- Code looks plausible but fails at runtime

**The solution with MCP:**
Cursor fetches the current docs in real time before writing code. Not memory. Not guessing. Actual current documentation. When Cursor told us "rendered live just now at 2026-02-28" — that's exactly what was happening.

### The Analogy

Think of an architect designing a building. Without MCP, they're working from a copy of the building code they read six months ago — and building codes change. With MCP, they have a live connection to the current official building code database that updates in real time. Same architect, dramatically more accurate work.

### What MCP Actually Is

**MCP = Model Context Protocol**

An open standard created by Anthropic that defines how AI assistants connect to external tools and data sources. It's a standardised "pipe" between an AI and a data source. Any service can build an MCP server — LiveKit built one for their docs, and more services are adding them constantly.

**Key insight:** MCP isn't magic. It's just a structured way for Cursor to make an API call to an external server and get data back in a format it understands. LiveKit's MCP server at `https://docs.livekit.io/mcp` responds to queries with current documentation content.

---

## MCP Deep Dive — How It Actually Works

### The Request Flow

When you type `@livekit-docs` in a Cursor prompt:

```
You type @livekit-docs in Cursor prompt
         ↓
Cursor identifies the MCP server reference
         ↓
Cursor sends a query to https://docs.livekit.io/mcp
         ↓
LiveKit's MCP server fetches relevant doc pages
         ↓
Returns structured content back to Cursor
         ↓
Cursor includes that content as context for its response
         ↓
Cursor writes code based on current docs, not training memory
```

### What "7 Tools, 3 Resources Enabled" Means

When you saw this in Cursor Settings → Tools & MCP:

**Tools (7)** — things the MCP server can *do*:
- Search documentation
- Browse specific doc pages
- Search GitHub repositories (livekit + livekit-examples orgs)
- Access changelogs for any LiveKit package
- Browse Python examples

**Resources (3)** — things the MCP server can *provide*:
- The full documentation content
- Code examples
- API references

Together these give Cursor comprehensive access to everything LiveKit has published.

### When MCP Fires vs When It Doesn't

MCP only fires when you explicitly reference it:
- `@livekit-docs` in your prompt → MCP fetches docs
- No mention → Cursor uses training data only

**Practical rule:** Whenever writing or debugging livekit-agents code, always include `@livekit-docs` in your Cursor prompt. It takes 2 seconds to type and makes responses dramatically more accurate.

### The Markdown Fallback — When MCP is Unavailable

Every LiveKit docs page is available as plain Markdown by appending `.md` to the URL:

```
https://docs.livekit.io/agents/overview.md
https://docs.livekit.io/agents/voice-agent.md
https://docs.livekit.io/agents/logic/tools.md
```

If MCP goes down or isn't responding:
1. Open the relevant docs page in your browser
2. Append `.md` to the URL
3. Copy the Markdown content
4. Paste it directly into your Cursor prompt as context

This gives you the same information MCP would have fetched, just manually. Always have this as your backup.

---

## What We Installed

### Installation Method Used
Clicked **"Add to Cursor"** button on `https://docs.livekit.io/mcp` → browser opened Cursor → clicked Install in Cursor dialog.

### What Got Installed
A new MCP server entry in Cursor's settings pointing to:
```json
{
  "livekit-docs": {
    "url": "https://docs.livekit.io/mcp"
  }
}
```

### Where to Find It in Cursor
```
Cursor → Settings (Cmd+,) → Tools & MCP → Installed MCP Servers
```
You should see `livekit-docs` with green toggle and "7 tools, 3 resources enabled".

### Manual Installation (if button doesn't work)
In Cursor Settings → Tools & MCP → New MCP Server, paste:
```json
{
  "livekit-docs": {
    "url": "https://docs.livekit.io/mcp"
  }
}
```

---

## AGENTS.md Update — Why It Matters

### What Was Added

```markdown
## LiveKit Documentation

LiveKit Agents is a fast-evolving project and the
documentation is updated frequently. Always consult
the @livekit-docs MCP server first for any questions
about livekit-agents, LiveKit SDKs, or LiveKit Cloud.
Fall back to official LiveKit resources if needed.
Keep answers traceable to documentation sources.
```

### Why This Matters

`AGENTS.md` is read by Cursor at the start of every Agent session. By adding these instructions, we've told Cursor:
- "This project uses a fast-evolving library"
- "Always check live docs before answering"
- "Cite your sources"

Without this update, Cursor might forget to use MCP even when it's relevant. With it, MCP usage is baked into every session automatically — just like the rules files.

### The Three-Layer System Now Complete

| Layer | File | What it does |
|---|---|---|
| Layer 1 | `.cursor/rules/core.mdc` | Enforces uv, Python 3.12, no langchain, type hints |
| Layer 2 | `.cursor/rules/livekit.mdc` | Enforces 1.x API patterns when agent files are open |
| Layer 3 | `AGENTS.md` | Tells Cursor to always consult live docs via MCP |

Together these three layers mean Cursor starts every session already knowing your conventions, the correct API patterns, and where to look for current information. This is the professional Cursor setup.

---

## The Verification Test — What It Proved

### The Prompt Used
```
@livekit-docs show me the correct AgentSession + Agent 
pattern for livekit-agents 1.x with a minimal working example
```

### The Key Line in the Response
> *"directly from the official docs (rendered live just now at 2026-02-28)"*

This is proof MCP is working. Cursor explicitly told us it fetched the docs in real time on the day we ran the test — not from training memory.

### What the Response Confirmed
- ✅ `AgentSession + Agent` is the correct 1.x pattern
- ✅ `@server.rtc_session` is the correct entry point decorator
- ✅ `agents.cli.run_app(server)` is the correct way to run
- ✅ `session.generate_reply()` triggers the opening greeting
- ✅ `uv run agent.py dev` connects to LiveKit Cloud

---

## Key LiveKit 1.x Patterns — Learned From Live Docs

This is essential reference for Chat 6 when we write `agent.py`. Everything below came directly from the MCP-fetched live docs.

### The Core Pattern

```python
from livekit import agents, rtc
from livekit.agents import AgentServer, AgentSession, Agent, room_io
from livekit.plugins import silero

class Assistant(Agent):
    def __init__(self) -> None:
        super().__init__(
            instructions="Your system prompt goes here.",
        )

server = AgentServer()

@server.rtc_session(agent_name="my-agent")
async def my_agent(ctx: agents.JobContext) -> None:
    session = AgentSession(
        stt=deepgram.STT(model="nova-3"),
        llm=openai.LLM(model="gpt-4o-mini"),
        tts=elevenlabs.TTS(voice_id="your-voice-id"),
        vad=silero.VAD.load(),
    )
    await session.start(
        room=ctx.room,
        agent=Assistant(),
    )
    await session.generate_reply(
        instructions="Greet the user and offer your assistance."
    )

if __name__ == "__main__":
    agents.cli.run_app(server)
```

### Key Concepts — What Each Part Does

| Component | Role |
|---|---|
| `Agent` subclass | Defines AI logic — instructions, tools, pipeline overrides |
| `AgentSession` | Orchestrates the full pipeline: STT → LLM → TTS, manages turns |
| `AgentServer` | Hosts the session entry point |
| `@server.rtc_session` | The entry point decorator — wires a function to handle incoming calls |
| `session.start()` | Connects to LiveKit room, starts listening |
| `session.generate_reply()` | Triggers the opening greeting |
| `agents.cli.run_app(server)` | Gives you `console`, `dev`, `start` subcommands |

### Important for Our SIP Setup (Chat 8)

Since callers come via Twilio SIP (phone calls), we need telephony-specific noise cancellation:

```python
from livekit.plugins import noise_cancellation

# For SIP callers (phone) — use BVCTelephony
# For web/app callers — use BVC
room_options=room_io.RoomOptions(
    audio_input=room_io.AudioInputOptions(
        noise_cancellation=lambda params: (
            noise_cancellation.BVCTelephony()
            if params.participant.kind == rtc.ParticipantKind.PARTICIPANT_KIND_SIP
            else noise_cancellation.BVC()
        ),
    ),
)
```

All our callers are SIP (phone calls via Twilio), so we'll use `BVCTelephony`. MCP told us this — we didn't have to guess.

### Run Commands — Confirmed by Live Docs

```bash
# Download VAD and turn-detector model files (run once before first use)
uv run agent.py download-files

# Test locally in your terminal (no LiveKit Cloud needed)
uv run agent.py console

# Connect to LiveKit Cloud + Agents Playground (development)
uv run agent.py dev

# Production (used by Railway in Chat 16)
uv run agent.py start
```

### Two Ways to Specify Models

**Option 1 — Plugin objects (what we'll use — more control):**
```python
session = AgentSession(
    stt=deepgram.STT(model="nova-3"),
    llm=openai.LLM(model="gpt-4o-mini"),
    tts=elevenlabs.TTS(voice_id="your-voice-id"),
    vad=silero.VAD.load(),
)
```

**Option 2 — Inference shorthand strings (simpler, less control):**
```python
session = AgentSession(
    stt="deepgram/nova-3:multi",
    llm="openai/gpt-4.1-mini",
    tts="cartesia/sonic-3:voice-id",
    vad=silero.VAD.load(),
)
```

We'll use plugin objects for our build — they give us explicit control over model parameters and make our config.yml integration cleaner.

### Old 0.x vs New 1.x — Never Use These Again

| ❌ Old 0.x (wrong) | ✅ New 1.x (correct) |
|---|---|
| `VoicePipelineAgent` | `AgentSession + Agent` |
| `@llm.ai_callable` | `@function_tool` |
| `FunctionContext` class | Methods on `Agent` class |
| `WorkerOptions` entry point | `AgentServer + @server.rtc_session` |

---

## How To Do It Manually

If you ever need to set up the LiveKit MCP server manually (without the one-click button):

### Option 1 — Via Cursor Settings UI
1. Open Cursor → Settings (`Cmd+,`)
2. Navigate to Tools & MCP
3. Click "New MCP Server"
4. Add this JSON:
```json
{
  "livekit-docs": {
    "url": "https://docs.livekit.io/mcp"
  }
}
```
5. Save and verify green status appears

### Option 2 — Via Cursor Config File
Find Cursor's MCP config file (usually at `~/.cursor/mcp.json`) and add:
```json
{
  "mcpServers": {
    "livekit-docs": {
      "url": "https://docs.livekit.io/mcp"
    }
  }
}
```

### Update AGENTS.md Manually
Open `AGENTS.md` in any text editor and append:
```markdown
## LiveKit Documentation

LiveKit Agents is a fast-evolving project and the
documentation is updated frequently. Always consult
the @livekit-docs MCP server first for any questions
about livekit-agents, LiveKit SDKs, or LiveKit Cloud.
Fall back to official LiveKit resources if needed.
Keep answers traceable to documentation sources.
```

---

## Verify It Worked

### Check 1 — Cursor Settings
```
Cursor → Settings (Cmd+,) → Tools & MCP
```
Should show: `livekit-docs` with green toggle and "7 tools, 3 resources enabled"

### Check 2 — Live Test
In a new Cursor Agent session (`Cmd+I`):
```
@livekit-docs what is AgentSession in livekit-agents 1.x?
```

**Success:** Response mentions real-time fetch, uses 1.x patterns (`AgentSession`, `Agent`, `@function_tool`)

**Failure:** Response uses old patterns (`VoicePipelineAgent`, `@llm.ai_callable`) — means MCP isn't connecting, check Settings → Tools & MCP for red status

### Check 3 — AGENTS.md
```bash
cat AGENTS.md
# Bottom section should show the LiveKit Documentation instructions
```

### Your Confirmed Results ✅
- Green toggle in Cursor Settings ✅
- 7 tools, 3 resources enabled ✅
- Test response confirmed "rendered live just now at 2026-02-28" ✅
- AGENTS.md updated correctly ✅
- Git committed and pushed ✅

---

## Cursor Upskilling — MCP and the @ System

### Complete @ Reference for This Project

| What to type | When to use it | What happens |
|---|---|---|
| `@livekit-docs` | Any livekit-agents code | Fetches current LiveKit API docs |
| `@filename.py` | When Cursor needs to see a file | Includes file content in context |
| `@AGENTS.md` | When starting a complex session | Gives Cursor full project briefing |
| `@web` | For non-LiveKit current info | Web search |

### The Golden Rule for LiveKit Code

**Every Cursor prompt that touches `agent.py` or `tools.py` should include `@livekit-docs`.**

Good prompt structure:
```
@livekit-docs [what you want Cursor to do]
Following the patterns in @agent.py, [specific task].
Do not modify any other files.
```

### MCP vs Training Data — How to Tell the Difference

**MCP is working:** Response mentions live fetch, cites doc URLs, uses current patterns, may include date stamp

**Training data only:** Response has no citations, may use outdated patterns, feels generic

When in doubt, ask Cursor directly: "where did you get this information?" If MCP is working it will tell you.

### Other MCP Servers Worth Knowing About

The MCP ecosystem is growing. Other useful MCP servers you may add to future projects:

| Service | MCP URL | Use case |
|---|---|---|
| LiveKit | `https://docs.livekit.io/mcp` | What we installed |
| Qdrant | Available on their docs site | Vector DB queries (future project) |
| Stripe | Available via Stripe docs | Payment integration projects |

You add any MCP server the same way — Cursor Settings → Tools & MCP → New MCP Server → paste the JSON.

---

## Architecture Reminder — Where Chat 3 Fits

```
Chat P  ✅  All 7 accounts created, API keys saved
Chat 1  ✅  Project scaffold, uv, pyproject.toml, uv.lock
Chat 2  ✅  Cursor rules — conventions enforced automatically
Chat 3  ✅  LiveKit MCP — live docs inside Cursor, AGENTS.md updated
──────────────────────────────────────────────────────────
FOUNDATION COMPLETE — Cursor is fully equipped
──────────────────────────────────────────────────────────
Chat 4  →   config.py — the config loader all other files depend on
Chat 5  →   Customer data files — RockGas config, known users, orders
Chat 6  →   agent.py — the voice agent itself, first working version
Chat 7  →   API key verification
Chat 8  →   SIP configuration (Twilio + LiveKit)
Chat 9  →   First real phone call
```

After Chat 3, the foundation is complete. Everything from here is actual product code.

---

## Handoff Into Chat 4

Paste these 3 bullets at the start of your next Cursor session:

```
1. Foundation complete — project scaffold (Chat 1), Cursor rules
   (Chat 2), LiveKit MCP server (Chat 3) all in place. AGENTS.md
   updated to instruct Cursor to always consult @livekit-docs.
   MCP confirmed live: 7 tools, 3 resources enabled, green status.

2. livekit-agents==1.4.3 installed. 1.x patterns confirmed via
   live docs: AgentSession + Agent, @server.rtc_session entry
   point, agents.cli.run_app(server) runner, @function_tool for
   tools. SIP callers need BVCTelephony noise cancellation.

3. Ready for Chat 4: config.py — the central config loader that
   reads customers/{ACTIVE_CUSTOMER}/config.yml into typed Python
   objects. Every other file (agent.py, tools.py, ingest.py)
   imports from config.py. This is the foundation of the
   configurable-by-design architecture.
```

---

## Golden Rules — Updated After Chat 3

> **Always include `@livekit-docs` in Cursor prompts** when writing livekit-agents code

> **MCP fallback:** append `.md` to any LiveKit docs URL for Markdown version

> **Never `pip install`** — always `uv add` or `uv sync`

> **Never commit `.env`** — commit `.env.example` only

> **Always commit `uv.lock`** — Railway needs it for reproducible builds

> **Never hardcode customer values** — everything customer-specific in `customers/{name}/config.yml`

> **Use livekit-agents 1.x patterns** — `AgentSession + Agent`, `@function_tool`, never 0.x

> **Greeting templates in `config.yml`** — user data (`name`, `email`) in `known_users.json`

> **Start a new Cursor chat at ~70% context** — ask for state summary first

> **MCP is live-fetched** — if Cursor doesn't cite docs, add `@livekit-docs` to your prompt

---

*voice-agent-demo | Chat 3 Learning Notes | February 2026*
