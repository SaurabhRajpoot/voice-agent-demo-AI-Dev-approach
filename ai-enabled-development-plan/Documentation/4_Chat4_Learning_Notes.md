# Chat 4 — Learning Notes
## AI Voice Agent Demo | config.py — Central Configuration Loader

> **Project:** voice-agent-demo
> **Date:** February 2026
> **Phase:** Day 1 — Foundation (final piece)
> **Status:** ✅ Complete

---

## Table of Contents
1. [What We Built](#what-we-built)
2. [The Big Picture — Why config.py Exists](#the-big-picture)
3. [Python Concepts Learned](#python-concepts-learned)
4. [Every Part of config.py Explained](#every-part-of-configpy-explained)
5. [The Cursor Prompt — Exact Copy](#the-cursor-prompt)
6. [What Cursor Did — Step by Step](#what-cursor-did)
7. [Smart Decisions Cursor Made](#smart-decisions-cursor-made)
8. [How To Do It Manually](#how-to-do-it-manually)
9. [Verify It Worked](#verify-it-worked)
10. [How config.py Connects to Everything](#how-configpy-connects-to-everything)
11. [Handoff Into Chat 5](#handoff-into-chat-5)

---

## What We Built

A single Python file — `config.py` — that is the **central front door to all customer configuration**. Every other file in the project imports from this one file. It reads `customers/{ACTIVE_CUSTOMER}/config.yml`, parses it, and returns typed Python objects.

**End state after Chat 4:**
```
voice-agent-demo/
├── config.py           ← NEW — the config loader
├── .cursor/rules/
├── customers/rockgas/docs/
├── AGENTS.md
├── pyproject.toml
└── uv.lock
```

**What `config.py` provides to the rest of the project:**
```python
from config import load_config, lookup_user

cfg = load_config()          # reads ACTIVE_CUSTOMER → config.yml → typed object
cfg.agent.name               # "Aria"
cfg.agent.welcome_message    # "Welcome to RockGas..."
cfg.llm.model                # "gpt-4o-mini"
cfg.tts.voice_id             # "your-elevenlabs-voice-id"
cfg.qdrant.collection        # "rockgas-kb"

user = lookup_user(cfg, "+6421000001")  # returns {name, email} or None
```

---

## The Big Picture — Why config.py Exists

### The Problem It Solves

In a naive build, a developer writes `agent.py` like this:

```python
# ❌ WRONG — hardcoded, not reusable
session = AgentSession(
    llm=openai.LLM(model="gpt-4o-mini"),
    tts=elevenlabs.TTS(voice_id="abc123"),
)
await session.generate_reply(
    instructions="Welcome to RockGas. How can I help?"
)
```

This works for RockGas. But when you want to demo for MBIE, you have to edit `agent.py` directly — changing values scattered across hundreds of lines of code, risking breaking something.

### The Solution — One Front Door

With `config.py`, the same `agent.py` works for any customer:

```python
# ✅ CORRECT — configurable by design
cfg = load_config()  # reads ACTIVE_CUSTOMER from .env

session = AgentSession(
    llm=openai.LLM(model=cfg.llm.model),
    tts=elevenlabs.TTS(voice_id=cfg.tts.voice_id),
)
await session.generate_reply(
    instructions=cfg.agent.welcome_message
)
```

Switching from RockGas to MBIE = change one line in `.env`:
```bash
ACTIVE_CUSTOMER=mbie  # was: rockgas
```

Zero code changes. Zero risk.

### The Hotel Concierge Analogy

`config.py` is like a hotel concierge. Every department (agent, tools, ingest) asks the concierge for information. The concierge knows which guest is checked in (`ACTIVE_CUSTOMER`) and gives the right answer for that guest. No department needs to know the details — they just ask the concierge.

---

## Python Concepts Learned

### Dataclasses — What They Are and Why We Use Them

A dataclass is Python's way of creating a structured container with named, typed fields. It's declared with the `@dataclass` decorator.

**Without dataclasses (fragile):**
```python
# Dictionary — no type checking, no autocomplete, easy to mistype
config = yaml.safe_load(file)
voice_id = config["tts"]["voice_id"]  # what if key is wrong?
```

**With dataclasses (robust):**
```python
@dataclass
class TTSConfig:
    voice_id: str

cfg = load_config()
voice_id = cfg.tts.voice_id  # typed, autocompletable, clear
```

**Why dataclasses specifically (not regular classes)?**
Dataclasses auto-generate `__init__`, `__repr__`, and `__eq__` methods. You get a clean structured object without writing boilerplate. Python 3.7+ has them built in — no extra package needed.

---

### Type Hints — What They Are

Type hints tell Python (and Cursor) what type each variable should be. They don't enforce anything at runtime but they:
- Enable Cursor autocomplete
- Make code self-documenting
- Catch mistakes early when using tools like mypy

```python
def lookup_user(cfg: CustomerConfig, phone: str) -> dict | None:
    #              ^^^ type hint         ^^^        ^^^^^^^^^^
    #         parameter type          return type   union type
```

**`dict | None`** — Python 3.10+ syntax meaning "returns either a dict or None". Valid because we're on Python 3.12.

---

### The `@dataclass` Decorator

A decorator is a function that wraps another function or class to add behaviour. `@dataclass` is Python's built-in decorator that transforms a class definition into a structured data container.

```python
from dataclasses import dataclass

@dataclass              # ← this decorator does the magic
class AgentConfig:
    name: str           # ← field with type hint
    welcome_message: str
    welcome_known_template: str
```

After this, `AgentConfig(name="Aria", welcome_message="...", welcome_known_template="...")` works automatically — the `__init__` is generated for free.

---

### `pathlib.Path` — Modern File Path Handling

`pathlib.Path` is Python's modern way to work with file paths. Better than string concatenation.

```python
from pathlib import Path

# Old way (fragile on Windows)
path = "customers/" + customer_id + "/config.yml"

# Modern way (works on all platforms)
path = Path("customers") / customer_id / "config.yml"

# Check if file exists
if not path.exists():
    raise FileNotFoundError(...)

# Open it
with path.open("r", encoding="utf-8") as fh:
    content = yaml.safe_load(fh)
```

The `/` operator on `Path` objects builds paths — it doesn't divide numbers.

---

### `yaml.safe_load()` — Parsing YAML Files

YAML (Yet Another Markup Language) is a human-readable config format — easier to read and write than JSON for configuration files. `yaml.safe_load()` parses a YAML file into a Python dictionary.

```yaml
# config.yml
agent:
  name: Aria
  welcome_message: "Welcome to RockGas"
```

```python
import yaml

with open("config.yml") as f:
    raw = yaml.safe_load(f)

raw["agent"]["name"]           # "Aria"
raw["agent"]["welcome_message"] # "Welcome to RockGas"
```

**Why `safe_load` not `load`?** `yaml.load()` can execute arbitrary Python code embedded in the YAML file — a security risk. `yaml.safe_load()` only parses data, never executes code. Always use `safe_load`.

---

### `load_dotenv()` — Loading the `.env` File

`load_dotenv()` from the `python-dotenv` package reads your `.env` file and loads all key=value pairs into `os.environ`. This means when you call `os.environ.get("ACTIVE_CUSTOMER")` after `load_dotenv()`, it finds the value from your `.env` file.

```python
from dotenv import load_dotenv
import os

load_dotenv()  # reads .env file → puts values in os.environ

customer_id = os.environ.get("ACTIVE_CUSTOMER")  # now works
```

**Important behaviour:** `load_dotenv()` is a no-op (does nothing) if the variables are already set in the environment. This means it works correctly in:
- Local development: reads from `.env`
- Railway production: reads from Railway's environment variables (already set, `.env` not present)
- CI/CD: reads from CI environment variables

---

### Raising Exceptions — `ValueError` and `FileNotFoundError`

When something is wrong, Python functions should raise exceptions rather than silently continuing. We raise two types:

```python
# When required config is missing
raise ValueError(
    "ACTIVE_CUSTOMER environment variable is not set. "
    "Add it to your .env file."
)

# When a required file doesn't exist
raise FileNotFoundError(
    f"Configuration file not found: {config_path}."
)
```

**Why raise rather than return None?**
If `ACTIVE_CUSTOMER` is missing, nothing in the system can work. A loud failure with a clear message is better than a silent failure that crashes somewhere unexpected later with a confusing error.

---

## Every Part of config.py Explained

### The Full File

```python
"""Central configuration loader for the voice-agent-demo project."""

import json
import os
from dataclasses import dataclass
from pathlib import Path

import yaml
from dotenv import load_dotenv


@dataclass
class AgentConfig:
    """Configuration for the agent's identity and greeting behaviour."""
    name: str
    welcome_message: str
    welcome_known_template: str


@dataclass
class LLMConfig:
    """Configuration for the language model."""
    model: str
    temperature: float


@dataclass
class TTSConfig:
    """Configuration for the text-to-speech voice."""
    voice_id: str


@dataclass
class QdrantConfig:
    """Configuration for the Qdrant vector store collection."""
    collection: str


@dataclass
class DataConfig:
    """Paths to the customer data files."""
    known_users_file: str
    orders_file: str


@dataclass
class CustomerConfig:
    """Top-level configuration for a single customer deployment."""
    customer_id: str
    agent: AgentConfig
    llm: LLMConfig
    tts: TTSConfig
    qdrant: QdrantConfig
    data: DataConfig


def load_config() -> CustomerConfig:
    """Load and return the customer configuration.

    Reads the ACTIVE_CUSTOMER environment variable, locates the corresponding
    customers/{ACTIVE_CUSTOMER}/config.yml file, parses it, and returns a
    fully populated CustomerConfig dataclass.

    Returns
    -------
    CustomerConfig
        Populated configuration for the active customer.

    Raises
    ------
    ValueError
        If the ACTIVE_CUSTOMER environment variable is not set.
    FileNotFoundError
        If the config.yml file for the active customer does not exist.
    """
    load_dotenv()

    customer_id = os.environ.get("ACTIVE_CUSTOMER")
    if not customer_id:
        raise ValueError(
            "ACTIVE_CUSTOMER environment variable is not set. "
            "Add it to your .env file or export it before running the agent."
        )

    config_path = Path("customers") / customer_id / "config.yml"
    if not config_path.exists():
        raise FileNotFoundError(
            f"Configuration file not found: {config_path}. "
            f"Ensure customers/{customer_id}/config.yml exists."
        )

    with config_path.open("r", encoding="utf-8") as fh:
        raw: dict = yaml.safe_load(fh)

    return CustomerConfig(
        customer_id=customer_id,
        agent=AgentConfig(
            name=raw["agent"]["name"],
            welcome_message=raw["agent"]["welcome_message"],
            welcome_known_template=raw["agent"]["welcome_known_template"],
        ),
        llm=LLMConfig(
            model=raw["llm"]["model"],
            temperature=float(raw["llm"]["temperature"]),
        ),
        tts=TTSConfig(
            voice_id=raw["tts"]["voice_id"],
        ),
        qdrant=QdrantConfig(
            collection=raw["qdrant"]["collection"],
        ),
        data=DataConfig(
            known_users_file=raw["data"]["known_users_file"],
            orders_file=raw["data"]["orders_file"],
        ),
    )


def lookup_user(cfg: CustomerConfig, phone: str) -> dict | None:
    """Look up a caller by their E.164 phone number.

    Parameters
    ----------
    cfg:
        The active customer configuration.
    phone:
        Caller phone number in E.164 format, e.g. +64211234567.

    Returns
    -------
    dict | None
        A dict with name and email keys when the caller is known,
        or None if they are not in the file.
    """
    users_path = Path(cfg.data.known_users_file)
    if not users_path.exists():
        return None

    with users_path.open("r", encoding="utf-8") as fh:
        users: dict = json.load(fh)

    return users.get(phone)
```

---

### Line-by-Line: The Dataclass Hierarchy

```
CustomerConfig              ← top level, one per customer
├── customer_id: str        ← "rockgas", "mbie", etc.
├── agent: AgentConfig      ← name, greeting templates
├── llm: LLMConfig          ← model, temperature
├── tts: TTSConfig          ← voice_id
├── qdrant: QdrantConfig    ← collection name
└── data: DataConfig        ← file paths
```

This hierarchy mirrors the structure of `config.yml` exactly. Each section in YAML maps to one dataclass.

---

### Line-by-Line: `load_config()`

```python
load_dotenv()
```
Step 1: Load `.env` file into environment. No-op in production where env vars are already set.

```python
customer_id = os.environ.get("ACTIVE_CUSTOMER")
if not customer_id:
    raise ValueError(...)
```
Step 2: Read `ACTIVE_CUSTOMER`. Fail loudly if missing — everything depends on this.

```python
config_path = Path("customers") / customer_id / "config.yml"
if not config_path.exists():
    raise FileNotFoundError(...)
```
Step 3: Build the path to the right config file. Fail loudly if the customer folder doesn't exist.

```python
with config_path.open("r", encoding="utf-8") as fh:
    raw: dict = yaml.safe_load(fh)
```
Step 4: Parse the YAML into a raw Python dictionary.

```python
return CustomerConfig(
    customer_id=customer_id,
    agent=AgentConfig(...),
    ...
)
```
Step 5: Map each YAML section into its dataclass. Return the fully typed object.

---

### Line-by-Line: `lookup_user()`

```python
users_path = Path(cfg.data.known_users_file)
if not users_path.exists():
    return None
```
Graceful handling — if the file doesn't exist yet (e.g. empty customer scaffold), return None instead of crashing.

```python
with users_path.open("r", encoding="utf-8") as fh:
    users: dict = json.load(fh)
return users.get(phone)
```
Load the JSON. Use `.get(phone)` which returns `None` if the key doesn't exist — no KeyError.

---

## The Cursor Prompt — Exact Copy

**Handoff context (pasted first):**
```
1. Foundation complete — project scaffold (Chat 1), Cursor rules
   (Chat 2), LiveKit MCP server (Chat 3) all in place. AGENTS.md
   updated to instruct Cursor to always consult @livekit-docs.
   MCP confirmed live: 7 tools, 3 resources enabled.

2. livekit-agents==1.4.3 installed. 1.x patterns confirmed:
   AgentSession + Agent, @server.rtc_session, @function_tool.
   SIP callers use BVCTelephony noise cancellation.

3. known_users.json schema: keys = E.164 phone numbers, values =
   {name, email} only. Greeting templates live in config.yml.
   Ready for Chat 4: config.py — the central config loader.
```

**Main prompt:**
```
Create config.py in the project root.
Do not modify any other files.
Only use these imports: dataclasses, pathlib, os, yaml,
dotenv (python-dotenv).

config.py must do the following:

1. Define these dataclasses:
   - AgentConfig: name(str), welcome_message(str),
     welcome_known_template(str)
   - LLMConfig: model(str), temperature(float)
   - TTSConfig: voice_id(str)
   - QdrantConfig: collection(str)
   - DataConfig: known_users_file(str), orders_file(str)
   - CustomerConfig: customer_id(str), agent(AgentConfig),
     llm(LLMConfig), tts(TTSConfig), qdrant(QdrantConfig),
     data(DataConfig)

2. Define load_config() function that:
   - Loads .env using python-dotenv
   - Reads ACTIVE_CUSTOMER env var (raise ValueError if missing)
   - Builds path: customers/{ACTIVE_CUSTOMER}/config.yml
   - Raises FileNotFoundError if that path does not exist
   - Reads and parses the YAML file
   - Returns a fully populated CustomerConfig dataclass

3. Define lookup_user(cfg: CustomerConfig, phone: str) function:
   - Reads known_users_file path from cfg.data.known_users_file
   - Returns the user dict if phone exists, None if not found
   - Phone numbers are E.164 format (+64xxxxxxxxx)

4. All functions must have type hints and docstrings.

After completing, show me the full file content.
```

---

## What Cursor Did — Step by Step

### STEP 1 — Defined all 6 dataclasses
Cursor created the dataclass hierarchy in order: leaf classes first (`AgentConfig`, `LLMConfig`, `TTSConfig`, `QdrantConfig`, `DataConfig`), then the top-level `CustomerConfig` that references them all. This ordering matters — Python needs to see a class before it can be referenced.

### STEP 2 — Wrote `load_config()` with proper error handling
Cursor correctly implemented the two-stage validation:
1. Check `ACTIVE_CUSTOMER` exists (ValueError if not)
2. Check the config file exists (FileNotFoundError if not)

Then parsed YAML and mapped each section to its dataclass.

### STEP 3 — Wrote `lookup_user()` with graceful file handling
Rather than crashing if `known_users_file` doesn't exist yet, Cursor returned `None` — allowing the agent to handle unknown callers gracefully.

### STEP 4 — Added NumPy-style docstrings
Cursor used the NumPy docstring format (with `Parameters`, `Returns`, `Raises` sections) — a professional Python convention for public functions.

### STEP 5 — Used `float()` cast on temperature
Small but important: `temperature=float(raw["llm"]["temperature"])` guards against YAML parsing `0.7` as a string in some edge cases.

---

## Smart Decisions Cursor Made

These weren't in the prompt — Cursor added them because `core.mdc` rules and good Python practice guided it:

| Decision | Why it's correct |
|---|---|
| `float()` cast on temperature | Guards against YAML parsing edge cases |
| `users_path.exists()` check before reading | Graceful handling during customer scaffold setup |
| `users.get(phone)` not `users[phone]` | Returns None instead of raising KeyError |
| `load_dotenv()` first in `load_config()` | Ensures env vars are available before any reads |
| NumPy-style docstrings | Professional Python convention, clear parameter documentation |
| `encoding="utf-8"` on all file opens | Explicit encoding — works on all platforms |

---

## How To Do It Manually

If you ever need to create a config loader from scratch without Cursor:

### Step 1: Create the file
```bash
touch config.py
```

### Step 2: Add imports
```python
import json
import os
from dataclasses import dataclass
from pathlib import Path

import yaml
from dotenv import load_dotenv
```

### Step 3: Define dataclasses (leaf classes first)
```python
@dataclass
class AgentConfig:
    name: str
    welcome_message: str
    welcome_known_template: str

@dataclass
class LLMConfig:
    model: str
    temperature: float

@dataclass
class TTSConfig:
    voice_id: str

@dataclass
class QdrantConfig:
    collection: str

@dataclass
class DataConfig:
    known_users_file: str
    orders_file: str

@dataclass
class CustomerConfig:
    customer_id: str
    agent: AgentConfig
    llm: LLMConfig
    tts: TTSConfig
    qdrant: QdrantConfig
    data: DataConfig
```

### Step 4: Write `load_config()`
```python
def load_config() -> CustomerConfig:
    load_dotenv()
    customer_id = os.environ.get("ACTIVE_CUSTOMER")
    if not customer_id:
        raise ValueError("ACTIVE_CUSTOMER not set")
    config_path = Path("customers") / customer_id / "config.yml"
    if not config_path.exists():
        raise FileNotFoundError(f"Not found: {config_path}")
    with config_path.open("r", encoding="utf-8") as fh:
        raw = yaml.safe_load(fh)
    return CustomerConfig(
        customer_id=customer_id,
        agent=AgentConfig(**raw["agent"]),
        llm=LLMConfig(
            model=raw["llm"]["model"],
            temperature=float(raw["llm"]["temperature"]),
        ),
        tts=TTSConfig(**raw["tts"]),
        qdrant=QdrantConfig(**raw["qdrant"]),
        data=DataConfig(**raw["data"]),
    )
```

### Step 5: Write `lookup_user()`
```python
def lookup_user(cfg: CustomerConfig, phone: str) -> dict | None:
    users_path = Path(cfg.data.known_users_file)
    if not users_path.exists():
        return None
    with users_path.open("r", encoding="utf-8") as fh:
        users = json.load(fh)
    return users.get(phone)
```

### Step 6: Verify
```bash
uv run python -c "from config import load_config, lookup_user; print('config.py imports OK')"
# Expected: config.py imports OK
```

---

## Verify It Worked

### Command:
```bash
uv run python -c "from config import load_config, lookup_user; print('config.py imports OK')"
```

### Your confirmed result ✅:
```
config.py imports OK
```

### What this proves:
- `config.py` is valid Python syntax
- All imports resolve correctly (`yaml`, `dotenv`, `dataclasses`, `pathlib`, `json`, `os`)
- The module loads without errors
- Both `load_config` and `lookup_user` are importable

### What it doesn't prove yet:
`load_config()` wasn't actually *called* — we just imported it. If you called it now it would raise `ValueError` because `ACTIVE_CUSTOMER` isn't set in `.env` yet. That gets fixed in Chat 5 when we create the customer data files.

---

## How config.py Connects to Everything

This is the dependency map of the full project. Every arrow means "imports from":

```
agent.py  ──────────────────┐
tools.py  ──────────────────┤──→  config.py  ──→  customers/{name}/config.yml
ingest.py ──────────────────┘                 ──→  customers/{name}/known_users.json
                                              ──→  customers/{name}/orders.csv
```

**`agent.py` uses:**
- `cfg.agent.name` — agent persona
- `cfg.agent.welcome_message` — anonymous caller greeting
- `cfg.agent.welcome_known_template` — known caller greeting
- `cfg.llm.model` — which OpenAI model
- `cfg.tts.voice_id` — which ElevenLabs voice
- `lookup_user(cfg, phone)` — identify caller

**`tools.py` uses:**
- `cfg.qdrant.collection` — which vector DB collection to search
- `cfg.data.orders_file` — where to find order data
- `lookup_user(cfg, phone)` — pre-fill email for known callers

**`ingest.py` uses:**
- `cfg.qdrant.collection` — where to store embeddings
- `cfg.data.known_users_file` — validate user data

**The key insight:** `config.py` is the only file that knows about `ACTIVE_CUSTOMER`. Everything else just asks `config.py`. This is the single responsibility principle — one file owns one job.

---

## Architecture Reminder — Where Chat 4 Fits

```
Chat P  ✅  All 7 accounts created, API keys saved
Chat 1  ✅  Project scaffold, uv, pyproject.toml, uv.lock
Chat 2  ✅  Cursor rules — conventions enforced automatically
Chat 3  ✅  LiveKit MCP — live docs inside Cursor
Chat 4  ✅  config.py — CustomerConfig dataclasses, load_config(), lookup_user()
────────────────────────────────────────────────────────────────
Chat 5  →   Customer data files — config.yml, known_users.json, orders.csv
Chat 6  →   agent.py — the voice agent, imports from config.py
Chat 7  →   API key verification
Chat 8  →   SIP configuration (Twilio + LiveKit)
Chat 9  →   First real phone call
```

---

## Handoff Into Chat 5

Paste these 3 bullets at the start of your next Cursor session:

```
1. config.py complete and verified — 6 dataclasses (AgentConfig,
   LLMConfig, TTSConfig, QdrantConfig, DataConfig, CustomerConfig),
   load_config() reads ACTIVE_CUSTOMER → config.yml → typed object,
   lookup_user() does E.164 phone lookup returning dict|None.
   Import test passed: config.py imports OK.

2. Config architecture: config.py is the single front door to all
   customer config. agent.py, tools.py, ingest.py all import from
   config.py only. Switching customers = change ACTIVE_CUSTOMER
   env var only, zero code changes.

3. Ready for Chat 5: create the RockGas customer data files —
   customers/rockgas/config.yml (real values), 
   customers/rockgas/known_users.json (name+email per E.164 number),
   customers/rockgas/orders.csv (sample order data). These files
   are what load_config() and lookup_user() will actually read.
```

---

## Golden Rules — Updated After Chat 4

> **Always `from config import load_config`** — never read config.yml directly in other files

> **config.py is the only file that knows about `ACTIVE_CUSTOMER`** — all other files get values from `cfg`

> **Dataclasses over dictionaries** — typed, autocompletable, self-documenting

> **`yaml.safe_load()` not `yaml.load()`** — security best practice, never executes code

> **`path.exists()` before opening files** — graceful failure over crash

> **`dict.get(key)` not `dict[key]`** — returns None instead of KeyError for optional lookups

> **Always include `@livekit-docs` in Cursor prompts** when writing livekit-agents code

> **Never `pip install`** — always `uv add` or `uv sync`

> **Never commit `.env`** — commit `.env.example` only

> **Always commit `uv.lock`** — Railway needs it for reproducible builds

---

*voice-agent-demo | Chat 4 Learning Notes | February 2026*
