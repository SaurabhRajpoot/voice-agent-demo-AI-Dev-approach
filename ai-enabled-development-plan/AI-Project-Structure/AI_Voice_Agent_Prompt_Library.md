# AI Voice Agent — Presales Demo & Cursor Learning Lab
## Complete Chat Prompt Library

> **21 Chats · Day-by-Day Build Plan · Copy-Paste Ready**

---

## How to Use This Library

1. Each chat = one session in your Claude Project: **AI Voice Agent — Presales Demo & Cursor Learning Lab**
2. Start each Claude chat by pasting the **🔜 NEXT CHAT HANDOFF** bullets from the previous chat
3. Copy the **📋 PROMPT** exactly into Cursor Agent Mode (Cmd+I) — do not paraphrase
4. Run the **✔️ VERIFY** commands in your terminal before moving to the next chat
5. If Cursor does something unexpected, ask Claude to explain it before you fix it — that is how you learn

---

## Chat Index at a Glance

| # | Title | Phase | Cursor Mode | Est. Time |
|---|-------|-------|-------------|-----------|
| P | Account Setup & API Keys | Pre-Weekend | Browser only | 45 min |
| 1 | Project Scaffold | Day 1 | Agent Mode | 15 min |
| 2 | Cursor Rules (.mdc files) | Day 1 | Agent Mode | 10 min |
| 3 | LiveKit MCP Server | Day 1 | Setup only | 15 min |
| 4 | config.py | Day 1 | Agent Mode | 20 min |
| 5 | Customer Data Files | Day 1 | Agent Mode | 20 min |
| 6 | agent.py v1 + Persona + Latency | Day 1 | Agent Mode + MCP | 30 min |
| 7 | API Keys Verification | Day 1 | Terminal only | 20 min |
| 8 | SIP Configuration | Day 1 | Browser only | 60–90 min |
| 9 | First Real Call | Day 1 | Terminal + fix | 30 min |
| 10 | ingest.py — Qdrant Pipeline | Day 2 | Agent Mode | 30 min |
| 11 | RockGas KB Content | Day 2 | Agent Mode | 30 min |
| 12 | search_knowledge_base Tool | Day 2 | Inline Edit + MCP | 25 min |
| 13 | lookup_order Tool | Day 2 | Inline Edit + MCP | 20 min |
| 14 | capture_email Tool (Gmail) | Day 2 | Inline Edit + MCP | 20 min |
| 15 | Full Demo Rehearsal | Day 2 | Terminal + phone | 45 min |
| 16 | Railway Deploy | Weekend 2 | Agent Mode | 30 min |
| 17 | Sitemap Ingestion Pipeline | Weekend 2 | Inline Edit | 30 min |
| 18 | Second Customer — MBIE | Weekend 2 | Agent Mode | 25 min |
| 19 | Final Latency Polish + README | Weekend 2 | Agent Mode | 30 min |
| 20 | Demo Day Prep & Checklist | Weekend 2 | No coding | 20 min |

---

---

# 🟢 PRE-WEEKEND

---

## Chat P — Account Setup & API Keys

**Phase:** Pre-Weekend — No Cursor Needed

### 🎯 GOAL
All 7 service accounts created and API keys saved safely before you write a single line of code.

### ✅ PREREQUISITES
- A browser, a notepad/password manager, a credit card (most services have free tiers)
- 20–30 min of uninterrupted time

### 🖥️ CURSOR MODE
No Cursor — this is browser-only account setup. Come back to Cursor for Chat 1.

### 📋 PROMPT (copy exactly)
```
No Cursor needed for this chat.

Ask me (Claude) in the new project:

"Walk me through creating all accounts for the
voice agent — LiveKit Cloud, Twilio NZ number,
Deepgram, ElevenLabs, OpenAI, Qdrant Cloud,
and Railway — one at a time. Explain what each
service does in the pipeline as we go."
```

### ✔️ VERIFY
```
Notepad has 7 API keys filled in
Twilio NZ number purchased and active
LiveKit Cloud project created
```

### 🔜 NEXT CHAT HANDOFF
1. All 7 API keys saved: LIVEKIT_URL, LIVEKIT_API_KEY, LIVEKIT_API_SECRET, DEEPGRAM_API_KEY, ELEVEN_API_KEY, OPENAI_API_KEY, QDRANT_URL, QDRANT_API_KEY
2. Twilio NZ number purchased: +64xxxxxxxxx
3. LiveKit Cloud project name and SIP section located in dashboard

---

---

# 🔵 DAY 1 — FOUNDATION
**Goal: Real call working by end of day**

---

## Chat 1 — Project Scaffold

**Phase:** Day 1 — Foundation

### 🎯 GOAL
A properly structured Python project exists on disk, uv is initialised, all dependencies are declared, and the folder structure matches the implementation plan.

### ✅ PREREQUISITES
- Chat P complete — all API keys saved
- uv installed on MacBook: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Empty folder created: `mkdir rockgas-voice-agent && cd rockgas-voice-agent`
- Cursor open on that empty folder

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** — creating multiple files across a folder structure. This is exactly what Agent Mode is built for: multi-file, multi-step work in one instruction.

### 📋 PROMPT (copy exactly)
```
I am starting a new Python project called rockgas-voice-agent.
Create the following using uv (not pip):

1. Run: uv init --python 3.12
2. Create this folder structure:
   customers/rockgas/docs/
   customers/rockgas/
   tests/
3. Create pyproject.toml with these dependencies:
   livekit-agents~=1.0, livekit-plugins-deepgram,
   livekit-plugins-elevenlabs, livekit-plugins-openai,
   livekit-plugins-silero, qdrant-client, openai,
   pyyaml, python-dotenv
   Dev deps: pytest, pytest-asyncio
4. Create .env.example with blank keys:
   ACTIVE_CUSTOMER, LIVEKIT_URL, LIVEKIT_API_KEY,
   LIVEKIT_API_SECRET, DEEPGRAM_API_KEY, ELEVEN_API_KEY,
   OPENAI_API_KEY, QDRANT_URL, QDRANT_API_KEY,
   GMAIL_APP_PASSWORD
5. Create .gitignore (Python standard + .env)
6. Create AGENTS.md with project stack summary

Do not modify any other files.
After completing, show me all file contents.
```

### ✔️ VERIFY
```bash
uv sync
ls -la                    # should show .venv created
uv run python --version   # should show 3.12.x
```

### 🔜 NEXT CHAT HANDOFF
1. Project scaffolded with uv, Python 3.12, pyproject.toml, .env.example, AGENTS.md, .gitignore
2. Folder structure: rockgas-voice-agent/ with customers/rockgas/docs/ and tests/
3. uv sync ran successfully — .venv created and all packages installed

---

## Chat 2 — Cursor Rules Setup (.mdc files)

**Phase:** Day 1 — Foundation

### 🎯 GOAL
`.cursor/rules/` directory created with three .mdc rules files that make Cursor smarter about this specific project for every future chat.

### ✅ PREREQUISITES
- Chat 1 complete
- Project folder open in Cursor

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** — creating multiple new files in a new directory. After this chat, every future Cursor session will automatically load the relevant rules.

### 📋 PROMPT (copy exactly)
```
Create the .cursor/rules/ directory with three files.
Do not modify any existing files.

FILE 1: .cursor/rules/core.mdc
---
alwaysApply: true
---
Use uv for all packages. Never pip install.
Python 3.12, async/await everywhere.
Type hints + docstrings on all functions.
No langchain, no langgraph.
Config via: from config import load_config
Never hardcode customer values — use config.yml.
Do not modify files outside current task scope.
Show full file content after any edit.

FILE 2: .cursor/rules/livekit.mdc
---
description: LiveKit Agents 1.x patterns
globs: agent.py, tools.py, src/**/*.py
alwaysApply: false
---
Use AgentSession + Agent (NOT VoicePipelineAgent).
Use @function_tool decorator (NOT @llm.ai_callable).
Tools: methods on Agent class OR Agent(tools=[...]).
Tool docstrings = LLM tool descriptions. Write carefully.
Always use @livekit-docs MCP for API questions.

FILE 3: .cursor/rules/config.mdc
---
description: Config and data file conventions
globs: customers/**/*.yml, customers/**/*.json
alwaysApply: false
---
known_users.json keys = E.164 phone numbers.
Values must have: name, preferred_greeting,
email (optional), last_intent (optional).
orders.csv columns: order_id, account, name,
product, status, eta.

After creating all three, show me each file.
```

### ✔️ VERIFY
```bash
ls .cursor/rules/   # should show 3 .mdc files
# Open any .mdc file in Cursor — check frontmatter is valid YAML
```

### 🔜 NEXT CHAT HANDOFF
1. `.cursor/rules/` created with core.mdc (alwaysApply), livekit.mdc (globs: agent.py, tools.py), config.mdc (globs: customers/**)
2. core.mdc is always loaded in every Cursor Agent session — enforces uv, no-pip, no-langchain rules automatically
3. livekit.mdc auto-loads whenever agent.py or tools.py is open — enforces AgentSession + @function_tool pattern

---

## Chat 3 — LiveKit MCP Server

**Phase:** Day 1 — Foundation

### 🎯 GOAL
LiveKit Docs MCP server connected to Cursor, verified working, and understood — so all future agent/tool code is written against live docs, not hallucinated patterns.

### ✅ PREREQUISITES
- Chat 2 complete
- Cursor open

### 🖥️ CURSOR MODE
No Cursor coding in this chat — it's configuration + learning. MCP = giving Cursor a live connection to a knowledge source, like adding a reference book to a researcher's desk.

### 📋 PROMPT (copy exactly)
```
No code to write — this is a setup and learning chat.

Ask me (Claude) in the new project:

"Help me install the LiveKit Docs MCP server in Cursor.
Walk me through:
1. Going to docs.livekit.io/mcp and clicking
   the Cursor install button
2. What MCP (Model Context Protocol) is —
   explain it with an analogy
3. How to verify it's working in Cursor
4. How to use @livekit-docs in a Cursor prompt
5. What the fallback is if MCP isn't available

Then test it: ask Cursor using @livekit-docs:
'What is the correct import for AgentSession
and @function_tool in livekit-agents 1.x?'"
```

### ✔️ VERIFY
```
In Cursor settings, MCP section shows livekit-docs connected
@livekit-docs responds in a Cursor Agent prompt with real API info
The import answer matches:
  from livekit.agents import AgentSession, Agent, function_tool
```

### 🔜 NEXT CHAT HANDOFF
1. LiveKit Docs MCP server installed at https://docs.livekit.io/mcp — connected in Cursor MCP settings
2. All future agent/tool Cursor prompts should include @livekit-docs to pull live API docs into context
3. Fallback: append .md to any docs.livekit.io URL to get a pasteable Markdown version of that page

---

## Chat 4 — config.py

**Phase:** Day 1 — Foundation

### 🎯 GOAL
`config.py` loads `customers/rockgas/config.yml` into typed Python dataclasses, and `load_config()` works correctly from the terminal.

### ✅ PREREQUISITES
- Chat 3 complete
- Cursor open on project

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** — creating a new file with multiple interconnected classes. Inline Edit would be too limiting here.

### 📋 PROMPT (copy exactly)
```
Create config.py in the project root.
Do not modify any other files.
Allowed imports: dataclasses, os, pathlib, yaml,
python-dotenv.

The file must:
1. Define dataclasses for: VoiceConfig, AgentConfig,
   QdrantConfig, EmailConfig, CustomerConfig
2. CustomerConfig must have a lookup_user(phone: str)
   method that loads known_users.json and returns
   the user dict or None
3. load_config() reads ACTIVE_CUSTOMER from env
   (default 'rockgas'), loads
   customers/{name}/config.yml, returns CustomerConfig
4. All fields have type hints and docstrings

After writing, show me the full file content.
```

### ✔️ VERIFY
```bash
uv run python -c "from config import load_config; c = load_config(); print(c.customer_id)"
# Right now: should raise FileNotFoundError — correct, config.yml doesn't exist yet
# After Chat 5: expected output: rockgas
```

### 🔜 NEXT CHAT HANDOFF
1. config.py created with CustomerConfig dataclass, load_config() reads ACTIVE_CUSTOMER env var
2. lookup_user(phone) method will read known_users.json when it exists
3. The test command fails with FileNotFoundError — expected; config.yml gets created in Chat 5

---

## Chat 5 — Customer Data Files

**Phase:** Day 1 — Foundation

### 🎯 GOAL
`customers/rockgas/config.yml`, `known_users.json`, and `orders.csv` created with realistic RockGas demo data — and `load_config()` passes its first real test.

### ✅ PREREQUISITES
- Chat 4 complete
- Your mobile number ready (will be added as a known user)
- Cursor open on project

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** — creating multiple files in a subdirectory. config.mdc rules will auto-load because of the glob pattern, enforcing the correct schema.

### 📋 PROMPT (copy exactly)
```
Create three files. Do not modify any other files.

FILE 1: customers/rockgas/config.yml
Schema: customer_id, display_name, voice section
(elevenlabs_voice_id: 'EXAVITQu4vr4xnSDxMaL',
elevenlabs_model: 'eleven_turbo_v2_5',
deepgram_language: 'en-NZ', deepgram_model: 'nova-3'),
agent section (welcome_message, welcome_known_template
with {name} placeholder, persona as multiline string
for a helpful RockGas NZ LPG supplier assistant,
email_prompt, goodbye_message),
qdrant section (collection_name: 'rockgas_kb'),
email section (smtp_from, subject_template).

FILE 2: customers/rockgas/known_users.json
Add 3 realistic NZ contacts. Schema per user:
{ name, preferred_greeting, email, last_intent }
Add MY number as a known user: +6421XXXXXXX
(I will fill in my real number after).

FILE 3: customers/rockgas/orders.csv
Columns: order_id, account, name, product,
status, eta. Add 5 realistic rows.

Validate all three files are parseable.
Show me each file after creating it.
```

### ✔️ VERIFY
```bash
uv run python -c "from config import load_config; c = load_config(); print(c.display_name)"
# Expected: RockGas

uv run python -c "from config import load_config; c = load_config(); print(c.lookup_user('+6421XXXXXXX'))"
# Expected: your user dict
```

### 🔜 NEXT CHAT HANDOFF
1. customers/rockgas/config.yml, known_users.json, orders.csv all created and parseable
2. load_config() returns RockGas config correctly — Chat 4 test now passes
3. My mobile number is in known_users.json — personalised greeting will trigger when I call

---

## Chat 6 — agent.py v1 + Persona + Basic Latency Tuning

**Phase:** Day 1 — Foundation

### 🎯 GOAL
`agent.py` runs locally using livekit-agents 1.x (`AgentSession` + `Agent`), loads the RockGas persona from config, and is tuned for minimum perceived latency from the start.

### ✅ PREREQUISITES
- Chats 1–5 complete
- `.env` populated with real API keys
- `uv sync` run successfully
- Cursor open on project with @livekit-docs MCP active

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I) with @livekit-docs in the prompt** — the most important prompt of the build. The MCP server pulls live API docs so Cursor writes correct 1.x code, not hallucinated 0.x patterns.

### 📋 PROMPT (copy exactly)
```
Using @livekit-docs, create agent.py in the project root.
Do not modify any other files.
Use livekit-agents 1.x API only.
Allowed imports: livekit.agents, livekit.plugins
(deepgram, elevenlabs, openai, silero), config,
os, dotenv.

The agent must:
1. Use AgentSession + Agent pattern (NOT VoicePipelineAgent)
2. Load config via load_config()
3. Extract caller phone from SIP participant attributes
   (sip.callFrom), call lookup_user(), use
   preferred_greeting if known, welcome_message if not
4. Pass persona from config.agent.persona as
   Agent(instructions=...)
5. Use Deepgram Nova-3, GPT-4o-mini, ElevenLabs
   Turbo v2.5, Silero VAD
6. Set in AgentSession: min_endpointing_delay=0.3,
   allow_interruptions=True
7. Use agents.cli.run_app() as entry point

Show me the full file after writing.
```

### ✔️ VERIFY
```bash
uv run python agent.py dev
# Expected: 'Starting agent worker' in logs, no import errors
# If LIVEKIT credentials wrong: connection error (expected at this stage)
```

### 🔜 NEXT CHAT HANDOFF
1. agent.py created using AgentSession + Agent, loads persona from config.yml, handles known/unknown callers
2. Latency settings baked in from day 1: min_endpointing_delay=0.3, allow_interruptions=True
3. `uv run python agent.py dev` starts without errors — ready for SIP connection in Chat 8

---

## Chat 7 — API Keys Verification

**Phase:** Day 1 — Foundation

### 🎯 GOAL
Every API key in `.env` is verified to work before the first real call — no silent failures on demo day.

### ✅ PREREQUISITES
- Chat 6 complete
- `.env` file populated with all real keys

### 🖥️ CURSOR MODE
No Cursor coding — this is verification only. Run each test in your MacBook terminal. If a key fails, return to the account dashboard and regenerate it.

### 📋 PROMPT (copy exactly)
```
No new files. Help me verify each API key works.

Ask me (Claude) in the new project:

"I have all my API keys in .env. Walk me through
a verification test for each one:
- LiveKit: connect test
- Deepgram: transcribe a short audio clip
- ElevenLabs: generate a 5-word TTS sample
- OpenAI: send a 1-token completion
- Qdrant: ping the cluster

Give me the exact uv run python -c command
for each test, explain what a passing result
looks like vs a failing one."
```

### ✔️ VERIFY
```
All 5 API test commands return successful responses
No AuthenticationError or ConnectionError in any test
Qdrant cluster URL and key are correct
```

### 🔜 NEXT CHAT HANDOFF
1. All API keys verified working: LiveKit, Deepgram, ElevenLabs, OpenAI, Qdrant
2. .env is complete and correct — ready for first real call
3. Any failed key is fixed before proceeding to SIP setup

---

## Chat 8 — SIP Configuration (Twilio + LiveKit)

**Phase:** Day 1 — Foundation

### 🎯 GOAL
Twilio number connects to LiveKit Cloud via Elastic SIP Trunk — verified by a real call reaching the LiveKit dashboard (even if agent doesn't answer yet).

### ✅ PREREQUISITES
- Chat 7 complete — all API keys verified
- LiveKit Cloud dashboard open
- Twilio console open
- Browser on both simultaneously

### 🖥️ CURSOR MODE
No Cursor — manual dashboard configuration. Budget 60–90 min for this step. The SIP URI format and credential matching between Twilio and LiveKit is the trickiest part.

### 📋 PROMPT (copy exactly)
```
No Cursor needed — this is manual browser configuration.

Ask me (Claude) in the new project:

"Walk me through the Twilio + LiveKit SIP setup
step by step, click by click:
1. Configure LiveKit Cloud SIP Inbound Trunk
2. Create Twilio Elastic SIP Trunk
3. Connect my Twilio number to the trunk
4. What exactly happens when someone dials my number
5. Verification step — how do I confirm it's working
   before any agent code is involved
Explain what SIP is using a simple analogy."
```

### ✔️ VERIFY
```
Call Twilio number from mobile
LiveKit Cloud dashboard > Rooms shows an incoming session attempt
Call may disconnect (no agent yet) — that is OK
The SIP handshake is what we're verifying
```

### 🔜 NEXT CHAT HANDOFF
1. Twilio Elastic SIP Trunk created and connected to LiveKit SIP Inbound Trunk URI
2. Twilio NZ number configured to route to SIP trunk
3. Verified: incoming call creates a session event in LiveKit dashboard

---

## Chat 9 — First Real Call

**Phase:** Day 1 — Foundation

### 🎯 GOAL
You dial the Twilio number from your mobile, the agent answers, speaks the greeting, and handles a 2-turn conversation — running entirely on your MacBook.

### ✅ PREREQUISITES
- Chats 1–8 complete
- SIP trunk verified in Chat 8
- `.env` complete
- Terminal open in project folder

### 🖥️ CURSOR MODE
Terminal-first chat. Keep Cursor open to fix bugs inline (Cmd+K on the specific line) if something errors. The agent connects OUT to LiveKit Cloud — no inbound ports needed on MacBook.

### 📋 PROMPT (copy exactly)
```
No new code (unless there's a bug to fix).

Ask me (Claude) in the new project:

"I'm about to run my agent for the first time.
Walk me through:
1. The exact command to start the agent locally
2. What I should see in terminal logs when it's ready
3. What success looks like when I call the number
4. What to do if the call connects but no audio
5. What to do if I get an error in the logs

Then stay with me while I make the first call and
help me debug anything that goes wrong."
```

### ✔️ VERIFY
```bash
uv run python agent.py dev
# Call from mobile → hear the RockGas greeting
# Ask a question → get a response (LLM knowledge, no RAG yet)
# Say something mid-response → agent stops and listens (interruptions working)
```

### 🔜 NEXT CHAT HANDOFF
1. First real call working — agent answers, greets, handles multi-turn conversation
2. Running locally with: `uv run python agent.py dev`
3. Interruptions working, latency feels natural — ready to add RAG and tools on Day 2

---

---

# 🔷 DAY 2 — INTELLIGENCE
**Goal: Full RAG + tools demo-ready**

---

## Chat 10 — ingest.py — Qdrant Ingestion Pipeline

**Phase:** Day 2 — Intelligence

### 🎯 GOAL
`ingest.py` chunks text files from `customers/rockgas/docs/`, embeds them with OpenAI, and stores them in a Qdrant Cloud collection — verified by visible chunks in the dashboard.

### ✅ PREREQUISITES
- Day 1 complete (Chats 1–9)
- Qdrant Cloud cluster running
- At least one .txt file in customers/rockgas/docs/
- Cursor open on project

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** — new file with async logic and multiple libraries. After Chat 11 adds the real KB content, you'll re-run this script to populate the real demo data.

### 📋 PROMPT (copy exactly)
```
Create ingest.py in the project root.
Do not modify any other files.
Allowed imports: qdrant_client, openai (AsyncOpenAI),
pathlib, uuid, os, dotenv, config, tiktoken.

The script must:
1. Load config via load_config()
2. Create Qdrant collection if not exists:
   name = cfg.qdrant.collection_name,
   vector size = 1536, distance = COSINE
3. Read all .txt and .md files from
   customers/{customer_id}/docs/
4. Chunk each file: ~400 tokens, 50 token overlap
5. Embed each chunk with OpenAI text-embedding-3-small
6. Upsert to Qdrant with payload:
   { text, source, customer_id }
7. Print progress: 'Ingested X chunks into {collection_name}'

Entry point: if __name__ == '__main__': asyncio.run(main())
Show me the full file after writing.
```

### ✔️ VERIFY
```bash
uv run python ingest.py
# Expected: 'Ingested N chunks into rockgas_kb'
# Verify in Qdrant Cloud dashboard: rockgas_kb collection shows N vectors
```

### 🔜 NEXT CHAT HANDOFF
1. ingest.py created — chunks docs, embeds with text-embedding-3-small, stores in Qdrant rockgas_kb collection
2. Qdrant Cloud dashboard shows rockgas_kb collection with chunks from docs/
3. Ready to add real KB content in Chat 11, then wire search tool in Chat 12

---

## Chat 11 — RockGas Knowledge Base Content

**Phase:** Day 2 — Intelligence

### 🎯 GOAL
5+ text files in `customers/rockgas/docs/` covering real RockGas topics — ready to ingest so the agent answers from a real KB, not LLM guesswork.

### ✅ PREREQUISITES
- Chat 10 complete
- Cursor open

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** — creating multiple content files. You can ask Cursor to make content more specific after it drafts it. Inline Edit (Cmd+K) is good for refining a specific file once it exists.

### 📋 PROMPT (copy exactly)
```
Create 5 text files in customers/rockgas/docs/.
Do not modify any other files.
Content must be realistic NZ LPG supplier information.

FILE 1: docs/pricing.txt
LPG bottle sizes (9kg, 19kg, 45kg), current prices,
rental fees, swap vs refill options.

FILE 2: docs/delivery.txt
Delivery areas (NZ regions), lead times, rural
surcharges, how to track an order.

FILE 3: docs/safety.txt
Gas cylinder safety, storage rules, what to do
if you smell gas, pressure regulator info.

FILE 4: docs/accounts.txt
How to open an account, direct debit setup,
online account management, contact details.

FILE 5: docs/faq.txt
10 common customer questions with clear answers.

Make content specific enough that a RAG search
will return distinct, useful chunks.
Show me each file after creating it.
```

### ✔️ VERIFY
```bash
uv run python ingest.py
# Expected: 'Ingested N chunks into rockgas_kb' (N should be 30+ chunks)
# Qdrant dashboard shows 30+ vectors in rockgas_kb
```

### 🔜 NEXT CHAT HANDOFF
1. 5 KB files created in docs/ — pricing, delivery, safety, accounts, FAQ
2. ingest.py re-run with real content — 30+ chunks now in rockgas_kb Qdrant collection
3. Ready to build search_knowledge_base tool in Chat 12

---

## Chat 12 — search_knowledge_base Tool

**Phase:** Day 2 — Intelligence

### 🎯 GOAL
Agent answers questions from the RockGas KB (not LLM memory) — test proves answer comes from a real Qdrant chunk.

### ✅ PREREQUISITES
- Chats 10–11 complete
- Qdrant has 30+ chunks
- Cursor open with @livekit-docs MCP active

### 🖥️ CURSOR MODE
**Inline Edit (Cmd+K)** with agent.py open — adding a method to an existing class. Use @livekit-docs so Cursor gets the @function_tool syntax exactly right.

### 📋 PROMPT (copy exactly)
```
Using @livekit-docs, add a search_knowledge_base
tool to agent.py. Only modify agent.py.

Add a @function_tool method to the Agent class:

@function_tool
async def search_knowledge_base(self, query: str) -> str:
    '''Search the RockGas knowledge base for product,
    pricing, delivery or safety information. Call this
    whenever the user asks about RockGas products,
    prices, delivery or services.'''
    # embed query with OpenAI text-embedding-3-small
    # search cfg.qdrant.collection_name, top 3,
    # score_threshold 0.72
    # return joined chunk texts or fallback message

Initialise QdrantClient and AsyncOpenAI once at
module level (not per call).
Load cfg from load_config() at module level.
Show me the full updated agent.py after editing.
```

### ✔️ VERIFY
```bash
uv run python agent.py dev
# Call number, ask: 'What is the price of a 45kg LPG bottle?'
# Expected: answer from KB chunk, not generic LLM answer
# Check terminal logs — should show tool call: search_knowledge_base
```

### 🔜 NEXT CHAT HANDOFF
1. search_knowledge_base @function_tool added to Agent class in agent.py
2. QdrantClient and OpenAI client initialised at module level — not per call
3. Test passed: pricing question returns answer from rockgas_kb chunk

---

## Chat 13 — lookup_order Tool

**Phase:** Day 2 — Intelligence

### 🎯 GOAL
Agent looks up a real order from `orders.csv` and tells the caller their delivery status.

### ✅ PREREQUISITES
- Chat 12 complete
- orders.csv has 5+ rows
- Cursor open

### 🖥️ CURSOR MODE
**Inline Edit (Cmd+K)** with agent.py open — adding another method to the Agent class. Small, focused change. The key lesson here is how the tool docstring determines when the LLM calls this vs the KB search.

### 📋 PROMPT (copy exactly)
```
Using @livekit-docs, add a lookup_order tool
to agent.py. Only modify agent.py.

1. At module level, load orders.csv into memory:
   import csv
   orders = list(csv.DictReader(open(
     f'customers/{cfg.customer_id}/orders.csv')))

2. Add @function_tool method to Agent class:

@function_tool
async def lookup_order(self, identifier: str) -> str:
    '''Look up a delivery order by customer name
    or account number. Call this when the user
    asks about their order, delivery, or account.'''
    # search orders list case-insensitively
    # on 'name' and 'account' fields
    # return order status string or not-found message

Show me the full updated agent.py.
```

### ✔️ VERIFY
```bash
uv run python agent.py dev
# Call, ask: 'Where is my order? My name is [name from orders.csv]'
# Expected: agent returns order status and ETA
# Check logs for lookup_order tool call
```

### 🔜 NEXT CHAT HANDOFF
1. lookup_order @function_tool added — reads from orders.csv loaded at startup
2. Agent correctly routes order questions to this tool and KB questions to search_knowledge_base
3. Two tools working simultaneously — LLM decides which to call based on docstrings

---

## Chat 14 — capture_email Tool (Real Gmail SMTP)

**Phase:** Day 2 — Intelligence

### 🎯 GOAL
Agent captures email address at end of call and sends a real summary email to the caller.

### ✅ PREREQUISITES
- Chat 13 complete
- Gmail account ready
- Gmail App Password created (myaccount.google.com > Security > App Passwords)
- GMAIL_APP_PASSWORD in .env

### 🖥️ CURSOR MODE
**Inline Edit (Cmd+K)** — adding one more method to the existing Agent class. Short, focused. The Gmail App Password avoids OAuth complexity — it's a 16-character password specific to this app.

### 📋 PROMPT (copy exactly)
```
Using @livekit-docs, add a capture_email tool
to agent.py. Only modify agent.py.
Allowed new imports: smtplib, email.mime.text.

Add @function_tool method to Agent class:

@function_tool
async def capture_email(self,
    email_address: str,
    conversation_summary: str) -> str:
    '''Capture the caller's email address and send
    a conversation summary. Call this ONLY when
    the caller explicitly agrees to share their
    email and wants a summary sent.'''
    # send via Gmail SMTP SSL port 465
    # from: cfg.email.smtp_from
    # subject: cfg.email.subject_template
    # body: conversation_summary
    # return confirmation string

Use GMAIL_APP_PASSWORD from os.getenv().
Wrap in try/except — return error message if fails.
Show me the full updated agent.py.
```

### ✔️ VERIFY
```bash
uv run python agent.py dev
# Call, have conversation, at end say: 'Can you send me a summary?'
# Give email address when asked
# Check inbox — summary email should arrive within 30 seconds
```

### 🔜 NEXT CHAT HANDOFF
1. capture_email @function_tool added — sends real Gmail SMTP email on caller consent
2. All three tools working: search_knowledge_base, lookup_order, capture_email
3. Full demo feature set complete — ready for end-to-end rehearsal in Chat 15

---

## Chat 15 — Full Demo Rehearsal

**Phase:** Day 2 — Intelligence

### 🎯 GOAL
Three demo scenarios run end-to-end smoothly, rough edges fixed, demo is ready to show.

### ✅ PREREQUISITES
- Chats 1–14 complete
- Agent running locally
- Mobile phone ready
- orders.csv has your name as a test entry

### 🖥️ CURSOR MODE
Terminal + phone — this is demo rehearsal, not coding. Use Inline Edit (Cmd+K) for any quick fixes that come up. Run each scenario twice to build muscle memory for the live demo.

### 📋 PROMPT (copy exactly)
```
No new files unless fixing a bug.

Ask me (Claude) in the new project:

"Run me through three demo scenarios and help
me evaluate each one:

SCENARIO A — Unknown caller:
Generic greeting → ask pricing question → KB answers
→ offer email → capture it.

SCENARIO B — Known caller (my mobile):
Personalised greeting using preferred_greeting →
ask about my order → lookup_order returns status.

SCENARIO C — Out of KB scope:
Ask something the KB doesn't cover →
agent admits it doesn't know →
offers to have team follow up → captures email.

After each scenario, tell me what felt rough
and help me fix it. Then give me the
demo day checklist."
```

### ✔️ VERIFY
```
All 3 scenarios complete without errors or dead air
Tool calls visible in terminal logs for each relevant question
Email arrives in inbox after Scenario A and C
Personalised greeting triggers on Scenario B
```

### 🔜 NEXT CHAT HANDOFF
1. All 3 demo scenarios tested and working end-to-end
2. Rough edges fixed — agent feels natural, latency is acceptable
3. Demo is ready to run from MacBook — Railway deploy is optional next step

---

---

# 🟣 WEEKEND 2 — PLATFORM
**Goal: Reusable, cloud-hosted, documented**

---

## Chat 16 — Railway Deploy

**Phase:** Weekend 2 — Platform

### 🎯 GOAL
Agent runs on Railway without your MacBook — calls work identically from the cloud.

### ✅ PREREQUISITES
- Chats 1–15 complete
- Railway account created
- railway CLI installed: `npm install -g @railway/cli`
- Cursor open on project

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** — creating two new infrastructure files. The Dockerfile is the packaging layer; railway.toml tells Railway how to run it. Key lesson: uv.lock committed to git = Railway knows exactly what to install.

### 📋 PROMPT (copy exactly)
```
Create Dockerfile and railway.toml in project root.
Do not modify any other files.

Dockerfile must:
1. Base image: ghcr.io/astral-sh/uv:python3.12-bookworm-slim
2. Set WORKDIR /app
3. Copy pyproject.toml and uv.lock first
4. RUN uv sync --locked --no-dev
5. COPY . .
6. CMD ["uv", "run", "python", "agent.py", "start"]

railway.toml must define:
- build command
- start command
- healthcheck

Show me both files after creating them.

Then give me the exact Railway CLI commands
to deploy and set all environment variables.
```

### ✔️ VERIFY
```bash
railway up                  # deploy succeeds
railway logs --tail         # no errors in startup
# Call Twilio number → agent answers from Railway (not MacBook)
uv run python agent.py dev  # still works locally (unchanged)
```

### 🔜 NEXT CHAT HANDOFF
1. Dockerfile and railway.toml created — agent deployed to Railway
2. All env vars set in Railway dashboard (same as .env but cloud-hosted)
3. Calls work without MacBook — Railway keeps process always-on on Hobby plan

---

## Chat 17 — Sitemap Ingestion Pipeline

**Phase:** Weekend 2 — Platform

### 🎯 GOAL
`ingest.py` extended to automatically scrape content from a website sitemap URL — making new customer onboarding self-service.

### ✅ PREREQUISITES
- Chat 16 complete
- Cursor open on project

### 🖥️ CURSOR MODE
**Inline Edit (Cmd+K)** — extending an existing file. You know ingest.py well by now. This is the chat where the whole multi-customer vision becomes real.

### 📋 PROMPT (copy exactly)
```
Extend ingest.py to add sitemap scraping.
Only modify ingest.py.
Allowed new imports: httpx, bs4 (beautifulsoup4),
xml.etree.ElementTree.
Add both to pyproject.toml dependencies.

Add function: async def ingest_from_sitemap(
    sitemap_url: str, customer_id: str) -> None

It must:
1. Fetch sitemap_url, parse XML, extract <loc> URLs
2. For each URL: fetch page, extract main text
   (remove nav, header, footer, scripts)
3. Chunk, embed, upsert to Qdrant
   (same chunking as file ingestion)
4. Print: 'Scraped {url}: {n} chunks'

Add optional sitemap_url field to config.yml schema.
Update main() to call ingest_from_sitemap if
sitemap_url is set in config.

Show me the full updated ingest.py.
```

### ✔️ VERIFY
```bash
# Add to customers/rockgas/config.yml:
# sitemap_url: https://rockgas.co.nz/sitemap.xml
uv run python ingest.py
# Expected: scraped pages + file chunks both in Qdrant
# Test: ask about something only on the website
```

### 🔜 NEXT CHAT HANDOFF
1. ingest.py supports sitemap scraping via sitemap_url in config.yml
2. New customer onboarding = point sitemap_url at their website + run ingest.py
3. No code changes needed for new customers — fully self-service

---

## Chat 18 — Second Customer — MBIE Config

**Phase:** Weekend 2 — Platform

### 🎯 GOAL
MBIE customer config added, demo switches with one env var change, collections stay completely isolated.

### ✅ PREREQUISITES
- Chat 17 complete
- Cursor open on project

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** — creating a new customer folder from scratch. This proves the whole configurable-by-design architecture works. No code touched — only new data files.

### 📋 PROMPT (copy exactly)
```
Create customers/mbie/ folder and data files.
Do not modify any existing files.

Create customers/mbie/config.yml with:
- customer_id: mbie
- Different persona (NZ Government business
  information assistant)
- Different voice (different ElevenLabs voice ID)
- Different welcome_message
- qdrant collection_name: mbie_kb
- Different email subject

Create customers/mbie/known_users.json
with 2 sample contacts.

Create customers/mbie/docs/ with 2 sample
text files (business registration FAQ,
employment law basics).

Then show me the exact commands to:
1. Ingest MBIE KB
2. Switch demo from RockGas to MBIE
3. Verify MBIE questions don't return RockGas answers

Show all new files after creating them.
```

### ✔️ VERIFY
```bash
ACTIVE_CUSTOMER=mbie uv run python agent.py dev
# Call → hear MBIE greeting (not RockGas)

ACTIVE_CUSTOMER=rockgas uv run python agent.py dev
# Call → hear RockGas greeting (not MBIE) — isolation confirmed
```

### 🔜 NEXT CHAT HANDOFF
1. MBIE customer config, known_users.json, docs/ created — second customer is live
2. Switching demo = ACTIVE_CUSTOMER=rockgas or ACTIVE_CUSTOMER=mbie — zero code changes
3. Qdrant isolation confirmed: rockgas_kb and mbie_kb are completely separate collections

---

## Chat 19 — Final Latency Polish + README

**Phase:** Weekend 2 — Platform

### 🎯 GOAL
End-to-end latency under 1 second perceived, and a README.md that lets someone else set up this project from scratch.

### ✅ PREREQUISITES
- Chats 1–18 complete
- Cursor open

### 🖥️ CURSOR MODE
**Agent Mode (Cmd+I)** for README creation, **Inline Edit (Cmd+K)** for system prompt trimming. The README is your own documentation — you'll thank yourself later.

### 📋 PROMPT (copy exactly)
```
Two tasks. Handle sequentially.

TASK 1: Create latency_check.py to measure
approximate TTFB (time to first audio byte).
Print: STT time, LLM first token time, TTS time.
Do not modify agent.py.

TASK 2: Create README.md in project root.
Must cover:
- What this project does (2 sentences)
- Prerequisites (accounts + keys needed)
- Quick start (exact commands from clone to first call)
- Adding a new customer (4-step process)
- Architecture diagram (ASCII)
- Troubleshooting (top 5 issues)

Show me both files after creating.

Then: review agent.py system prompt in config.yml
and suggest any changes to reduce token count
while keeping demo quality.
```

### ✔️ VERIFY
```bash
uv run python latency_check.py   # shows component timings
cat README.md                    # readable, complete, no placeholder text
git add . && git commit -m 'final build'
```

### 🔜 NEXT CHAT HANDOFF
1. README.md created — project is self-documenting
2. Latency measured — each component timing visible
3. System prompt reviewed for token efficiency

---

## Chat 20 — Demo Day Prep & Checklist

**Phase:** Weekend 2 — Platform

### 🎯 GOAL
30-minute pre-demo checklist complete, all 3 scenarios rehearsed from Railway, talk track ready, break/fix plan in hand.

### ✅ PREREQUISITES
- Chats 1–19 complete
- Demo happening soon
- Mobile phone charged

### 🖥️ CURSOR MODE
No Cursor coding — this is preparation and rehearsal. Run the demo exactly as you will on the day, from the same network, same device. Do this 2 hours before, not 2 minutes before.

### 📋 PROMPT (copy exactly)
```
No new code.

Ask me (Claude) in the new project:

"It's demo day. Help me with:

1. The 30-minute pre-demo verification checklist
   (what to test, what to check in each dashboard)

2. Talk track for each scenario — what do I say
   while the AI is responding? How do I frame
   what's happening to a non-technical audience?

3. Break/fix guide — top 5 things that could go
   wrong mid-demo and the fastest fix for each

4. How to switch from local to Railway and back
   in under 60 seconds if needed"
```

### ✔️ VERIFY
```bash
# Run all 3 scenarios from Railway (not local)
# Ask: 'What is the price of a 45kg LPG bottle?' → correct KB answer
# Call from known mobile → personalised greeting
# Order lookup returns correct row from orders.csv
```

### 🔜 NEXT CHAT HANDOFF
1. Demo day checklist complete — all scenarios verified on Railway
2. Talk track prepared for each scenario
3. Break/fix guide ready — you know exactly what to do if something fails

---

---

# APPENDIX — Quick Reference

## Cursor Mode Cheat Sheet

| Mode | Shortcut | When to use |
|------|----------|-------------|
| **Agent Mode** | Cmd+I | Creating new files, multi-file features, running terminal commands, building entire modules |
| **Inline Edit** | Cmd+K | Single-file edits, adding a method to an existing class, refactoring one function |
| **Tab Autocomplete** | Tab | Accepting a suggestion, then Tab again to chain the next logical edit |
| **New Chat** | When context ~70% full | Ask Cursor to summarise state, paste summary as first message in new chat |

---

## Key Terminal Commands

```bash
# Install deps / update after adding a package
uv sync

# Run agent locally (connects OUT to LiveKit Cloud — no ngrok needed)
uv run python agent.py dev

# Run ingestion (populate Qdrant KB)
ACTIVE_CUSTOMER=rockgas uv run python ingest.py

# Switch customer
ACTIVE_CUSTOMER=mbie uv run python agent.py dev

# Deploy to Railway
railway up

# Check Railway logs live
railway logs --tail

# Re-deploy with env var change
railway variables set ACTIVE_CUSTOMER=mbie && railway redeploy
```

---

## Environment Variables Reference

```bash
ACTIVE_CUSTOMER=rockgas          # which customer folder to load
LIVEKIT_URL=wss://xxx.livekit.cloud
LIVEKIT_API_KEY=APIxxxxxxxxxx
LIVEKIT_API_SECRET=xxxxxxxxxx
DEEPGRAM_API_KEY=xxxxxxxxxxxx
ELEVEN_API_KEY=sk_xxxxxxxxxxxx
OPENAI_API_KEY=sk-xxxxxxxxxxxx
QDRANT_URL=https://xxxx.aws.cloud.qdrant.io
QDRANT_API_KEY=xxxxxxxxxxxx
GMAIL_APP_PASSWORD=xxxx xxxx xxxx xxxx
```

---

## Top 5 Demo Failures + Fastest Fix

| # | Failure | Fastest Fix |
|---|---------|-------------|
| 1 | No audio on call | Check ElevenLabs API key. Fallback: `Agent(tts=openai.TTS())` |
| 2 | Call connects, no answer | Check agent is running. Check LiveKit dispatch rules. |
| 3 | Qdrant returns empty | Lower score threshold from 0.72 to 0.65 in search tool |
| 4 | Railway cold start (first call slow) | Call number yourself 5 min before demo to warm the process |
| 5 | Wrong customer greeting | Check ACTIVE_CUSTOMER in Railway dashboard, redeploy |

---

## Architecture at a Glance

```
Caller (PSTN)
    ↓
Twilio +64-xxx-xxxx
    ↓
Elastic SIP Trunk
    ↓
LiveKit Cloud (SIP → WebRTC)
    ↓
Python Agent (MacBook dev OR Railway)
    ↓ ↓ ↓ ↓
   VAD STT LLM TTS
(Silero) (Deepgram) (GPT-4o-mini) (ElevenLabs)
              ↓
         Tools
    ┌─────┼─────┐
    KB  Order  Email
  (Qdrant) (CSV) (Gmail)
    ↓
LiveKit Cloud
    ↓
Caller hears response
```

---

*AI Voice Agent — Presales Demo & Cursor Learning Lab*
*System prompt: 1_System_Instructions.MD in Claude Project knowledge*
