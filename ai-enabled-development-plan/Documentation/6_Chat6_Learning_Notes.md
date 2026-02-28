# Chat 6 — Learning Notes
## AI Voice Agent Demo | agent.py — The Voice Agent

> **Project:** voice-agent-demo
> **Date:** February 2026
> **Phase:** Day 2 — Core Agent
> **Status:** ✅ Complete — Sam is live and talking

---

## Table of Contents
1. [What We Built](#what-we-built)
2. [The Big Picture — What agent.py Does](#the-big-picture)
3. [The Four-Component Pipeline](#the-four-component-pipeline)
4. [Every Part of agent.py Explained](#every-part-of-agentpy-explained)
5. [Key Python Concepts Learned](#key-python-concepts-learned)
6. [The Latency Numbers — What They Mean](#the-latency-numbers)
7. [Issues We Hit and How We Fixed Them](#issues-we-hit)
8. [The Cursor Prompt — Exact Copy](#the-cursor-prompt)
9. [How To Do It Manually](#how-to-do-it-manually)
10. [Verify It Worked](#verify-it-worked)
11. [Handoff Into Chat 7](#handoff-into-chat-7)

---

## What We Built

`agent.py` — the actual voice agent. The file that answers calls, identifies callers, greets them correctly, and holds a real conversation using speech-to-text, a language model, and text-to-speech.

**End state after Chat 6:**
```
voice-agent-demo/
├── agent.py                ← NEW — the voice agent
├── config.py               ← reads customer config
├── customers/rockgas/
│   ├── config.yml          ← Sam's personality and model config
│   ├── known_users.json    ← caller CRM
│   └── orders.csv          ← order data
├── pyproject.toml          ← updated with noise-cancellation package
└── uv.lock                 ← updated lockfile
```

**What Sam can do after Chat 6:**
- Answer calls and greet callers by name (if known) or generically
- Hold a natural conversation about RockGas products
- Understand NZ-accented speech via Deepgram Nova-3
- Speak in a NZ male voice via ElevenLabs
- Run locally in console mode for testing

**What Sam cannot do yet (coming in later chats):**
- Look up orders (Chat 12)
- Search the knowledge base (Chat 14)
- Send email summaries (Chat 13)
- Answer real phone calls (Chat 9)

---

## The Big Picture — What agent.py Does

### The Flow From Dial to Response

```
1. agent.py starts → loads config → connects to LiveKit
         ↓
2. Call arrives (console mode: your microphone)
         ↓
3. Extract caller phone from SIP headers
   (console mode: no phone → anonymous)
         ↓
4. lookup_user(cfg, phone) → known or anonymous
         ↓
5. Build RockGasAgent with correct system prompt
         ↓
6. Create AgentSession (STT + LLM + TTS + VAD)
         ↓
7. session.start() → connect to room
         ↓
8. session.generate_reply() → Sam speaks the greeting
         ↓
9. Caller speaks → Deepgram transcribes
         ↓
10. GPT-4o-mini generates response
         ↓
11. ElevenLabs speaks the response
         ↓
12. Repeat steps 9-11 until call ends
```

### Why agent.py is The Most Important File

Every other file exists to support this one:
- `config.py` → feeds `agent.py` customer values
- `known_users.json` → feeds `agent.py` caller identity
- `config.yml` → feeds `agent.py` model choices and prompts
- Tools (Chat 12-14) → extend what `agent.py` can do

---

## The Four-Component Pipeline

### STT — Deepgram Nova-3

**What it does:** Converts the caller's speech into text in real time.

**Why Deepgram Nova-3:**
- Nova-3 is Deepgram's most accurate model for conversational speech
- Handles NZ accents well (important for RockGas demo)
- Streaming transcription — starts processing before you finish speaking
- Very low latency — typically under 300ms

**What you saw in the terminal:**
```
received user transcript: "Can you tell me what else, uh, Rock Gas delivers?"
```
That's Nova-3 at work — picked up "Rock Gas" (two words) correctly.

---

### VAD — Silero

**What it does:** Voice Activity Detection — detects when the caller starts and stops speaking. Without VAD, the system wouldn't know when to start or stop processing speech.

**Why Silero:**
- Lightweight — runs locally, no API call needed
- Fast — adds minimal latency
- Accurate — handles background noise well
- Pre-warmed via `server.setup_fnc = _prewarm` so it's ready before the first call arrives

**The prewarm pattern:**
```python
def _prewarm(proc: JobProcess) -> None:
    proc.userdata["vad"] = silero.VAD.load()

server.setup_fnc = _prewarm
```
This loads the VAD model once per worker process — not once per call. Faster call setup.

---

### LLM — OpenAI GPT-4o-mini

**What it does:** Given the conversation history and system prompt, generates what Sam should say next.

**Why gpt-4o-mini:**
- ~10x cheaper than gpt-4o
- ~2x faster than gpt-4o
- More than capable for structured conversational tasks
- Voice agents need speed more than raw intelligence
- Can always upgrade per customer via `config.yml`

**The system prompt is Sam's brain:**
The system prompt tells the LLM who Sam is, how to behave, and what he can and cannot do. It's built dynamically from `config.yml` values — never hardcoded.

---

### TTS — ElevenLabs

**What it does:** Converts Sam's text response into natural speech using the configured voice.

**Voice used:** `JFZ2Sw5TN92wIBOEx7pZ` — "Sarn", a young adult NZ male voice

**Why ElevenLabs:**
- Most natural-sounding TTS available
- NZ-specific voices available
- Low first-byte latency (323ms in our test)
- Voice ID is configurable per customer via `config.yml`

**The env var name:** `ELEVEN_API_KEY` — this is what the livekit-plugins-elevenlabs package looks for by default.

---

### Noise Cancellation — BVCTelephony

**What it does:** Filters out phone line noise, background noise, and telephony artifacts from the caller's audio before it reaches Deepgram.

**Why BVCTelephony specifically:**
- `BVC` = standard noise cancellation (for web/app callers)
- `BVCTelephony` = telephony-optimised (for SIP/phone callers)
- Since all RockGas callers come via Twilio SIP (phone calls), we always use `BVCTelephony`
- Applied conditionally — only for SIP participants

**The package:** `livekit-plugins-noise-cancellation` — installed separately from `livekit-agents`

---

## Every Part of agent.py Explained

### Module-Level Setup

```python
load_dotenv()
cfg: CustomerConfig = load_config()
server = AgentServer()
```

- `load_dotenv()` — loads `.env` before anything else runs
- `cfg = load_config()` — runs once at startup, shared across all calls
- `server = AgentServer()` — the server that hosts the agent entry point

**Why module-level config?**
Loading config once at startup (not once per call) means:
- Faster call handling — no file I/O on each call
- Consistent config across all concurrent calls
- Errors caught at startup, not mid-call

---

### The Prewarm Function

```python
def _prewarm(proc: JobProcess) -> None:
    """Warm the Silero VAD model before the first call arrives."""
    proc.userdata["vad"] = silero.VAD.load()

server.setup_fnc = _prewarm
```

`setup_fnc` runs before any calls are accepted. Loading Silero here means:
- VAD is ready instantly when first call arrives
- No delay on first call while model loads
- Model is stored in `proc.userdata` and retrieved in `entrypoint`

---

### The System Prompt Builder

```python
def _build_instructions(agent_name: str, caller_name: str | None) -> str:
    """Build the agent system prompt from config values."""
```

Key rules enforced in the prompt:
- **No markdown** — bullet points, bold, headers all sound terrible when spoken aloud
- **Under 2 sentences** — voice responses must be concise
- **NZ-friendly tone** — warm, professional, not stiff
- **Offer human handoff** — "I'll connect you to our team" when unsure
- **Caller name included** — if known, LLM uses it naturally in conversation

---

### SIP Phone Extraction

```python
def _extract_sip_phone(room: rtc.Room) -> str | None:
    """Extract caller phone from SIP participant attributes."""

def _normalise_nz_phone(raw: str) -> str:
    """Convert various NZ phone formats to E.164."""
```

**Why two functions?**
Extracting the phone and normalising it are separate concerns:
- Extract: find the right participant and attribute
- Normalise: convert `021XXXXXXX` → `+6421XXXXXXX`

**The LiveKit participant attribute:**
`sip.phoneNumber` is the attribute name populated from the SIP From header when a call comes in via Twilio. This was confirmed by the MCP-fetched live docs in Chat 3.

**NZ number formats handled:**
```
+6421XXXXXXX  → +6421XXXXXXX  (already E.164)
6421XXXXXXX   → +6421XXXXXXX  (missing + prefix)
021XXXXXXX    → +6421XXXXXXX  (local NZ format)
sip:+64...@host → +64...      (SIP URI format)
```

---

### RockGasAgent Class

```python
class RockGasAgent(Agent):
    def __init__(self, caller_name: str | None = None) -> None:
        super().__init__(
            instructions=_build_instructions(cfg.agent.name, caller_name),
        )
```

**Why a subclass of Agent?**
The `Agent` class is the 1.x pattern. Subclassing it lets us:
- Pass dynamic instructions (caller-aware system prompt)
- Add `@function_tool` methods in Chat 12-14
- Override pipeline nodes if needed

**Why `caller_name: str | None`?**
The agent needs to know if the caller is known before building its system prompt. Known caller → personalised prompt. Anonymous → generic prompt.

---

### The Entry Point

```python
@server.rtc_session(agent_name="rockgas-agent")
async def entrypoint(ctx: JobContext) -> None:
```

**`@server.rtc_session`** — the 1.x entry point decorator. Registers this function to handle incoming sessions. `agent_name` must match what's registered in LiveKit Cloud for explicit dispatch.

**`async def`** — all LiveKit agent functions are async. This is mandatory — the entire audio pipeline is asynchronous I/O.

**`ctx: JobContext`** — the context object that gives you:
- `ctx.room` — the LiveKit room for this call
- `ctx.proc.userdata` — access to prewarmed models (Silero VAD)
- `ctx.connect()` — connect to the room

---

### Session Creation

```python
session = AgentSession(
    stt=deepgram.STT(model="nova-3"),
    llm=openai.LLM(
        model=cfg.llm.model,
        temperature=cfg.llm.temperature,
    ),
    tts=elevenlabs.TTS(voice_id=cfg.tts.voice_id),
    vad=vad,
)
```

Every value comes from `cfg` — zero hardcoded model names. Switching models for a different customer = change `config.yml` only.

---

### Starting the Session

```python
await session.start(
    room=ctx.room,
    agent=RockGasAgent(caller_name=caller_name),
    room_options=room_io.RoomOptions(
        audio_input=room_io.AudioInputOptions(
            noise_cancellation=noise_cancellation.BVCTelephony(),
        ),
    ),
)
await session.generate_reply(instructions=greeting)
```

**`session.start()`** — connects to the room, subscribes to audio tracks, begins listening.

**`room_options`** — applies `BVCTelephony` noise cancellation to all incoming audio. Critical for phone call quality.

**`session.generate_reply(instructions=greeting)`** — triggers Sam's opening line. The greeting string from `config.yml` is passed as instructions — the LLM speaks it as its first turn.

---

## Key Python Concepts Learned

### `async` / `await` — Why It's Everywhere

Voice agents deal with multiple things happening simultaneously:
- Receiving audio from the caller
- Sending audio to Deepgram for transcription
- Waiting for the LLM to respond
- Streaming TTS audio back to the caller

If any of these blocked (waited synchronously), the whole system would freeze. `async/await` lets Python switch between tasks while waiting — like a chef who puts something in the oven and starts chopping vegetables while it cooks, rather than standing there watching it.

**Rule for this project:** Every function that touches the network, audio, or LiveKit must be `async def` and called with `await`.

---

### `@server.rtc_session` Decorator

A decorator that registers a function as the handler for incoming LiveKit sessions. When a call arrives, LiveKit calls this function with a `JobContext`.

```python
@server.rtc_session(agent_name="rockgas-agent")
async def entrypoint(ctx: JobContext) -> None:
    # This runs once per incoming call
```

The `agent_name` is how LiveKit Cloud knows which agent to dispatch the call to — it must match the agent name registered in the LiveKit Cloud dashboard.

---

### `proc.userdata` — Sharing Data Between Functions

`proc.userdata` is a dictionary attached to the worker process. It's how the prewarm function shares the loaded VAD model with the entrypoint function:

```python
# In prewarm (runs once at startup):
proc.userdata["vad"] = silero.VAD.load()

# In entrypoint (runs per call):
vad = ctx.proc.userdata.get("vad") or silero.VAD.load()
```

The `or silero.VAD.load()` is a safety fallback — if somehow prewarm didn't run, load it fresh. Never crash, always have a fallback.

---

### `ctx.connect()` Before Participant Lookup

```python
await ctx.connect()
# NOW ctx.room.remote_participants is populated
phone = _extract_sip_phone(ctx.room)
```

**Critical order:** You must connect to the room before checking participants. Before `connect()`, the room object exists but has no participants in it yet. This was a specific pattern confirmed by the MCP live docs.

---

## The Latency Numbers — What They Mean

From your test session:
```
end_of_turn: 599ms · llm_ttft: 618ms · tts_ttfb: 323ms · e2e: 1543ms
```

| Metric | Value | Benchmark |
|---|---|---|
| `end_of_turn` | 599ms | Time VAD took to detect you stopped speaking |
| `llm_ttft` | 618ms | Time to first token from GPT-4o-mini |
| `tts_ttfb` | 323ms | Time to first audio byte from ElevenLabs |
| `e2e` | 1543ms | Total end-to-end response time |

**1.5 seconds is good.** Human conversation has natural pauses of 200-500ms. Under 2 seconds feels responsive. Over 3 seconds starts feeling laggy.

**The breakdown:**
- ~600ms is VAD processing (unavoidable — needs to be sure you stopped)
- ~600ms is LLM thinking (gpt-4o-mini is fast)
- ~300ms is TTS startup (ElevenLabs streaming latency)

**Where we can improve later if needed:**
- `end_of_turn_delay` parameter on VAD (reduce wait time)
- Switch to a realtime model (eliminates STT + TTS entirely)
- Use a faster TTS provider

---

## Issues We Hit and How We Fixed Them

### Issue 1 — `noise_cancellation` Import Error

**Error:**
```
ImportError: cannot import name 'noise_cancellation' from 'livekit.plugins'
```

**Cause:** `livekit-plugins-noise-cancellation` is a separate optional package not included in `livekit-agents` by default.

**Fix:**
```bash
uv add livekit-plugins-noise-cancellation
```

**Lesson:** When MCP shows you a pattern that uses a plugin, check if that plugin needs a separate install. The pattern `from livekit.plugins import noise_cancellation` requires its own package.

---

### Issue 2 — ElevenLabs API Key Error

**Error:**
```
ValueError: ElevenLabs API key is required, either as argument 
or set ELEVEN_API_KEY environmental variable
```

**Cause:** The `livekit-plugins-elevenlabs` package looks for `ELEVEN_API_KEY` by default. Our `.env` had `ELEVENLABS_API_KEY`.

**Fix:** Renamed the variable in `.env` to `ELEVEN_API_KEY`. Also reverted `.env.example` to match.

**Lesson:** Plugin env var names are fixed by the plugin author — always check what name a plugin expects before naming your env var.

**The correct env var names for each plugin:**
| Plugin | Expected env var |
|---|---|
| `livekit-plugins-deepgram` | `DEEPGRAM_API_KEY` |
| `livekit-plugins-elevenlabs` | `ELEVEN_API_KEY` |
| `livekit-plugins-openai` | `OPENAI_API_KEY` |
| `livekit-plugins-silero` | (no API key needed) |

---

### Issue 3 — Terminal Stuck in vi Editor

**Cause:** Typed `wq` in terminal which opened vi editor mode accidentally.

**Fix:** Press `Escape` then type `:q!` and press Enter to exit vi.

**Lesson:** 
- `Ctrl+C` — stop a running process
- `Ctrl+Z` — suspend a process (send to background)
- `Escape` + `:q!` — exit vi/vim editor
- `q` in terminal — quit certain interactive programs (like `less`)

---

### Issue 4 — API Keys Visible in Screenshot

**What happened:** A screenshot showed the `.env` file open with all real API keys visible.

**Action required:** Rotate these keys in their respective consoles:
- LiveKit → regenerate API key + secret
- Deepgram → regenerate API key
- ElevenLabs → regenerate API key
- OpenAI → regenerate API key
- Qdrant → regenerate API key
- Twilio → regenerate auth token

**Lesson:** Never share screenshots with your IDE showing `.env` file contents. Even in a private chat, it's a good habit to never expose secrets visually.

---

## The Cursor Prompt — Exact Copy

**Handoff context:**
```
1. Full data layer complete and verified — customers/rockgas/
   contains config.yml (agent: Sam, gpt-4o-mini, ElevenLabs
   voice JFZ2Sw5TN92wIBOEx7pZ, qdrant collection: rockgas-kb),
   known_users.json (3 users, E.164 keys, name+email+account),
   orders.csv (5 orders, joins on account field).

2. load_config() and lookup_user() both verified working.
   Lookup for +6421084300555 returns Saurabh/ACC-001.
   Unknown number returns None. ACTIVE_CUSTOMER=rockgas in .env.

3. Ready for Chat 6: agent.py — the voice agent itself.
   Use @livekit-docs. Pattern: AgentSession + Agent (1.x).
   Entry point: AgentServer + @server.rtc_session.
   SIP callers need BVCTelephony noise cancellation.
   Config via load_config() — no hardcoded values.
```

**Main prompt:**
```
@livekit-docs

Create agent.py in the project root.
Do not modify any other files.

Use livekit-agents 1.x API only:
- AgentSession + Agent (never VoicePipelineAgent)
- AgentServer + @server.rtc_session entry point
- agents.cli.run_app(server) as the runner
- @function_tool decorator for any tools
- BVCTelephony noise cancellation (SIP callers)

Allowed imports only:
- livekit.agents, livekit.agents.AgentServer, AgentSession,
  Agent, room_io, JobContext
- livekit.plugins.deepgram, elevenlabs, openai, silero
- livekit.plugins.noise_cancellation
- livekit.rtc (for ParticipantKind)
- config (load_config, lookup_user)
- logging, os
- dotenv (load_dotenv)

agent.py must do the following:

1. Load config at module level:
   cfg = load_config()

2. Define RockGasAgent(Agent) subclass with:
   - __init__ that accepts caller_name: str | None
   - System prompt built from cfg values — never hardcoded.
     If caller_name is known: include their name in prompt.
     If anonymous: generic helpful assistant prompt.
   - Prompt must instruct Sam to:
     * Keep responses under 2 sentences where possible
     * Never use markdown, bullet points, or symbols in responses
       (this is voice — markdown sounds terrible when spoken)
     * Be warm, professional, with a NZ-friendly tone
     * Help with LPG product questions, orders, pricing, safety
     * If unsure about something, offer to connect to a human

3. Define async entrypoint function decorated with
   @server.rtc_session(agent_name="rockgas-agent"):
   - Extract caller phone from SIP participant headers
     (the actual number comes from the SIP From header,
      extract the number part only, convert to E.164 +64 format)
   - Call lookup_user(cfg, phone) to identify caller
   - Build correct greeting:
     * Known caller: cfg.agent.welcome_known_template
       formatted with their name
     * Anonymous: cfg.agent.welcome_message
   - Instantiate RockGasAgent with caller_name if known
   - Create AgentSession with:
     * stt = deepgram.STT(model="nova-3")
     * llm = openai.LLM(model=cfg.llm.model,
                        temperature=cfg.llm.temperature)
     * tts = elevenlabs.TTS(voice_id=cfg.tts.voice_id)
     * vad = silero.VAD.load()
   - Apply BVCTelephony noise cancellation for SIP participants
   - Start session and generate opening reply with the
     correct greeting as instructions

4. Add type hints and docstrings to all functions.

5. Entry point:
   if __name__ == "__main__":
       agents.cli.run_app(server)

After completing, show me the full file content.
```

---

## How To Do It Manually

### Install the missing package
```bash
uv add livekit-plugins-noise-cancellation
```

### Verify imports
```bash
uv run python -c "from livekit.plugins import noise_cancellation; print('OK')"
```

### Download model files (once)
```bash
uv run agent.py download-files
```

### Test locally
```bash
uv run agent.py console   # uses your microphone
```

### Stop the agent
```
Ctrl+C
```

### Connect to LiveKit Cloud (development)
```bash
uv run agent.py dev
```

### Production mode (used by Railway)
```bash
uv run agent.py start
```

---

## Verify It Worked

### Import check:
```bash
uv run python -c "import agent; print('agent.py imports OK')"
# Expected: agent.py imports OK
```

### Console test confirmed ✅:
- Sam started and connected
- Microphone input transcribed correctly by Deepgram
- GPT-4o-mini generated responses
- ElevenLabs spoke responses aloud
- End-to-end latency: 1543ms ✅

---

## Architecture Reminder — Where Chat 6 Fits

```
Chat P  ✅  All 7 accounts created, API keys saved
Chat 1  ✅  Project scaffold, uv, pyproject.toml, uv.lock
Chat 2  ✅  Cursor rules — conventions enforced automatically
Chat 3  ✅  LiveKit MCP — live docs inside Cursor
Chat 4  ✅  config.py — CustomerConfig, load_config(), lookup_user()
Chat 5  ✅  RockGas data — config.yml, known_users.json, orders.csv
Chat 6  ✅  agent.py — Sam is live, talking, 1543ms e2e latency
═══════════════════════════════════════════════════════════════
Chat 7  →   API key verification script
Chat 8  →   SIP trunk config (Twilio → LiveKit)
Chat 9  →   First REAL phone call to +6498869807
Chat 10 →   Dockerfile + Railway deployment config
Chat 11 →   Knowledge base documents
Chat 12 →   Order lookup tool (@function_tool)
Chat 13 →   Email capture tool (@function_tool)
Chat 14 →   KB search tool (@function_tool)
Chat 15 →   ingest.py — load KB into Qdrant
Chat 16 →   Deploy to Railway — Sam goes to production
```

---

## Handoff Into Chat 7

Paste these 3 bullets at the start of your next Cursor session:

```
1. agent.py complete and verified — RockGasAgent(Agent) subclass,
   AgentSession with Deepgram Nova-3 STT + gpt-4o-mini LLM +
   ElevenLabs TTS + Silero VAD, BVCTelephony noise cancellation,
   SIP phone extraction, known/anonymous caller greeting logic.
   Console test passed — e2e latency 1543ms.

2. Package livekit-plugins-noise-cancellation==0.2.5 added to
   pyproject.toml and uv.lock. ElevenLabs env var confirmed as
   ELEVEN_API_KEY. All imports verified clean.

3. Ready for Chat 7: API key verification script — a utility
   that tests all 5 service connections (LiveKit, Deepgram,
   ElevenLabs, OpenAI, Qdrant) and reports pass/fail before
   we configure SIP in Chat 8.
```

---

## Golden Rules — Updated After Chat 6

> **`async def` everywhere** — all LiveKit functions must be async

> **`await ctx.connect()` before participant lookup** — room is empty until connected

> **Never markdown in voice responses** — bullets and bold sound terrible when spoken

> **`BVCTelephony` for SIP callers** — `BVC` for web/app, `BVCTelephony` for phone

> **Prewarm heavy models** — use `server.setup_fnc` to load Silero before first call

> **Plugin env var names are fixed** — `ELEVEN_API_KEY` not `ELEVENLABS_API_KEY`

> **`Ctrl+C`** — stop any running process in terminal

> **Never show `.env` in screenshots** — rotate keys if it happens

> **Always `@livekit-docs` in Cursor prompts** — for any livekit-agents code

> **`uv add` not `pip install`** — always, even for missing packages discovered late

> **Never commit `.env`** — commit `.env.example` only

> **Always commit `uv.lock`** — Railway needs it for reproducible builds

---

*voice-agent-demo | Chat 6 Learning Notes | February 2026*
