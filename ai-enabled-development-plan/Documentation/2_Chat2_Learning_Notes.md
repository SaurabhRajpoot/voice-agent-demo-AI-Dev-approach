# Chat 2 — Learning Notes
## AI Voice Agent Demo | Cursor Rules Setup

> **Project:** voice-agent-demo
> **Date:** February 2026
> **Phase:** Day 1 — Foundation
> **Status:** ✅ Complete

---

## Table of Contents
1. [What We Built](#what-we-built)
2. [The Big Picture — Why Rules Matter](#the-big-picture)
3. [Understanding Cursor Deeply](#understanding-cursor-deeply)
4. [Every Rule File Explained](#every-rule-file-explained)
5. [Key Concepts Learned](#key-concepts-learned)
6. [The Cursor Prompt — Exact Copy](#the-cursor-prompt)
7. [What Cursor Did — Step by Step](#what-cursor-did-step-by-step)
8. [How To Do It Manually (Without Cursor)](#how-to-do-it-manually)
9. [Verify It Worked](#verify-it-worked)
10. [Schema Decisions Made & Why](#schema-decisions-made-and-why)
11. [Handoff Into Chat 3](#handoff-into-chat-3)

---

## What We Built

Three `.mdc` rule files in a `.cursor/rules/` directory. These are **persistent instructions** that Cursor loads automatically — no need to repeat your project conventions in every prompt ever again.

**End state after Chat 2:**
```
voice-agent-demo/
├── .cursor/
│   └── rules/
│       ├── core.mdc        ← always loaded in every session
│       ├── livekit.mdc     ← loads when agent.py or tools.py is open
│       └── config.mdc      ← loads when customers/** files are open
├── customers/rockgas/docs/
├── tests/
├── .env.example
├── AGENTS.md
├── pyproject.toml
└── uv.lock
```

---

## The Big Picture — Why Rules Matter

### The Problem Without Rules

Every time you open a new Cursor Agent session, Cursor starts fresh. It has no memory of your project. Without rules, you'd need to repeat in every single prompt:

- "use uv not pip"
- "use AgentSession + Agent, not VoicePipelineAgent"
- "never hardcode customer values"
- "known_users.json only has name and email"

Across 20 chats and hundreds of prompts, that's exhausting — and easy to forget, leading to Cursor writing code that breaks your conventions.

### The Solution — Standing Instructions

`.cursor/rules/*.mdc` files are **always-on instructions** for Cursor. Write them once, and they load automatically based on rules you define. Think of it as the difference between:

- **Without rules:** Briefing a new contractor every single day from scratch
- **With rules:** Having a policy document on the desk that every contractor reads before starting work

### Why This Matters More As The Project Grows

Right now with 1 file, repeating conventions in each prompt is manageable. By Chat 12, you'll have `agent.py`, `tools.py`, `config.py`, `ingest.py`, and multiple customer folders open simultaneously. Rules files ensure Cursor stays consistent across all of them automatically.

---

## Understanding Cursor Deeply

This is your complete Cursor upskilling reference. Everything you need to use Cursor like a professional.

---

### The Three Cursor Modes — When to Use Each

| Mode | Shortcut | What It Does | When to Use |
|---|---|---|---|
| **Agent Mode** | `Cmd+I` | Full agentic control — reads project, runs terminal commands, creates/edits multiple files, verifies results | New files, multi-step tasks, anything spanning more than one file |
| **Inline Edit** | `Cmd+K` | Focused edit on selected code or a specific file section | Adding a method to an existing class, fixing a bug, refining one function |
| **Tab Autocomplete** | `Tab` | Accepts Cursor's inline suggestion as you type | Accepting completions, chaining suggestions |

**The mental model:**
- Agent Mode = "Cursor, here is a task, go do it"
- Inline Edit = "Cursor, here is this specific thing, change it this way"
- Tab = "Yes, that suggestion is right, accept it"

**A practical example from this build:**

| Chat | Task | Why This Mode |
|---|---|---|
| Chat 1 | Create 6 files across multiple directories | Agent Mode — multi-file, needs terminal |
| Chat 2 | Create 3 new rule files | Agent Mode — new directory + multiple files |
| Chat 4 | Create config.py with multiple classes | Agent Mode — new complex file |
| Chat 6 | Create agent.py | Agent Mode — new complex file |
| Chat 12 | Add search tool to existing agent.py | Inline Edit — one method in existing file |
| Chat 13 | Add order lookup to existing agent.py | Inline Edit — one method in existing file |
| Any | Accept a variable name suggestion | Tab |

---

### Cursor's Context Window — What It Can See

Cursor's AI has a context window — a limit to how much it can "see" at once. Understanding this makes you a better Cursor user.

**What Cursor reads automatically:**
- The file you have open
- Files you @-mention in your prompt
- `.cursor/rules/` files (based on trigger conditions)
- `AGENTS.md` (if it exists — another reason we created it)

**What Cursor does NOT read automatically:**
- Files you haven't opened or mentioned
- Your previous Cursor chat sessions (no memory between sessions)
- Docs from external services (unless you use MCP — see Chat 3)

**Practical implication:** If you want Cursor to know about a file, either open it or @-mention it in your prompt. Example: `@config.py make sure the new tool follows the same pattern`.

---

### The @ System — How to Reference Things in Cursor Prompts

The `@` symbol in Cursor prompts is how you pull specific things into context:

| What to type | What it does |
|---|---|
| `@filename.py` | Includes that file's content in context |
| `@livekit-docs` | Pulls from LiveKit MCP server (set up in Chat 3) |
| `@docs` | References your project documentation |
| `@web` | Searches the web for current information |

**Example prompt using @:**
```
Using @livekit-docs, add a @function_tool to agent.py
that searches the knowledge base. Follow the pattern
already established in @config.py.
```

This is much more effective than describing things from memory — you're giving Cursor the actual source material.

---

### Context Window Management — When to Start a New Chat

Cursor's context fills up during a session. When it gets full, responses get worse — Cursor starts forgetting earlier instructions or making inconsistent decisions.

**Watch for these signals:**
- Cursor starts ignoring your rules
- It reverts to old patterns (like using `pip` instead of `uv`)
- Responses feel less coherent
- Cursor shows a context warning indicator

**What to do when context is ~70% full:**

Step 1 — Ask Cursor to summarise before starting a new chat:
```
Summarise the current state of this project in 5 bullet points:
what files exist, what works, what's next.
```

Step 2 — Copy that summary

Step 3 — Open a new Cursor Agent chat (`Cmd+I` → New Chat)

Step 4 — Paste the summary as your first message

The rules files we created in Chat 2 mean you never lose the core conventions even when you start a new chat — they reload automatically.

---

### Model Selection — Which to Use When

| Model | Use For | Notes |
|---|---|---|
| `claude-sonnet-4.6` (default) | All Agent Mode tasks in this build | Best balance of intelligence + speed for multi-file agentic work |
| `claude-opus-4.6-high` | Genuinely hard architectural problems | Slower, uses more credits — overkill for most tasks |
| `gpt-5.3-codex` | Avoid for this project | Tends to hallucinate livekit-agents API patterns |
| `composer-1.5` | Cursor's own model | Fine but Claude Sonnet is more reliable for complex instructions |
| `MAX Mode` | Leave off | Burns through credits rapidly |

**Rule of thumb:** Stay on Claude Sonnet for everything in this build. Only switch if a specific task genuinely isn't working.

---

### How `.cursor/rules/*.mdc` Files Work

`.mdc` is Cursor's own file format — Markdown with YAML frontmatter that controls when the rule loads.

**The frontmatter block:**
```
---
alwaysApply: true          ← loads in EVERY session, no conditions
---
```
or
```
---
description: What this rule covers
globs: agent.py, tools.py   ← loads ONLY when these files are open
alwaysApply: false
---
```

**Three trigger types:**

| Trigger | Syntax | When it loads |
|---|---|---|
| Always | `alwaysApply: true` | Every Cursor session, every file |
| File glob | `globs: agent.py` | Only when that specific file is open |
| Pattern glob | `globs: customers/**/*.yml` | Any file matching that pattern |

**Glob patterns explained:**
- `agent.py` — matches exactly that filename
- `customers/**/*.yml` — matches any `.yml` file anywhere inside `customers/`
- `src/**/*.py` — matches any `.py` file anywhere inside `src/`
- `**` means "any number of subdirectories"

---

### User Rules vs Project Rules

Cursor has two levels of rules:

**Project Rules** (`.cursor/rules/*.mdc`) — what we built in Chat 2
- Stored in your project folder
- Committed to git
- Apply to this project only
- Shared with anyone who clones the repo

**User Rules** (Cursor Settings → Rules for AI)
- Stored in your Cursor application settings
- Apply across ALL your projects
- Personal preferences — not project-specific

**What to put in User Rules (set these once for yourself):**
```
Always explain what you're about to do before doing it.
When you make a decision, briefly explain why.
If you're uncertain about an API, say so rather than guessing.
Prefer simple solutions over clever ones.
```

These are your personal working style preferences. They complement project rules — project rules enforce technical conventions, user rules shape how Cursor communicates with you.

**How to set User Rules:**
Cursor → Settings (Cmd+,) → Rules for AI → paste your preferences

---

## Every Rule File Explained

### FILE 1: `.cursor/rules/core.mdc`

```markdown
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
```

**What each line enforces:**

| Rule | Why it matters |
|---|---|
| `Use uv for all packages. Never pip install.` | Prevents Railway deployment failures from inconsistent package versions |
| `Python 3.12, async/await everywhere.` | livekit-agents requires 3.12+; async is mandatory for streaming audio pipeline |
| `Type hints + docstrings on all functions.` | Makes code readable and maintainable; docstrings on tools become LLM descriptions |
| `No langchain, no langgraph.` | Prevents 200ms+ latency overhead on every LLM call — unacceptable for voice |
| `Config via: from config import load_config` | Ensures all code uses the central config loader, never hardcoded paths |
| `Never hardcode customer values — use config.yml.` | The configurable-by-design principle — customer swap = env var change only |
| `Do not modify files outside current task scope.` | Prevents Cursor from making "helpful" changes to files you didn't ask it to touch |
| `Show full file content after any edit.` | So you can verify exactly what changed — no surprises |

---

### FILE 2: `.cursor/rules/livekit.mdc`

```markdown
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
```

**Why this rule exists:**
The livekit-agents library had a major API change between version 0.x and 1.x. The implementation plan document was written for 0.x. Without this rule, Cursor might revert to the old patterns from its training data:

| Old 0.x (WRONG) | New 1.x (CORRECT) |
|---|---|
| `VoicePipelineAgent` | `AgentSession + Agent` |
| `@llm.ai_callable` | `@function_tool` |
| `FunctionContext` class | Methods on `Agent` class |

This rule loads automatically whenever `agent.py` or `tools.py` is open — exactly the files where these patterns matter. You never have to remember to say "use the 1.x API" in your prompts.

---

### FILE 3: `.cursor/rules/config.mdc`

```markdown
---
description: Config and data file conventions
globs: customers/**/*.yml, customers/**/*.json
alwaysApply: false
---
known_users.json keys = E.164 phone numbers (+64xxxxxxxxx).
Values must have: name (string), email (string).
No other fields required — this is the lightweight CRM.
Known user lookup: if phone number exists in file = known user.
Greeting templates live in config.yml, not in known_users.json.
orders.csv columns: order_id, account, name, product, status, eta.
```

**Why this rule exists:**
When Cursor creates or edits customer data files, it needs to know the exact schema. Without this rule, it might invent extra fields, use the wrong format for phone numbers, or put greeting templates in the wrong place.

**The E.164 format explained:**
E.164 is the international standard for phone numbers. Format: `+[country code][number]`
- NZ example: `+6421000001` (not `021000001` or `0064-21-000001`)
- This matters because Twilio passes caller numbers in E.164 format via SIP headers
- The lookup in `agent.py` does: `known_users.json[caller_phone]` — the key must match exactly

---

## Key Concepts Learned

### The Architectural Decision — `known_users.json` Schema

During Chat 2 we refined the `known_users.json` schema. Here's the full reasoning:

**Original approach (rejected):**
Each user had a `preferred_greeting` field — a custom greeting string stored per person.

**Problem with that approach:**
- If RockGas changes their greeting style, you edit every single user entry
- Mixing customer policy (greeting style) with user data (who they are)
- More data to maintain for no real benefit

**Your better approach (adopted):**
Greeting templates live in `config.yml`. User data stays minimal.

```yaml
# customers/rockgas/config.yml — CUSTOMER POLICY
agent:
  welcome_message: "Welcome to RockGas. How can I help you today?"
  welcome_known_template: "Welcome back {name}! Great to hear from you."
```

```json
// customers/rockgas/known_users.json — USER DATA
{
  "+6421000001": {
    "name": "James Wilson",
    "email": "james@example.com"
  }
}
```

```python
# agent.py — HOW THEY COMBINE
user = cfg.lookup_user(caller_phone)
if user:
    greeting = cfg.agent.welcome_known_template.format(name=user["name"])
else:
    greeting = cfg.agent.welcome_message
```

**The principle this demonstrates:**
Separate *policy* (how we greet) from *data* (who they are). Policy belongs in config. Data belongs in data files.

---

### Why `.mdc` and Not `.md` or `.txt`

`.mdc` is Cursor's own format — it's standard Markdown but with the YAML frontmatter block at the top (`---` delimiters) that Cursor reads to know when and how to apply the rule. A plain `.md` file would just be a document. The `.mdc` extension signals to Cursor "this is a rule file, read the frontmatter."

---

### The `globs` Pattern — How File Matching Works

When you open a file in Cursor, it checks all your `.mdc` files and loads any whose `globs` pattern matches the filename.

```
You open: agent.py
Cursor checks:
  core.mdc        → alwaysApply: true → LOAD ✅
  livekit.mdc     → globs: agent.py  → MATCH → LOAD ✅
  config.mdc      → globs: customers/**/*.yml → no match → skip
```

```
You open: customers/rockgas/config.yml
Cursor checks:
  core.mdc        → alwaysApply: true → LOAD ✅
  livekit.mdc     → globs: agent.py  → no match → skip
  config.mdc      → globs: customers/**/*.yml → MATCH → LOAD ✅
```

This means you never have to think about which rules apply — Cursor handles it based on what you're working on.

---

## The Cursor Prompt — Exact Copy

> This is what was pasted into Cursor Agent Mode (`Cmd+I`). Copy exactly as written for reference.

**Handoff context (pasted first):**
```
1. Project scaffolded with uv, Python 3.12 — pyproject.toml,
   .env.example, AGENTS.md, .gitignore, and uv.lock all created.
   uv run python --version returns 3.12.12. .venv exists with
   90 packages installed including livekit-agents==1.4.3.

2. Folder structure in place: voice-agent-demo/ with
   customers/rockgas/docs/ and tests/ created correctly.

3. uv sync completed — all livekit-agents 1.x plugins,
   qdrant-client, openai, pyyaml, python-dotenv installed
   and locked in uv.lock. Ready for Chat 2: Cursor rules
   setup (.cursor/rules/*.mdc files that enforce project
   conventions automatically in every future session).
```

**Main prompt:**
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
known_users.json keys = E.164 phone numbers (+64xxxxxxxxx).
Values must have: name (string), email (string).
No other fields required — this is the lightweight CRM.
Known user lookup: if phone number exists in file = known user.
Greeting templates live in config.yml, not in known_users.json.
orders.csv columns: order_id, account, name, product, status, eta.

After creating all three, show me each file.
```

---

## What Cursor Did — Step by Step

### STEP 1 — Created `.cursor/rules/` directory
```bash
mkdir -p .cursor/rules
```
The `.cursor/` folder is Cursor's configuration directory for this project — similar to how `.git/` is git's configuration directory. The `rules/` subfolder specifically holds rule files.

### STEP 2 — Created `core.mdc`
Cursor created the file with the `alwaysApply: true` frontmatter. This single flag is what makes the rule load in every session without any conditions.

### STEP 3 — Created `livekit.mdc`
Cursor created the file with `alwaysApply: false` and the `globs` pattern. The glob `agent.py, tools.py, src/**/*.py` means this rule only activates when those files are open — preventing it from cluttering context when working on unrelated files.

### STEP 4 — Created `config.mdc`
Cursor created the file with the customer data schema. The glob `customers/**/*.yml, customers/**/*.json` covers all YAML and JSON files anywhere inside the `customers/` directory — for both current and future customers (rockgas, mbie, any new ones).

### STEP 5 — Showed all three files
Cursor displayed the full content of each file for verification. This is the "Show full file content after any edit" rule from `core.mdc` already working — Cursor followed its own new rule.

---

## How To Do It Manually (Without Cursor)

If you ever need to create Cursor rules from scratch manually:

### Step 1: Create the directory
```bash
mkdir -p .cursor/rules
```

### Step 2: Create `core.mdc`
```bash
cat > .cursor/rules/core.mdc << 'EOF'
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
EOF
```

### Step 3: Create `livekit.mdc`
```bash
cat > .cursor/rules/livekit.mdc << 'EOF'
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
EOF
```

### Step 4: Create `config.mdc`
```bash
cat > .cursor/rules/config.mdc << 'EOF'
---
description: Config and data file conventions
globs: customers/**/*.yml, customers/**/*.json
alwaysApply: false
---
known_users.json keys = E.164 phone numbers (+64xxxxxxxxx).
Values must have: name (string), email (string).
No other fields required — this is the lightweight CRM.
Known user lookup: if phone number exists in file = known user.
Greeting templates live in config.yml, not in known_users.json.
orders.csv columns: order_id, account, name, product, status, eta.
EOF
```

### Step 5: Verify
```bash
ls .cursor/rules/
# Expected: config.mdc  core.mdc  livekit.mdc

cat .cursor/rules/core.mdc
# Should show the frontmatter and rules
```

### Step 6: Commit
```bash
git add .cursor/
git commit -m "Chat 2: cursor rules — core, livekit, config"
git push
```

---

## Verify It Worked

### Command:
```bash
ls .cursor/rules/
```

### Your actual result (confirmed ✅):
```
config.mdc      core.mdc        livekit.mdc
```

### How to verify rules are loading in Cursor:
1. Open `agent.py` (doesn't exist yet — open any `.py` file)
2. In a new Agent chat, type: "what rules are you following for this project?"
3. Cursor should mention the livekit-agents 1.x patterns and uv conventions
4. This confirms the rules are being read

---

## Schema Decisions Made and Why

### `known_users.json` — Final Agreed Schema

```json
{
  "+6421000001": {
    "name": "James Wilson",
    "email": "james@example.com"
  },
  "+6421000002": {
    "name": "Sarah Thompson",
    "email": "sarah@example.com"
  }
}
```

**Why the phone number is the key (not a field):**
Twilio passes the caller's number via SIP headers. The lookup is: `known_users[caller_phone]`. Using the phone number as the dictionary key makes this a single O(1) lookup — instant, no searching required.

**Why only name and email:**
This file is a lightweight CRM — it answers one question: "given a phone number, who is this person?" Name personalises the greeting. Email pre-fills the capture flow. Nothing else is needed for the demo. YAGNI — You Ain't Gonna Need It.

**What each field is used for at runtime:**

| Field | Used by | Purpose |
|---|---|---|
| key (phone) | `agent.py` → `lookup_user()` | Identify if caller is known |
| `name` | `config.yml` template → greeting | "Welcome back {name}!" |
| `email` | `tools.py` → `capture_email` | Pre-fill email when sending summary |

### Responsibility Split — The Clean Architecture

| File | Owns | Does NOT own |
|---|---|---|
| `known_users.json` | Who is this person? | How to greet them |
| `config.yml` | How do we greet people? | Who specific people are |
| `orders.csv` | What have they ordered? | Who they are or how to greet them |
| `agent.py` | How do we combine all of the above? | Any hardcoded values |

---

## Cursor Upskilling Summary — What You Now Know

After Chats 1 and 2, here is your complete Cursor knowledge base:

### The Three Modes
- **Agent Mode** (`Cmd+I`) — multi-file tasks, terminal commands, new files
- **Inline Edit** (`Cmd+K`) — targeted edits to existing files
- **Tab** — accept autocomplete suggestions

### Rules System
- `alwaysApply: true` — loads every session
- `globs: pattern` — loads when matching files are open
- Project rules (`.cursor/rules/`) — per project, committed to git
- User rules (Settings) — personal style, all projects

### Context Management
- Context fills up — start new chat at ~70% full
- Ask Cursor to summarise state before switching chats
- Rules reload automatically in new chats — conventions never lost

### The @ System
- `@filename` — include file in context
- `@livekit-docs` — pull from MCP server (Chat 3)
- Use @ to give Cursor source material, not just descriptions

### Model Choice
- Claude Sonnet — default for everything in this build
- Never Auto — inconsistent results
- Never GPT models for livekit-agents code

---

## Architecture Reminder — Where Chat 2 Fits

```
Chat P  ✅  All 7 accounts created, API keys saved
Chat 1  ✅  Project structure, dependencies, uv.lock
Chat 2  ✅  Cursor rules — enforce conventions automatically forever
Chat 3  →   LiveKit MCP server — live API docs inside Cursor
Chat 4  →   config.py — loads customer YAML into typed Python objects
Chat 5  →   Customer data files — RockGas config, known users, orders
Chat 6  →   agent.py — the voice agent itself, first working version
Chat 7  →   API key verification
Chat 8  →   SIP configuration (Twilio + LiveKit)
Chat 9  →   First real phone call
```

---

## Handoff Into Chat 3

Paste these 3 bullets at the start of your Chat 3 session:

```
1. Project scaffolded and rules configured — .cursor/rules/ contains
   three .mdc files: core.mdc (alwaysApply), livekit.mdc (globs:
   agent.py, tools.py), config.mdc (globs: customers/**). All
   conventions are now enforced automatically in every Cursor session.

2. known_users.json schema locked: keys = E.164 phone numbers,
   values = {name, email} only. Greeting templates live in
   config.yml not in user data. This is the lightweight CRM pattern.

3. livekit-agents==1.4.3 installed. Rules enforce 1.x API patterns:
   AgentSession + Agent, @function_tool — never old 0.x patterns.
   Ready for Chat 3: LiveKit MCP server setup (browser config,
   no coding — gives Cursor live access to LiveKit API docs).
```

---

## Golden Rules — Updated After Chat 2

> **Never `pip install`** — always `uv add` or `uv sync`

> **Never commit `.env`** — commit `.env.example` only

> **Always commit `uv.lock`** — Railway needs it for reproducible builds

> **Never hardcode customer values** — everything customer-specific goes in `customers/{name}/config.yml`

> **Use livekit-agents 1.x patterns** — `AgentSession + Agent` and `@function_tool`

> **Greeting templates in `config.yml`** — user data (`name`, `email`) in `known_users.json`

> **Start a new Cursor chat at ~70% context** — ask for state summary first, paste it in new chat

> **Use `@filename` to give Cursor source material** — don't just describe things, reference them directly

---

*voice-agent-demo | Chat 2 Learning Notes | February 2026*
