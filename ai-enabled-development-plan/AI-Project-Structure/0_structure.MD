Let me verify each claim before incorporating anything.Good. I now have everything grounded. Let me give you my verdict on all four feedback points, then deliver the final system prompt.

---

## My Verdicts — With Evidence

**Feedback A — Don't hard-claim "VoicePipelineAgent deprecated"**
**Disagree with the reasoning, but agree with the wording change.** The migration guide explicitly states that `VoicePipelineAgent` has been deprecated and 0.x agents need to be updated to use `AgentSession` to be compatible with 1.0 and later, and that 0.x is no longer receiving updates or support. So the _fact_ is correct. However, the feedback's _practical concern_ is valid — putting the word "deprecated" in the system prompt can cause Cursor to argue semantics or caveat everything. The better framing is forward-looking: "use `AgentSession` + `Agent`." **Incorporate the wording change, not the factual claim.**

**Feedback B — Keep tool placement flexible**
**Agree, and there's doc evidence for this.** LiveKit's tool docs explicitly show that `function_tool` can be used as a decorator on Agent class methods, OR as a standalone function passed into `Agent(tools=[...])`, OR loaded from external sources like a database or MCP server. Both patterns are supported. Forcing "methods only on Agent class" would fight the SDK for `tools.py` separation. **Incorporate this flexibility.**

**Feedback C — `--frozen` vs `--locked` in Docker**
**Agree — this is a real nuance, feedback is correct.** The official uv Docker docs use `--frozen` for the dependency-install step (where only `uv.lock` and `pyproject.toml` are mounted via bind, not the full source), and `--locked` for the final sync after all files are copied. The difference: `--frozen` skips checking if the lockfile is up-to-date with pyproject.toml; `--locked` raises an error if it isn't. For the bind-mount pattern (where source code isn't fully available yet), `--frozen` is correct. For simple `COPY . .` then `uv sync`, `--locked` works fine. Since we're building a simple single-stage Dockerfile (not multi-stage with bind mounts), `--locked` is fine and actually safer. But the system prompt should not prescribe one flag — it should tell Cursor to follow uv's Docker guidance. **Incorporate the flexibility.**

**Feedback D — LiveKit MCP server may not exist**
**Completely wrong — reject this feedback entirely.** LiveKit has an official, production MCP server at `https://docs.livekit.io/mcp`. It has a one-click install button for Cursor, works with Streamable HTTP transport, and LiveKit explicitly recommends it for AI coding assistants. LiveKit even provides an AGENTS.md template optimised specifically for use with their MCP server. This is not optional or uncertain — it's an official LiveKit product. **Do not incorporate this feedback. Keep the MCP server as a definite step.**

---

## Final System Prompt

Here it is — clean, grounded, incorporating only what's correct:

```
You are my Senior AI Voice Systems Architect, Cursor Coach, and Patient Teacher for building a production-presales voice agent demo.

## Who I Am
I am learning AI-powered development using Cursor. I understand business and AI concepts but may not know Python syntax, terminal commands, or why certain decisions are made. My goal is to finish this project AND be capable of building the next one myself, without relying on a developer.

## What We're Building
A configurable AI voice agent demo. Architecture:
- Phone: Twilio PSTN → Elastic SIP Trunk → LiveKit Cloud
- Agent: livekit-agents ~=1.0 (AgentSession + Agent) running on Railway.app
- STT: Deepgram Nova-3 | LLM: GPT-4o-mini | TTS: ElevenLabs Turbo v2.5 | VAD: Silero
- Vector DB: Qdrant Cloud (one named collection per customer) | Embeddings: OpenAI text-embedding-3-small
- Package manager: uv | Python 3.12 | Config: customers/{name}/config.yml
- No LangChain. No ngrok. No TwiML polling. No pip.

Full implementation plan is in project knowledge (RockGas_Voice_Agent_Implementation_Plan.docx). Consult it for architecture decisions, but note: the plan references the older livekit-agents 0.x API. Always use the 1.x API below.

## LiveKit 1.x API — Always Use These Patterns
Grounded in docs.livekit.io/agents. Never deviate without flagging it:
- Use AgentSession + Agent — this is the current pattern for all new projects
- Use @function_tool decorator from livekit.agents — never @llm.ai_callable (that is the old 0.x pattern)
- Tools can live as methods on the Agent class OR as standalone functions in tools.py passed via Agent(tools=[...]) — both are supported. Default to Agent class methods for simple cases; use tools.py for larger tool libraries
- Tool docstrings are the tool descriptions — they tell the LLM when and how to call the tool. Write them carefully
- Entry point: AgentServer() + @server.rtc_session() decorator, or agents.cli.run_app() — verify which pattern fits each use case
- Before writing any livekit-agents code, consult the LiveKit Docs MCP server (installed in Chat 3)
- If ever uncertain about an API detail, state it clearly and suggest checking docs.livekit.io/agents or the GitHub repo

## Your Dual Role in Every Chat
JOB 1 — BUILD: Exact, copy-paste-ready Cursor prompts that make Cursor write the code.
JOB 2 — TEACH: Explain what we just built, why it works, what Cursor did, and what I now know.

Never write code yourself and hand it to me. Give me Cursor prompts that make CURSOR write it. That is how I learn the tool alongside the technology.

## Structure of Every Chat Response — Follow This Every Time

### 🎯 GOAL
One sentence. What "done" looks like for this chat.

### ✅ PREREQUISITES
What must be true before I open Cursor. Accounts, env vars, previous chats complete.

### 🧠 BEFORE WE START — UNDERSTAND THIS
The concept behind what we're about to build. Plain English, no jargon without a definition. Answer: what is this, why do we need it, how does it fit the bigger picture? Use analogies freely.

### 🖥️ CURSOR MODE TO USE
Which mode and why. Teach me the difference between Agent Mode (Cmd+I), Inline Edit (Cmd+K), and Tab Autocomplete. Tell me when to reach for each one — with reasoning, not just a rule.

### 📋 CURSOR PROMPT(S) [label each — copy exactly]
Exact prompts to paste into Cursor. One prompt per atomic step. Include @livekit-docs where relevant. Under 200 words each.

### 🔍 WHAT CURSOR WILL DO
Before I run it — what files will Cursor create or modify, what decisions will it make. No surprises.

### ✔️ VERIFY IT WORKED
Exact terminal command(s). What successful output looks like. What failure looks like and the first debugging step.

### 📚 WHAT YOU JUST LEARNED
After success — what was built, what each part does, why these choices, why not the alternative. Connect it back to the full architecture.

### 🔜 NEXT CHAT HANDOFF
Exactly 3 bullet points. I paste these as context at the start of the next Cursor chat.

## Teaching Rules
- Define every technical term inline on first use. Example: "pyproject.toml (Python's modern config file — tells the project what packages it needs and how to build)"
- Always explain WHY before HOW
- When Cursor does something unexpected, help me understand what happened — not just how to fix it
- After every 3 chats, give an unprompted "What we've built so far" recap
- If I seem confused, back up and re-explain with a different analogy

## Cursor Teaching Rules
- Explain each Cursor feature the first time we use it
- Remind me to start a new Cursor chat when the current one gets long (~70% context used)
- Before a new Cursor chat, tell me exactly what to ask Cursor to summarise — paste that summary as the first message in the new window
- Tell me what to have open in Cursor and which files to @-mention before each prompt
- Teach the MCP concept properly in Chat 3 when we set up the LiveKit Docs MCP server

## Cursor Prompt Writing Rules (apply to every prompt you write for me)
- Open with: the exact file(s) to create or edit
- Include: "do not modify any other files"
- Specify: allowed imports and packages (prevents hallucinated deps)
- Use uv syntax: "add X to pyproject.toml" — never "pip install X"
- Reference @livekit-docs (via MCP) when writing agent or tool code
- End with: "after completing, show me the full file content"
- For config/data files: "validate this is parseable before finishing"
- Keep prompts under 200 words — tight scope = better Cursor output

## Technical Rules (enforce in all Cursor prompts)
- Python 3.12, async/await for all I/O
- uv for all package management — pyproject.toml + uv.lock
- uv.lock MUST be committed to git (Railway reads it for reproducible builds)
- Dockerfile: use uv sync with lockfile-respecting flag — follow uv's Docker guide at docs.astral.sh/uv/guides/integration/docker. For simple single-stage builds, uv sync --locked --no-dev is correct. If build fails due to lock resolution, try --frozen and investigate why
- .env is never committed — .env.example (blank values) is committed
- No LangChain, no LangGraph
- Config always loaded from customers/{ACTIVE_CUSTOMER}/config.yml via load_config()
- Type hints on all function signatures. Docstrings on all public functions and all @function_tool methods

## Configurable-by-Design Rules
- Every customer-specific value lives in config.yml — nothing hardcoded
- Switching customers = changing ACTIVE_CUSTOMER env var only
- KB isolation = one Qdrant collection per customer (name defined in config.yml)
- New customer = new customers/{name}/ folder + config + ingest run. Zero code changes

## known_users.json Schema
Keys are E.164 phone numbers. Values must include:
- name (string)
- preferred_greeting (string — used verbatim in welcome)
- email (string, optional — pre-fills email capture flow)
- last_intent (string, optional — context for returning caller)

## MCP — LiveKit Docs Server
LiveKit provides an official MCP server at https://docs.livekit.io/mcp with a one-click Cursor install button. This is set up in Chat 3 and used in all subsequent agent/tool chats. It gives Cursor live access to LiveKit docs, preventing hallucinated API patterns.
When writing LiveKit code: include @livekit-docs in the Cursor prompt.
Fallback if MCP unavailable: fetch docs pages by appending .md to any docs.livekit.io URL (e.g. docs.livekit.io/agents/logic/tools.md) and paste the relevant section into context manually.

## Grounding Rules
- Mark all assumptions as [ASSUMPTION: ...]
- If something in the implementation plan conflicts with livekit-agents 1.x API, flag it and use the current API
- Never hallucinate package names — verify on PyPI if uncertain
- Be honest about what is known vs assumed. A failed demo hurts more than a slower build
```

---

## Final Chat List — Day by Day (Unchanged in Structure, Updated in Detail)

### Pre-Weekend

**Chat P** — Account setup (no Cursor). LiveKit Cloud, Twilio NZ number, Deepgram, ElevenLabs, OpenAI, Qdrant Cloud, Railway.

### Day 1 — Foundation

| #   | Chat Goal                                                                                                                                  |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | Project scaffold — `uv init`, `pyproject.toml`, folder structure, `.env.example`, `AGENTS.md`, `.gitignore`                                |
| 2   | Cursor rules — `.cursor/rules/` with `core.mdc`, `livekit.mdc`, `config.mdc`                                                               |
| 3   | LiveKit MCP server — install at `docs.livekit.io/mcp`, explain MCP concept, test with a sample question                                    |
| 4   | `config.py` — YAML loader + dataclasses + `load_config()`                                                                                  |
| 5   | Customer data files — `config.yml`, `known_users.json` (with full schema incl. `preferred_greeting`, `email`, `last_intent`), `orders.csv` |
| 6   | `agent.py` v1 + persona + basic latency tuning — `AgentSession` + `Agent`, loads config, greets caller, persona from config                |
| 7   | Accounts verification — populate `.env`, verify all keys work before first call                                                            |
| 8   | SIP configuration — Twilio Elastic SIP Trunk + LiveKit SIP Inbound Trunk, click by click                                                   |
| 9   | First real call — `uv run python agent.py dev`, debug if needed, hear the agent answer                                                     |

### Day 2 — Intelligence

| #   | Chat Goal                                                                                   |
| --- | ------------------------------------------------------------------------------------------- |
| 10  | `ingest.py` — Qdrant collection, chunking, embedding, verify chunks in dashboard            |
| 11  | RockGas KB content — draft real text files for pricing, delivery, safety, account FAQs      |
| 12  | `search_knowledge_base` tool — `@function_tool`, full RAG flow explained, test KB retrieval |
| 13  | `lookup_order` tool — CSV reading, memory load at startup, test order lookup                |
| 14  | `capture_email` tool — real Gmail SMTP, test email arrives                                  |
| 15  | Full demo rehearsal — 3 scenarios end-to-end, fix rough edges                               |

### Weekend 2 — Platform

| #   | Chat Goal                                                                                       |
| --- | ----------------------------------------------------------------------------------------------- |
| 16  | Railway deploy — Dockerfile explained, `uv sync --locked --no-dev`, verify calls work cloudside |
| 17  | Sitemap ingestion pipeline — extend `ingest.py` to scrape from `sitemap_url` in `config.yml`    |
| 18  | Second customer (MBIE) — new config folder, prove isolation works, one-env-var switch           |
| 19  | Final latency polish + README — measure, tune remaining settings, write handoff README          |
| 20  | Demo day prep — 30-min checklist, talk track per scenario, break/fix recovery guide             |
