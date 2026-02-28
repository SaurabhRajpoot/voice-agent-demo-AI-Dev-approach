# Chat 7 — Learning Notes
## AI Voice Agent Demo | verify_keys.py — API Key Verification Script

> **Project:** voice-agent-demo
> **Date:** February 2026
> **Phase:** Day 2 — Verification & Hardening
> **Status:** ✅ Complete — 5/5 services healthy

---

## Table of Contents
1. [What We Built](#what-we-built)
2. [The Big Picture — Why a Verification Script](#the-big-picture)
3. [Every Part of verify_keys.py Explained](#every-part-explained)
4. [Key Python Concepts Learned](#python-concepts-learned)
5. [Issues We Hit and How We Fixed Them](#issues-we-hit)
6. [pyproject.toml — Final State](#pyprojecttoml-final-state)
7. [The Cursor Prompt — Exact Copy](#the-cursor-prompt)
8. [How To Do It Manually](#how-to-do-it-manually)
9. [Verify It Worked](#verify-it-worked)
10. [Where tests/ Fits In](#where-tests-fits-in)
11. [Handoff Into Chat 8](#handoff-into-chat-8)

---

## What We Built

A standalone utility script — `verify_keys.py` — that tests all 5 external service connections independently and reports a clean pass/fail summary. Run it anytime to confirm all services are healthy before a demo.

**End state after Chat 7:**
```
voice-agent-demo/
├── verify_keys.py          ← NEW — pre-flight checklist
├── agent.py                ← the voice agent
├── config.py               ← config loader
├── customers/rockgas/      ← customer data
├── pyproject.toml          ← UPDATED — 3 new packages added
└── uv.lock                 ← UPDATED — lockfile updated
```

**Confirmed output:**
```
✅ LiveKit      — connected to voice-agent-demo-4dm5as09
✅ Deepgram     — account active
✅ OpenAI       — gpt-4o-mini responding
✅ ElevenLabs   — voice found
✅ Qdrant       — cluster reachable

5/5 services healthy
```

---

## The Big Picture — Why a Verification Script

### The Problem Without It

Without `verify_keys.py`, diagnosing a broken service means:
1. Running the full agent
2. Watching it crash
3. Reading a cryptic traceback
4. Guessing which service failed
5. Checking logs, retrying, wasting time

This is painful during development and disastrous right before a client demo.

### The Solution — Pre-Flight Checklist

`verify_keys.py` tests each service in isolation in under 10 seconds. Like a pilot's pre-flight checklist — you don't assume everything is fine, you verify it systematically before every flight.

```bash
uv run python verify_keys.py
# Run this before every demo, every deployment, anytime something feels off
```

### Where It Lives — And Where It Doesn't

**Location:** Project root — `verify_keys.py`
**NOT in:** `tests/` folder

This is an **operational utility** — it makes real API calls to live external services. It's not a unit test of your own code logic. The distinction matters:

| File | Location | Purpose | Makes real API calls? |
|---|---|---|---|
| `verify_keys.py` | project root | Check live services | ✅ Yes |
| `tests/test_config.py` | `tests/` | Test your Python logic | ❌ No (mocked) |

---

## Every Part of verify_keys.py Explained

### The Full File Structure

```python
# Imports — 5 SDKs + stdlib
import asyncio, os
from deepgram import DeepgramClient
from dotenv import load_dotenv
from elevenlabs import ElevenLabs
from livekit.api import LiveKitAPI, ListRoomsRequest
from openai import AsyncOpenAI
from qdrant_client import AsyncQdrantClient

load_dotenv()  # load .env before any os.environ.get()

_ELEVENLABS_VOICE_ID = os.environ.get("ELEVENLABS_VOICE_ID", "JFZ2Sw5TN92wIBOEx7pZ")

# 5 async check functions — one per service
async def check_livekit() -> tuple[bool, str]: ...
async def check_deepgram() -> tuple[bool, str]: ...
async def check_openai() -> tuple[bool, str]: ...
async def check_elevenlabs() -> tuple[bool, str]: ...
async def check_qdrant() -> tuple[bool, str]: ...

# Main orchestrator
async def main() -> None: ...

if __name__ == "__main__":
    asyncio.run(main())
```

---

### The Return Type Pattern — `tuple[bool, str]`

Every check function returns the same shape:
```python
async def check_livekit() -> tuple[bool, str]:
    return True, "connected to voice-agent-demo-4dm5as09"  # success
    return False, "Connection refused"                       # failure
```

**Why this pattern?**
- `bool` — did it pass or fail?
- `str` — what happened? (success message or error text)
- Consistent across all 5 checks — `main()` handles them all the same way
- One failure doesn't affect others — each function is independent

---

### check_livekit()

```python
async def check_livekit() -> tuple[bool, str]:
    url = os.environ.get("LIVEKIT_URL", "")
    api_key = os.environ.get("LIVEKIT_API_KEY", "")
    api_secret = os.environ.get("LIVEKIT_API_SECRET", "")

    try:
        async with LiveKitAPI(url=url, api_key=api_key, api_secret=api_secret) as api:
            await api.room.list_rooms(ListRoomsRequest())
            project = url.split("//")[-1].split(".")[0] if url else "unknown"
            return True, f"connected to {project}"
    except Exception as exc:
        return False, str(exc)
```

**What it tests:** Creates a LiveKit API client and lists rooms. If credentials are wrong, this throws an auth error immediately.

**`async with`** — LiveKitAPI is used as an async context manager. It handles opening and closing the connection cleanly, even if an error occurs.

**Project name extraction:**
```python
url = "wss://voice-agent-demo-4dm5as09.livekit.cloud"
url.split("//")[-1]          # "voice-agent-demo-4dm5as09.livekit.cloud"
    .split(".")[0]           # "voice-agent-demo-4dm5as09"
```
That's why the output shows `connected to voice-agent-demo-4dm5as09`.

---

### check_deepgram()

```python
async def check_deepgram() -> tuple[bool, str]:
    api_key = os.environ.get("DEEPGRAM_API_KEY", "")
    if not api_key:
        return False, "DEEPGRAM_API_KEY not set or empty"

    try:
        client = DeepgramClient(api_key=api_key)
        response = await asyncio.to_thread(client.manage.v1.projects.list)
        return True, "account active"
    except Exception as exc:
        return False, str(exc)
```

**What it tests:** Initialises the Deepgram client and fetches the projects list. If the key is invalid, Deepgram returns a 401 error.

**The early return pattern:**
```python
if not api_key:
    return False, "DEEPGRAM_API_KEY not set or empty"
```
No point making an API call if the key is empty. Return immediately with a clear message.

**`asyncio.to_thread()`** — Deepgram's SDK v6 uses synchronous (blocking) calls internally. `asyncio.to_thread()` runs a blocking function in a thread pool so it doesn't block the async event loop. This is the correct pattern when a non-async library must be used inside an async function.

**The SDK version fix:**
Original code used `client.manage.v("1").get_projects()` — the old pattern. Deepgram SDK v6 changed this to `client.manage.v1.projects.list`. This is a breaking change between SDK versions — exactly why we have a verify script.

---

### check_openai()

```python
async def check_openai() -> tuple[bool, str]:
    api_key = os.environ.get("OPENAI_API_KEY", "")

    try:
        client = AsyncOpenAI(api_key=api_key)
        await client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": "hi"}],
            max_tokens=1,
        )
        return True, "gpt-4o-mini responding"
    except Exception as exc:
        return False, str(exc)
```

**What it tests:** Makes a real (but minimal) API call. `max_tokens=1` means the response is as short as possible — costs fractions of a cent.

**Why `AsyncOpenAI` not `OpenAI`?** The `AsyncOpenAI` client has `await`-able methods. The regular `OpenAI` client is synchronous and would need `asyncio.to_thread()`. OpenAI's SDK provides both natively.

**Why actually call the API rather than just check the key format?** A key can be syntactically valid but revoked, expired, or over quota. Only a real API call confirms it actually works.

---

### check_elevenlabs()

```python
async def check_elevenlabs() -> tuple[bool, str]:
    api_key = os.environ.get("ELEVEN_API_KEY", "")

    try:
        client = ElevenLabs(api_key=api_key)
        response = await asyncio.to_thread(client.voices.get_all)
        voice_ids = {v.voice_id for v in response.voices}
        if _ELEVENLABS_VOICE_ID not in voice_ids:
            return False, f"voice {_ELEVENLABS_VOICE_ID} not found in account"
        return True, "voice found"
    except Exception as exc:
        return False, str(exc)
```

**What it tests:** Fetches all voices in your ElevenLabs account AND confirms your specific voice ID (`JFZ2Sw5TN92wIBOEx7pZ`) is present. This catches two failure modes:
1. Invalid API key → auth error
2. Voice deleted or wrong ID → "voice not found" message

**Set comprehension:**
```python
voice_ids = {v.voice_id for v in response.voices}
```
`{...}` creates a Python `set` — like a list but with instant lookup. Checking `if voice_id in set` is O(1) — instant, regardless of how many voices you have.

**`_ELEVENLABS_VOICE_ID` constant:**
```python
_ELEVENLABS_VOICE_ID = os.environ.get("ELEVENLABS_VOICE_ID", "JFZ2Sw5TN92wIBOEx7pZ")
```
Configurable via env var, falls back to RockGas voice ID. The `_` prefix is Python convention for "module-private" — not meant to be imported by other files.

---

### check_qdrant()

```python
async def check_qdrant() -> tuple[bool, str]:
    url = os.environ.get("QDRANT_URL", "")
    api_key = os.environ.get("QDRANT_API_KEY", "")

    try:
        client = AsyncQdrantClient(url=url, api_key=api_key)
        await client.get_collections()
        return True, "cluster reachable"
    except Exception as exc:
        return False, str(exc)
```

**What it tests:** Connects to your Qdrant cluster and lists collections. We don't check for the `rockgas-kb` collection specifically here — just that the cluster is reachable and credentials are valid. The collection is created in Chat 15 (ingest.py).

**`AsyncQdrantClient`** — Qdrant's native async client. Doesn't need `asyncio.to_thread()` unlike Deepgram and ElevenLabs, because Qdrant's Python SDK is async-native.

---

### main() — The Orchestrator

```python
async def main() -> None:
    checks: list[tuple[str, asyncio.Task[tuple[bool, str]]]] = []

    async with asyncio.TaskGroup() as tg:
        checks = [
            ("LiveKit",     tg.create_task(check_livekit())),
            ("Deepgram",    tg.create_task(check_deepgram())),
            ("OpenAI",      tg.create_task(check_openai())),
            ("ElevenLabs",  tg.create_task(check_elevenlabs())),
            ("Qdrant",      tg.create_task(check_qdrant())),
        ]

    passed = 0
    for name, task in checks:
        ok, message = task.result()
        icon = "✅" if ok else "❌"
        print(f"{icon} {name} — {message}")
        if ok:
            passed += 1

    print(f"\n{passed}/5 services healthy")
```

**The key design:** All 5 checks run **concurrently** — not sequentially. `asyncio.TaskGroup` starts all 5 at the same time and waits for all to complete. This means total runtime is roughly the slowest single check (~2-3 seconds) rather than 5 checks added together (~10-15 seconds).

---

## Key Python Concepts Learned

### `asyncio.TaskGroup` — Concurrent Execution

`TaskGroup` is Python 3.11+'s built-in way to run multiple async tasks concurrently and wait for all of them.

```python
async with asyncio.TaskGroup() as tg:
    task_a = tg.create_task(check_livekit())   # starts immediately
    task_b = tg.create_task(check_deepgram())  # starts immediately
    task_c = tg.create_task(check_openai())    # starts immediately
# All three run concurrently. Execution resumes here when ALL are done.
```

**Without TaskGroup (sequential — slow):**
```python
result_a = await check_livekit()    # waits 2s
result_b = await check_deepgram()   # then waits 2s
result_c = await check_openai()     # then waits 2s
# Total: 6 seconds
```

**With TaskGroup (concurrent — fast):**
```python
async with asyncio.TaskGroup() as tg:
    tasks = [tg.create_task(check()) for check in checks]
# Total: ~2 seconds (all run simultaneously)
```

This is why all 5 checks complete in ~3 seconds rather than ~15 seconds.

---

### `asyncio.to_thread()` — Bridging Sync and Async

Some SDKs (Deepgram, ElevenLabs) have synchronous (blocking) interfaces. Calling a blocking function directly inside an `async def` freezes the entire event loop — all other tasks stop.

```python
# ❌ WRONG — blocks the event loop
result = client.manage.v1.projects.list()

# ✅ CORRECT — runs in thread pool, doesn't block
result = await asyncio.to_thread(client.manage.v1.projects.list)
```

`asyncio.to_thread()` runs the blocking function in a separate thread, allowing the async event loop to continue handling other tasks while it waits.

**When to use it:**
- SDK has no async version
- File I/O that might be slow
- Any CPU-intensive computation

---

### `try/except Exception` — Defensive Error Handling

Every check function wraps its logic in `try/except`:

```python
try:
    # make the API call
    return True, "success message"
except Exception as exc:
    return False, str(exc)
```

**Why `except Exception` (broad)?**
Each service can fail in different ways — network errors, auth errors, timeout errors, SDK-specific errors. Catching all exceptions means one failed service never crashes the script — the other 4 checks still run and report.

**`str(exc)`** — converts any exception to a human-readable string. This is what appears after the ❌ icon if a check fails.

---

### `async with` — Async Context Managers

```python
async with LiveKitAPI(...) as api:
    await api.room.list_rooms(...)
```

`async with` is the async version of `with`. It:
1. Opens the connection (`__aenter__`)
2. Runs your code
3. Closes the connection cleanly (`__aexit__`) — even if an error occurs

Without `async with`, you'd have to manually open and close connections, and a crash would leave connections open (resource leak).

---

### `asyncio.run()` — The Entry Point for Async

```python
if __name__ == "__main__":
    asyncio.run(main())
```

`asyncio.run()` is how you start an async program from synchronous code (like the top level of a script). It:
1. Creates a new event loop
2. Runs `main()` until it completes
3. Closes the event loop

You only call `asyncio.run()` once — at the very entry point of the program. Everything else inside uses `await`.

---

## Issues We Hit and How We Fixed Them

### Issue 1 — `ModuleNotFoundError: No module named 'deepgram'`

**Error:**
```
ModuleNotFoundError: No module named 'deepgram'
```

**Cause:** `deepgram-sdk` is a separate package from `livekit-plugins-deepgram`. The livekit plugin includes only what it needs internally — not the full SDK for direct use.

**Fix:**
```bash
uv add deepgram-sdk
```

**Lesson:** livekit plugins bundle their own dependencies. If you want to use a service SDK directly (not through livekit), install it separately.

---

### Issue 2 — `ModuleNotFoundError: No module named 'elevenlabs'`

**Same pattern as Deepgram.** `livekit-plugins-elevenlabs` ≠ `elevenlabs` SDK.

**Fix:**
```bash
uv add elevenlabs
```

---

### Issue 3 — Deepgram SDK API Changed Between Versions

**Error:**
```
❌ Deepgram — 'ManageClient' object has no attribute 'v'
```

**Cause:** Cursor wrote `client.manage.v("1").get_projects()` — the old Deepgram SDK pattern. The installed version (6.x) changed this to `client.manage.v1.projects.list`.

**Fix:** Updated `check_deepgram()`:
```python
# Old pattern (wrong for SDK v6)
response = await asyncio.to_thread(client.manage.v("1").get_projects)

# Correct pattern for SDK v6
response = await asyncio.to_thread(client.manage.v1.projects.list)
```

**Lesson:** SDK versions change APIs. This is exactly why we have a verify script — to catch these mismatches quickly rather than debugging the full agent at runtime.

---

## pyproject.toml — Final State

After Chats 6 and 7, three new packages were discovered and added:

```toml
[project]
name = "voice-agent-demo"
version = "0.1.0"
description = "AI voice pre-sales agent"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    # Core agent framework
    "livekit-agents~=1.0",
    # LiveKit plugins
    "livekit-plugins-deepgram",
    "livekit-plugins-elevenlabs",
    "livekit-plugins-openai",
    "livekit-plugins-silero",
    # Added in Chat 6 — needed for BVCTelephony
    "livekit-plugins-noise-cancellation>=0.2.5",
    # Vector database
    "qdrant-client",
    # OpenAI SDK (for embeddings)
    "openai",
    # Config and env
    "pyyaml",
    "python-dotenv",
    # Added in Chat 7 — direct SDKs for verify_keys.py
    "deepgram-sdk>=6.0.1",
    "elevenlabs>=2.37.0",
]

[dependency-groups]
dev = [
    "pytest",
    "pytest-asyncio",
]
```

**Why direct SDKs alongside livekit plugins?**

| Package | Used by | Purpose |
|---|---|---|
| `livekit-plugins-deepgram` | `agent.py` | Deepgram STT inside the voice pipeline |
| `deepgram-sdk` | `verify_keys.py` | Direct API calls to check account status |
| `livekit-plugins-elevenlabs` | `agent.py` | ElevenLabs TTS inside the voice pipeline |
| `elevenlabs` | `verify_keys.py` | Direct API calls to check voice exists |

The livekit plugins use the SDKs internally but don't expose them for direct use. You need both.

---

## The Cursor Prompt — Exact Copy

**Handoff context:**
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

**Main prompt:**
```
Create verify_keys.py in the project root.
Do not modify any other files.

Only use these imports:
- asyncio
- os
- dotenv (load_dotenv)
- livekit.api (LiveKitAPI)
- openai (AsyncOpenAI)
- deepgram (DeepgramClient)
- elevenlabs (ElevenLabs)
- qdrant_client (AsyncQdrantClient)

verify_keys.py must do the following:

1. Load .env at module level using load_dotenv()

2. Define one async check function per service,
   each returns tuple[bool, str] — (passed, message):

   check_livekit() → connect using LIVEKIT_URL,
     LIVEKIT_API_KEY, LIVEKIT_API_SECRET.
     List rooms to confirm connection works.
     Return True + "connected to {project}" on success.

   check_deepgram() → authenticate using DEEPGRAM_API_KEY.
     Call projects.get() to confirm key is valid.
     Return True + "account active" on success.

   check_openai() → authenticate using OPENAI_API_KEY.
     Make minimal chat completion (max_tokens=1,
     model="gpt-4o-mini", message="hi").
     Return True + "gpt-4o-mini responding" on success.

   check_elevenlabs() → authenticate using ELEVEN_API_KEY.
     Fetch voices list. Confirm voice ID from .env or
     hardcode JFZ2Sw5TN92wIBOEx7pZ as the check target.
     Return True + "voice found" on success.

   check_qdrant() → connect using QDRANT_URL, QDRANT_API_KEY.
     Call get_collections() to confirm cluster is reachable.
     Return True + "cluster reachable" on success.

3. Define async main() that:
   - Runs all 5 checks
   - Prints result for each:
     ✅ ServiceName — message   (on pass)
     ❌ ServiceName — error     (on fail)
   - Prints summary: "X/5 services healthy"
   - Each check runs independently — one failure
     does not stop the others

4. Entry point:
   if __name__ == "__main__":
       asyncio.run(main())

5. Type hints and docstrings on all functions.

After completing, show me the full file content.
```

---

## How To Do It Manually

### Install the required direct SDKs
```bash
uv add deepgram-sdk
uv add elevenlabs
```

### Run the script
```bash
uv run python verify_keys.py
```

### If a check fails — debug that service specifically
```bash
# Test just LiveKit
uv run python -c "
import asyncio
from livekit.api import LiveKitAPI, ListRoomsRequest
import os
from dotenv import load_dotenv
load_dotenv()
async def test():
    async with LiveKitAPI(
        url=os.environ['LIVEKIT_URL'],
        api_key=os.environ['LIVEKIT_API_KEY'],
        api_secret=os.environ['LIVEKIT_API_SECRET']
    ) as api:
        rooms = await api.room.list_rooms(ListRoomsRequest())
        print('OK:', rooms)
asyncio.run(test())
"
```

---

## Verify It Worked

### Command:
```bash
uv run python verify_keys.py
```

### Your confirmed result ✅:
```
✅ LiveKit      — connected to voice-agent-demo-4dm5as09
✅ Deepgram     — account active
✅ OpenAI       — gpt-4o-mini responding
✅ ElevenLabs   — voice found
✅ Qdrant       — cluster reachable

5/5 services healthy
```

---

## Where tests/ Fits In

We have an empty `tests/` folder — here's what it's for and when we'll use it.

### `tests/` = Automated Unit Tests

These test your own Python logic — not external services:

```python
# tests/test_config.py — tests we'll add in a later chat
def test_load_config_missing_customer():
    """load_config() raises ValueError if ACTIVE_CUSTOMER not set."""

def test_lookup_user_known():
    """lookup_user() returns dict for a known phone number."""

def test_lookup_user_unknown():
    """lookup_user() returns None for unknown number."""

def test_normalise_nz_phone_local():
    """021XXXXXXX normalises to +6421XXXXXXX."""

def test_normalise_nz_phone_e164():
    """+6421XXXXXXX stays unchanged."""
```

Run with:
```bash
uv run pytest
```

**Key differences:**
| | `verify_keys.py` | `tests/test_*.py` |
|---|---|---|
| Tests | External services | Your own code logic |
| API calls | Yes — real calls | No — mocked/fake data |
| Costs money | Yes (tiny) | No |
| Run when | Before every demo | After every code change |
| Speed | ~3 seconds | < 1 second |

---

## Architecture Reminder — Where Chat 7 Fits

```
Chat P  ✅  All 7 accounts created, API keys saved
Chat 1  ✅  Project scaffold, uv, pyproject.toml, uv.lock
Chat 2  ✅  Cursor rules — conventions enforced automatically
Chat 3  ✅  LiveKit MCP — live docs inside Cursor
Chat 4  ✅  config.py — CustomerConfig, load_config(), lookup_user()
Chat 5  ✅  RockGas data — config.yml, known_users.json, orders.csv
Chat 6  ✅  agent.py — Sam is live, 1543ms e2e latency
Chat 7  ✅  verify_keys.py — 5/5 services healthy
═══════════════════════════════════════════════════════════════
Chat 8  →   SIP trunk config (Twilio → LiveKit) — browser config
Chat 9  →   First REAL phone call to +6498869807
Chat 10 →   Dockerfile + Railway deployment config
Chat 11 →   Knowledge base documents
Chat 12 →   Order lookup tool (@function_tool)
Chat 13 →   Email capture tool (@function_tool)
Chat 14 →   KB search tool (Qdrant)
Chat 15 →   ingest.py — load KB into Qdrant
Chat 16 →   Deploy to Railway — Sam goes to production
```

---

## Handoff Into Chat 8

Paste these 3 bullets at the start of your next session:

```
1. verify_keys.py complete — 5/5 services healthy (LiveKit,
   Deepgram, OpenAI, ElevenLabs, Qdrant). All direct SDKs
   installed: deepgram-sdk==6.0.1, elevenlabs==2.37.0.
   pyproject.toml and uv.lock updated and committed.

2. Full stack verified and working: agent.py runs in console
   mode (1543ms e2e), verify_keys.py confirms all 5 services,
   config pipeline tested end-to-end. ACTIVE_CUSTOMER=rockgas.

3. Ready for Chat 8: SIP trunk configuration — browser-based
   setup in Twilio and LiveKit consoles to route calls from
   Twilio NZ number +6498869807 through to the LiveKit agent.
   No code in this chat — pure configuration.
```

---

## Golden Rules — Updated After Chat 7

> **Run `verify_keys.py` before every demo** — 10 seconds, saves hours

> **livekit plugins ≠ direct SDKs** — install both if you need direct access

> **`asyncio.TaskGroup`** — run multiple async tasks concurrently, not sequentially

> **`asyncio.to_thread()`** — wrap blocking SDK calls inside async functions

> **`try/except Exception`** — each check independent, one failure doesn't stop others

> **`async with`** — always use for connections that need clean teardown

> **`asyncio.run(main())`** — the one entry point for async programs

> **SDK versions change APIs** — verify scripts catch this instantly

> **`Ctrl+C`** — stop any running process in terminal

> **Always `uv add`** — never `pip install`, even for missing packages

> **Never commit `.env`** — rotate keys if accidentally exposed

> **Always commit `uv.lock`** — Railway needs it for reproducible builds

---

*voice-agent-demo | Chat 7 Learning Notes | February 2026*
