# Chat 5 — Learning Notes
## AI Voice Agent Demo | RockGas Customer Data Files

> **Project:** voice-agent-demo
> **Date:** February 2026
> **Phase:** Day 1 — Foundation (final data layer)
> **Status:** ✅ Complete

---

## Table of Contents
1. [What We Built](#what-we-built)
2. [The Big Picture — Why These Files Matter](#the-big-picture)
3. [Every File Explained](#every-file-explained)
4. [Key Design Decisions Made](#key-design-decisions-made)
5. [How The Data Layer Connects at Runtime](#how-the-data-layer-connects-at-runtime)
6. [The Cursor Prompts — Exact Copies](#the-cursor-prompts)
7. [What Cursor Did — Step by Step](#what-cursor-did)
8. [How To Do It Manually](#how-to-do-it-manually)
9. [Verify It Worked](#verify-it-worked)
10. [Handoff Into Chat 6](#handoff-into-chat-6)

---

## What We Built

Three customer data files that make RockGas a real, working customer in the system. Plus one mid-chat refinement: adding an `account` field to `known_users.json` for clean, unambiguous order matching.

**Files created and updated:**
```
voice-agent-demo/
├── customers/
│   └── rockgas/
│       ├── config.yml          ← NEW — Sam's personality, model config
│       ├── known_users.json    ← NEW — lightweight CRM (name, email, account)
│       ├── orders.csv          ← NEW — sample order data for lookup tool
│       └── docs/               ← empty for now, filled in Chat 11
├── config.py                   ← reads all three files above
└── .cursor/rules/config.mdc   ← UPDATED — account field added to schema
```

**What became possible after Chat 5:**
For the first time, `load_config()` ran end-to-end and returned real data. The full pipeline from `.env` → `config.yml` → typed Python object → user lookup is working and verified.

---

## The Big Picture — Why These Files Matter

### The Foundation is Now Complete

Chats 1-4 built the infrastructure. Chat 5 is where the infrastructure meets real data for the first time.

```
Chat 1: project structure exists
Chat 2: conventions enforced
Chat 3: live docs connected
Chat 4: config.py knows HOW to read data
Chat 5: actual data EXISTS to be read  ← we are here
Chat 6: agent.py uses all of the above to answer calls
```

### The Three-File Data Layer

Every customer demo needs exactly three data files. No more, no less:

| File | Question It Answers | Used By |
|---|---|---|
| `config.yml` | How does this agent behave? | `load_config()` at startup |
| `known_users.json` | Who is this caller? | `lookup_user()` on every call |
| `orders.csv` | What have they ordered? | Order lookup tool (Chat 12) |

A new customer demo = a new folder + these three files. Zero code changes ever.

---

## Every File Explained

### FILE 1: `customers/rockgas/config.yml`

```yaml
agent:
  name: Sam
  welcome_message: "Thank you for calling RockGas. I'm Sam,
    your virtual assistant. How can I help you today?"
  welcome_known_template: "Welcome back {name}! Great to hear
    from you. How can I help you today?"

llm:
  model: gpt-4o-mini
  temperature: 0.7

tts:
  voice_id: JFZ2Sw5TN92wIBOEx7pZ

qdrant:
  collection: rockgas-kb

data:
  known_users_file: customers/rockgas/known_users.json
  orders_file: customers/rockgas/orders.csv
```

**What each section does at runtime:**

| Section | Field | Used where | Effect |
|---|---|---|---|
| `agent` | `name` | `agent.py` | Sam introduces himself |
| `agent` | `welcome_message` | `agent.py` | Anonymous caller greeting |
| `agent` | `welcome_known_template` | `agent.py` | "Welcome back {name}!" |
| `llm` | `model` | `agent.py` → `AgentSession` | Which OpenAI model thinks |
| `llm` | `temperature` | `agent.py` → `AgentSession` | Response creativity (0=rigid, 1=creative) |
| `tts` | `voice_id` | `agent.py` → `AgentSession` | Which ElevenLabs voice speaks |
| `qdrant` | `collection` | `tools.py` | Which KB to search |
| `data` | `known_users_file` | `lookup_user()` | Where to find caller records |
| `data` | `orders_file` | `tools.py` | Where to find order records |

**Why `temperature: 0.7`?**
Temperature controls how creative vs predictable the LLM is:
- `0.0` — robotic, repetitive, very consistent
- `0.7` — natural, slightly varied, still professional
- `1.0` — creative, unpredictable, occasionally odd

For a voice agent, 0.7 is the sweet spot — natural enough to not sound like a robot, consistent enough to not go off-script.

**Why `gpt-4o-mini` not `gpt-4o`?**
- `gpt-4o-mini` is ~10x cheaper and ~2x faster than `gpt-4o`
- For a voice agent, speed matters more than raw intelligence
- Every 100ms of LLM latency = a perceptible pause in conversation
- `gpt-4o-mini` is more than capable for structured tasks like order lookup and KB search
- Can always upgrade per customer by changing one line in config.yml

---

### FILE 2: `customers/rockgas/known_users.json`

```json
{
  "+6421084300555": {
    "name": "Saurabh",
    "email": "saurabhsastra@gmail.com",
    "account": "ACC-001"
  },
  "+6421000002": {
    "name": "James Wilson",
    "email": "james.wilson@example.com",
    "account": "ACC-002"
  },
  "+6421000003": {
    "name": "Sarah Thompson",
    "email": "sarah.thompson@example.com",
    "account": "ACC-003"
  }
}
```

**The schema — three fields, each with a job:**

| Field | Type | Purpose |
|---|---|---|
| `name` | string | Personalise greeting: "Welcome back Saurabh!" |
| `email` | string | Pre-fill email capture tool (Chat 13) |
| `account` | string | Join key to `orders.csv` — find their orders |

**Why the phone number is the key (not a field inside):**
Twilio passes the caller's number via SIP headers in E.164 format. The lookup is `users[caller_phone]` — a single dictionary key lookup, O(1) speed, no searching required. If phone was a field inside, we'd have to loop through every user to find a match — slower and more complex.

**E.164 format — why it matters:**
E.164 is the international standard: `+[country code][number]`, no spaces, no dashes.
- NZ mobile: `+6421XXXXXXX` (+ then 64 then 9 digits = 12 characters total)
- The key must match the format Twilio sends exactly — one character difference = lookup fails = caller treated as anonymous
- Always store and compare in E.164 format

---

### FILE 3: `customers/rockgas/orders.csv`

```csv
order_id,account,name,product,status,eta
RG-1001,ACC-001,Saurabh,45kg LPG Bottle,Delivered,2026-02-20
RG-1002,ACC-001,Saurabh,9kg LPG Bottle,In Transit,2026-03-05
RG-1003,ACC-002,James Wilson,45kg LPG Bottle,Processing,2026-03-10
RG-1004,ACC-003,Sarah Thompson,9kg LPG Bottle,Delivered,2026-02-25
RG-1005,ACC-003,Sarah Thompson,Gas Appliance Service,Scheduled,2026-03-15
```

**Column meanings:**

| Column | Example | Purpose |
|---|---|---|
| `order_id` | RG-1001 | Unique order reference |
| `account` | ACC-001 | **Join key** — matches `account` in `known_users.json` |
| `name` | Saurabh | Human-readable name (secondary, not used for lookup) |
| `product` | 45kg LPG Bottle | What was ordered |
| `status` | In Transit | Current state |
| `eta` | 2026-03-05 | Expected delivery date |

**Why CSV and not a database?**
For a demo, CSV is ideal:
- Human-readable — client can open it in Excel
- Easy to edit — add a realistic order in 30 seconds
- No database setup required
- `pandas` or Python's built-in `csv` module reads it trivially

In a real production deployment, this would be replaced by an API call to the client's actual order management system. The tool interface stays the same — only the data source changes.

---

## Key Design Decisions Made

### Decision 1 — Agent Name: Sam (not Aria)

**Original plan:** Agent named "Aria"
**Problem identified:** Voice ID `JFZ2Sw5TN92wIBOEx7pZ` is a male NZ voice ("Sarn - New Zealand"). Calling a male voice "Aria" creates a jarring mismatch on the call.
**Decision:** Named the agent "Sam" — neutral, professional, very NZ-friendly, matches the voice character.

**Why named agents are enterprise best practice:**
Real enterprise virtual assistants almost universally have names:
- Vodafone → "TOBi"
- ANZ Bank → "Jamie"
- Air New Zealand → "Oscar"
- Amazon → "Alexa"

A name makes the agent feel like a product, not a chatbot. It gives callers something to reference and sets expectations clearly in the first sentence: *"I'm Sam, RockGas's virtual assistant."*

---

### Decision 2 — Account Field Added to `known_users.json`

**Original schema:** `{name, email}` only
**Problem identified:** Order lookup by name is fragile — two customers with the same name would return mixed orders. Name formatting differences ("Saurabh" vs "S. Kumar") could cause misses.
**Decision:** Added `account` field that directly joins to `orders.csv` account column.

**The join that now happens at runtime:**
```
Caller: +6421084300555
    ↓
lookup_user() → account: "ACC-001"
    ↓
Order lookup tool filters orders.csv where account == "ACC-001"
    ↓
Returns: RG-1001 (Delivered) + RG-1002 (In Transit)
    ↓
Sam: "I can see you have two orders — a 45kg bottle 
     delivered on Feb 20, and a 9kg bottle currently 
     in transit, expected March 5."
```

**The upgrade path to real CRM:**
In production, replace `known_users.json` with an API call to the client's CRM (Salesforce, HubSpot, etc.). The `account` field maps directly to their account ID. Zero changes to the tool code.

---

### Decision 3 — `config.mdc` Rule Updated

Updated `.cursor/rules/config.mdc` to document the `account` field so Cursor enforces the correct schema in all future customer data files:

```
known_users.json keys = E.164 phone numbers (+64xxxxxxxxx).
Values must have: name (string), email (string),
account (string — matches account column in orders.csv).
Order lookup joins on account field, not name.
```

This means for any future customer (MBIE, Contact Energy, etc.), Cursor will automatically create `known_users.json` with the correct three fields.

---

## How The Data Layer Connects at Runtime

### Full Call Flow — Data Layer View

```
1. Caller dials +6498869807 (Twilio NZ number)
         ↓
2. Twilio passes caller number (+6421084300555) via SIP
         ↓
3. agent.py receives call, reads ACTIVE_CUSTOMER=rockgas
         ↓
4. load_config() → customers/rockgas/config.yml → CustomerConfig
         ↓
5. lookup_user(cfg, "+6421084300555")
         ↓
6. Reads customers/rockgas/known_users.json
         → Found: {name: "Saurabh", email: "...", account: "ACC-001"}
         ↓
7. Agent greets: "Welcome back Saurabh! Great to hear from you."
         ↓
8. Caller: "Can you check my order status?"
         ↓
9. Order lookup tool reads customers/rockgas/orders.csv
         → Filters: account == "ACC-001"
         → Returns: RG-1001 (Delivered), RG-1002 (In Transit)
         ↓
10. Sam: "I can see you have two orders..."
```

### What Happens for an Unknown Caller

```
1. Caller dials — number NOT in known_users.json
         ↓
2. lookup_user() returns None
         ↓
3. Agent greets: "Thank you for calling RockGas. I'm Sam,
                  your virtual assistant. How can I help?"
         ↓
4. Agent asks for name/account number to look up orders
```

---

## The Cursor Prompts — Exact Copies

### Main Prompt (initial file creation):

**Handoff context:**
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
   customers/rockgas/config.yml, customers/rockgas/known_users.json,
   customers/rockgas/orders.csv. These are what load_config() and
   lookup_user() will actually read at runtime.
```

**Main prompt:**
```
Create three files inside customers/rockgas/.
Do not modify any other files.

FILE 1: customers/rockgas/config.yml
agent:
  name: Sam
  welcome_message: "Thank you for calling RockGas. I'm Sam,
    your virtual assistant. How can I help you today?"
  welcome_known_template: "Welcome back {name}! Great to hear
    from you. How can I help you today?"

llm:
  model: gpt-4o-mini
  temperature: 0.7

tts:
  voice_id: JFZ2Sw5TN92wIBOEx7pZ

qdrant:
  collection: rockgas-kb

data:
  known_users_file: customers/rockgas/known_users.json
  orders_file: customers/rockgas/orders.csv

FILE 2: customers/rockgas/known_users.json
Keys are E.164 phone numbers. Values have name and email only.
Create this exact content:
{
  "+6421084300555": {
    "name": "Saurabh",
    "email": "saurabhsastra@gmail.com"
  },
  "+6421000002": {
    "name": "James Wilson",
    "email": "james.wilson@example.com"
  },
  "+6421000003": {
    "name": "Sarah Thompson",
    "email": "sarah.thompson@example.com"
  }
}

FILE 3: customers/rockgas/orders.csv
Create with these exact columns and sample rows:
order_id,account,name,product,status,eta
RG-1001,ACC-001,Saurabh,45kg LPG Bottle,Delivered,2026-02-20
RG-1002,ACC-001,Saurabh,9kg LPG Bottle,In Transit,2026-03-05
RG-1003,ACC-002,James Wilson,45kg LPG Bottle,Processing,2026-03-10
RG-1004,ACC-003,Sarah Thompson,9kg LPG Bottle,Delivered,2026-02-25
RG-1005,ACC-003,Sarah Thompson,Gas Appliance Service,Scheduled,2026-03-15

After creating all three files, validate that the YAML
is parseable and the JSON is valid. Then show me the
full content of each file.
```

### Update Prompt (account field added):
```
Update two files. Do not modify any other files.

FILE 1: customers/rockgas/known_users.json
Replace the entire file content with:
{
  "+6421084300555": {
    "name": "Saurabh",
    "email": "saurabhsastra@gmail.com",
    "account": "ACC-001"
  },
  "+6421000002": {
    "name": "James Wilson",
    "email": "james.wilson@example.com",
    "account": "ACC-002"
  },
  "+6421000003": {
    "name": "Sarah Thompson",
    "email": "sarah.thompson@example.com",
    "account": "ACC-003"
  }
}

FILE 2: .cursor/rules/config.mdc
Replace the known_users.json schema line with:
known_users.json keys = E.164 phone numbers (+64xxxxxxxxx).
Values must have: name (string), email (string),
account (string — matches account column in orders.csv).
Order lookup joins on account field, not name.
Greeting templates live in config.yml, not in known_users.json.
orders.csv columns: order_id, account, name, product, status, eta.

After completing, show me the full content of both files.
```

---

## What Cursor Did — Step by Step

### Initial Creation

**STEP 1 — Created `config.yml`**
Cursor created the YAML file with all five sections. YAML uses indentation (2 spaces) to show hierarchy — `name` is nested under `agent`, `model` under `llm`, etc. Cursor validated the YAML was parseable before finishing.

**STEP 2 — Created `known_users.json`**
Cursor created the JSON with E.164 phone numbers as keys. JSON uses `{}` for objects, `""` for strings. Cursor validated the JSON was syntactically correct.

**STEP 3 — Created `orders.csv`**
Cursor created the CSV with a header row followed by 5 data rows. CSV uses commas to separate values, newlines to separate rows. No quotes needed unless a value contains a comma.

**STEP 4 — Validated all three**
Cursor confirmed: YAML parsed into 5 top-level keys, JSON valid with 3 E.164 keys, CSV valid with 5 rows and 6 columns.

### Account Field Update

**STEP 1 — Updated `known_users.json`**
Added `"account": "ACC-00X"` to each user entry. Used Inline Edit to replace the file content precisely.

**STEP 2 — Updated `.cursor/rules/config.mdc`**
Updated the schema documentation to include the `account` field and document that order lookup joins on account, not name. This ensures future customer data files are created with the correct schema automatically.

---

## How To Do It Manually

### Create `config.yml`
```bash
cat > customers/rockgas/config.yml << 'EOF'
agent:
  name: Sam
  welcome_message: "Thank you for calling RockGas. I'm Sam, your virtual assistant. How can I help you today?"
  welcome_known_template: "Welcome back {name}! Great to hear from you. How can I help you today?"

llm:
  model: gpt-4o-mini
  temperature: 0.7

tts:
  voice_id: JFZ2Sw5TN92wIBOEx7pZ

qdrant:
  collection: rockgas-kb

data:
  known_users_file: customers/rockgas/known_users.json
  orders_file: customers/rockgas/orders.csv
EOF
```

### Create `known_users.json`
```bash
cat > customers/rockgas/known_users.json << 'EOF'
{
  "+6421084300555": {
    "name": "Saurabh",
    "email": "saurabhsastra@gmail.com",
    "account": "ACC-001"
  },
  "+6421000002": {
    "name": "James Wilson",
    "email": "james.wilson@example.com",
    "account": "ACC-002"
  },
  "+6421000003": {
    "name": "Sarah Thompson",
    "email": "sarah.thompson@example.com",
    "account": "ACC-003"
  }
}
EOF
```

### Create `orders.csv`
```bash
cat > customers/rockgas/orders.csv << 'EOF'
order_id,account,name,product,status,eta
RG-1001,ACC-001,Saurabh,45kg LPG Bottle,Delivered,2026-02-20
RG-1002,ACC-001,Saurabh,9kg LPG Bottle,In Transit,2026-03-05
RG-1003,ACC-002,James Wilson,45kg LPG Bottle,Processing,2026-03-10
RG-1004,ACC-003,Sarah Thompson,9kg LPG Bottle,Delivered,2026-02-25
RG-1005,ACC-003,Sarah Thompson,Gas Appliance Service,Scheduled,2026-03-15
EOF
```

### Verify end-to-end
```bash
uv run python -c "
from config import load_config, lookup_user
cfg = load_config()
print('Agent:', cfg.agent.name)
user = lookup_user(cfg, '+6421084300555')
print('User:', user)
print('Account:', user['account'])
unknown = lookup_user(cfg, '+6400000000')
print('Unknown:', unknown)
"
```

---

## Verify It Worked

### Commands run and confirmed:

**Initial verify:**
```bash
uv run python -c "
from config import load_config, lookup_user
cfg = load_config()
print('Customer:', cfg.customer_id)
print('Agent name:', cfg.agent.name)
print('LLM model:', cfg.llm.model)
print('Voice ID:', cfg.tts.voice_id)
print('Qdrant collection:', cfg.qdrant.collection)
user = lookup_user(cfg, '+6421084300555')
print('Lookup Saurabh:', user)
unknown = lookup_user(cfg, '+6400000000')
print('Lookup unknown:', unknown)
"
```

**Confirmed output ✅:**
```
Customer: rockgas
Agent name: Sam
LLM model: gpt-4o-mini
Voice ID: JFZ2Sw5TN92wIBOEx7pZ
Qdrant collection: rockgas-kb
Lookup Saurabh: {'name': 'Saurabh', 'email': 'saurabhsastra@gmail.com', 'account': 'ACC-001'}
Lookup unknown: None
```

---

## Architecture Reminder — Where Chat 5 Fits

```
Chat P  ✅  All 7 accounts created, API keys saved
Chat 1  ✅  Project scaffold, uv, pyproject.toml, uv.lock
Chat 2  ✅  Cursor rules — conventions enforced automatically
Chat 3  ✅  LiveKit MCP — live docs inside Cursor
Chat 4  ✅  config.py — CustomerConfig, load_config(), lookup_user()
Chat 5  ✅  RockGas data — config.yml, known_users.json, orders.csv
═══════════════════════════════════════════════════════════
FOUNDATION 100% COMPLETE — All infrastructure in place
═══════════════════════════════════════════════════════════
Chat 6  →   agent.py — THE voice agent (the big one)
Chat 7  →   API key verification
Chat 8  →   SIP configuration (Twilio + LiveKit)
Chat 9  →   First real phone call
Chat 10 →   Dockerfile + Railway deployment config
Chat 11 →   Knowledge base documents (RockGas docs)
Chat 12 →   Order lookup tool
Chat 13 →   Email capture tool
Chat 14 →   KB search tool (Qdrant)
Chat 15 →   ingest.py — load KB into Qdrant
Chat 16 →   Deploy to Railway
```

---

## Handoff Into Chat 6

Paste these 3 bullets at the start of your next Cursor session:

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

---

## Golden Rules — Updated After Chat 5

> **Agent name must match voice gender** — mismatch breaks caller trust instantly

> **Named agents are enterprise standard** — name + "virtual assistant" in first sentence

> **E.164 format strictly** — `+64XXXXXXXXX`, no spaces, no dashes, must match Twilio exactly

> **Order lookup joins on `account` not `name`** — account is unambiguous, name is fragile

> **Three files per customer, always** — `config.yml`, `known_users.json`, `orders.csv`

> **New customer = new folder + three files** — zero code changes ever

> **`temperature: 0.7`** — sweet spot for voice agents, natural but not unpredictable

> **`gpt-4o-mini` for speed** — voice agents need low latency more than raw intelligence

> **Always `from config import load_config`** — never read config files directly

> **Always include `@livekit-docs`** — in every Cursor prompt touching agent code

> **Never `pip install`** — always `uv add` or `uv sync`

> **Never commit `.env`** — commit `.env.example` only

---

*voice-agent-demo | Chat 5 Learning Notes | February 2026*
