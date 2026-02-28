# Chat 8 — Learning Notes
## AI Voice Agent Demo | SIP Trunk Configuration — LiveKit + Twilio

> **Project:** voice-agent-demo
> **Date:** 28 February 2026
> **Phase:** Day 2 — SIP Pipeline Configuration
> **Status:** ✅ Complete — Sam answered real phone calls, recognised Saurabh by name

---

## Table of Contents
1. [What We Built](#what-we-built)
2. [The Full Call Journey](#the-full-call-journey)
3. [Key Concepts — SIP, PSTN, Trunks](#key-concepts)
4. [Part 1 — LiveKit Configuration](#part-1-livekit-configuration)
5. [Part 2 — Twilio Configuration](#part-2-twilio-configuration)
6. [Code Change — agent_name Made Configurable](#code-change)
7. [The Two Successful Calls — Log Analysis](#the-two-successful-calls)
8. [Naming Decisions Made](#naming-decisions-made)
9. [What's Shared vs Per-Customer](#shared-vs-per-customer)
10. [Issues We Hit and How We Fixed Them](#issues-we-hit)
11. [Complete Configuration Reference](#complete-configuration-reference)
12. [Handoff Into Chat 9](#handoff-into-chat-9)

---

## What We Built

A complete SIP pipeline connecting Twilio's phone network to LiveKit Cloud, enabling real inbound phone calls to be answered by Sam the AI voice agent.

**Before Chat 8:**
```
Someone dials +6498869807 → Twilio → ??? → nowhere
```

**After Chat 8:**
```
Someone dials +6498869807
        ↓
Twilio (voiceagent-demo-pstn-trunk)
        ↓
LiveKit SIP URI (sip:5tc1esg6moj.sip.livekit.cloud)
        ↓
LiveKit Dispatch Rule (rockgas-dispatch → rockgas-agent)
        ↓
agent.py → Sam speaks
```

**What was proven:**
- Real phone calls answered ✅
- Caller identified by phone number ✅
- Saurabh greeted by name on second call ✅
- NZ-accented speech transcribed accurately ✅
- Sam answered questions about RockGas services ✅
- Clean session close on hangup ✅

---

## The Full Call Journey

Understanding this end-to-end flow is essential for debugging any future issues.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        INBOUND CALL FLOW                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. Caller dials +6498869807 from their mobile                      │
│           ↓                                                         │
│  2. NZ phone network (PSTN) routes to Twilio                        │
│           ↓                                                         │
│  3. Twilio checks: "what do I do with this number?"                 │
│     → Looks at phone number config                                  │
│     → Configure with: SIP Trunk                                     │
│     → SIP Trunk: voiceagent-demo-pstn-trunk                         │
│           ↓                                                         │
│  4. Twilio SIP Trunk forwards the call                              │
│     → Origination URI: sip:5tc1esg6moj.sip.livekit.cloud           │
│           ↓                                                         │
│  5. LiveKit SIP service receives the SIP INVITE                     │
│     → Checks: which trunk does this match?                          │
│     → Matches: voiceagent-demo-inbound-pstn (+6498869807)           │
│           ↓                                                         │
│  6. LiveKit checks dispatch rules for this trunk                    │
│     → Matches: rockgas-dispatch                                     │
│     → Creates room: call-_+642108430555_[random]                    │
│     → Dispatches job to: rockgas-agent                              │
│           ↓                                                         │
│  7. agent.py receives the job                                       │
│     → Connects to the room                                          │
│     → Extracts phone from sip.phoneNumber attribute                 │
│     → Calls lookup_user() → finds/doesn't find caller               │
│     → Builds greeting (personalised or generic)                     │
│     → Starts AgentSession (STT + LLM + TTS + VAD)                  │
│     → Sam speaks the greeting                                       │
│           ↓                                                         │
│  8. Conversation loop until caller hangs up                         │
│     → Caller speaks → Deepgram → GPT-4o-mini → ElevenLabs          │
│           ↓                                                         │
│  9. Caller hangs up → CLIENT_INITIATED disconnect                   │
│     → Session closes cleanly                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts

### PSTN — Public Switched Telephone Network
The regular phone network. When someone dials a number from their mobile or landline, they're using PSTN. Twilio bridges PSTN to the internet.

### SIP — Session Initiation Protocol
The standard language phone systems use to set up, manage, and end calls. Think of it as the postal system for phone calls — it handles the "envelope" (who's calling, where to send it), not the actual voice content. SIP messages are text-based and look similar to HTTP requests.

### SIP Trunk
A virtual phone line that bridges PSTN and internet-based systems. Twilio's Elastic SIP Trunking converts incoming PSTN calls into SIP messages and sends them over the internet. It's called a "trunk" because it carries many calls simultaneously, like a trunk line.

### SIP URI
A SIP address — like an email address but for phone calls:
```
sip:5tc1esg6moj.sip.livekit.cloud
```
This is LiveKit's SIP endpoint — the "address" where LiveKit receives incoming SIP calls.

### Origination URI (Twilio term)
Where Twilio sends calls when they arrive on a trunk. "Origination" from Twilio's perspective means "where the call originates from the PSTN into our network". Confusingly named — it means "destination" from our perspective.

### Dispatch Rule (LiveKit term)
A rule that tells LiveKit what to do when a call arrives on a SIP trunk:
- Which room to create
- Which agent to dispatch
- Which trunk(s) to apply the rule to

### Individual Room Type
Each caller gets their own private room. This is what we want — one room per phone call. The alternative is "Direct" (all callers share one room) which is wrong for phone calls.

---

## Part 1 — LiveKit Configuration

### Where to Go
`cloud.livekit.io` → select `voice-agent-demo` project → **Telephony** in left sidebar

---

### Step 1 — Create Inbound SIP Trunk

**Navigate to:** Telephony → SIP trunks → **Create new trunk** (top right button)

**Form filled in:**

| Field | Value | Notes |
|---|---|---|
| Trunk name | `voiceagent-demo-inbound-pstn` | Generic name — reused for all customer demos |
| Trunk direction | `Inbound` | Receiving calls, not making them |
| Numbers | `+6498869807` | Our NZ Twilio number in E.164 format |
| Allowed addresses | empty | Accept from any IP — Twilio IPs change |
| Media encryption (SRTP) | Disabled | Not needed for demo |
| Include headers | No headers | Default |
| Enable Krisp | Unchecked | We use BVCTelephony in agent.py instead |

**Why no credentials?**
The new LiveKit UI (Feb 2026) removed username/password fields from the trunk creation form. LiveKit now uses IP-based trust or open access (when Allowed addresses is empty). Authentication is handled at the dispatch rule level.

**Why "voiceagent-demo-inbound-pstn" not "rockgas-inbound"?**
This trunk is platform infrastructure — shared across ALL customer demos. RockGas, MBIE, Contact Energy all use the same trunk. Only the dispatch rule is customer-specific.

**Result after creation:**
```
Trunk ID:   ST_ZrPoUjLXcwSe
Trunk name: voiceagent-demo-inbound-pstn
Numbers:    +6498869807
Created:    28 Feb 2026, 18:00:30
```

**Your LiveKit SIP URI** (visible at top of SIP trunks page):
```
sip:5tc1esg6moj.sip.livekit.cloud
```
Save this — needed for Twilio configuration.

---

### Step 2 — Create Dispatch Rule

**Navigate to:** Telephony → Dispatch rules → **Create new dispatch rule** (top right button)

**Form filled in:**

| Field | Value | Notes |
|---|---|---|
| Rule name | `rockgas-dispatch` | Customer-specific — identifies which customer |
| Rule type | `Individual` | Each call gets its own private room |
| Room prefix | `call-` | Rooms named call-_+64XXX_[random] |
| Agent name | `rockgas-agent` | Must match exactly what's in agent.py |
| Dispatch metadata | empty | Not needed |
| Inbound routing | **Trunks tab** → `voiceagent-demo-inbound-pstn` | Link to our trunk |

**Why "rockgas-dispatch" not "voiceagent-demo-dispatch"?**
One dispatch rule per customer — the name should clearly identify which customer it belongs to. When you add MBIE later, create `mbie-dispatch`. No limit on number of dispatch rules on any plan.

**Why Individual not Direct?**
Individual = each caller gets their own private room. Direct = all callers share one room. Phone calls must be Individual — you don't want two callers hearing each other.

**Why does Agent name matter so much?**
`rockgas-agent` in the dispatch rule must match exactly the `agent_name` parameter in `agent.py`:
```python
@server.rtc_session(agent_name="rockgas-agent")
```
If these don't match, LiveKit dispatches the job but no agent picks it up — call goes silent.

**Result after creation:**
```
Dispatch Rule ID: SDR_wPp92Dk3AAiK
Rule name:        rockgas-dispatch
Inbound routing:  ST_ZrPoUjLXcwSe (voiceagent-demo-inbound-pstn)
Destination room: call-<caller-number>
Agents:           rockgas-agent
Rule type:        Individual
Created:          28 Feb 2026, 18:15:21
```

---

## Part 2 — Twilio Configuration

### Where to Go
`console.twilio.com` → your account

---

### Step 3 — Create Elastic SIP Trunk

**Navigate to:** Left sidebar → **Elastic SIP Trunking** → **Manage** → **Trunks** → **Create new SIP Trunk**

**Form filled in:**
```
Friendly Name: voiceagent-demo-pstn-trunk
```

**Result:**
```
Trunk SID:     TK2386c19bbac210029fc90c4b300a2da6
Friendly name: voiceagent-demo-pstn-trunk
```

**Why generic name?**
This trunk is reused for all demos. Never needs to change. One Twilio trunk → one LiveKit SIP URI → many dispatch rules.

---

### Step 4 — Add Origination URI

**Navigate to:** Inside your trunk → **Origination** tab → **Add new Origination URI**

**Form filled in:**

| Field | Value | Notes |
|---|---|---|
| Origination SIP URI | `sip:5tc1esg6moj.sip.livekit.cloud` | Your LiveKit SIP URI |
| Priority | `10` | Used for failover — 10 is standard |
| Weight | `10` | Load balancing weight — 10 is standard |
| Enabled | `enabled` | Active ✅ |

**What is Origination URI?**
This tells Twilio "when a call comes in on this trunk, forward it to this SIP address". Despite the confusing name, it's the destination — LiveKit's SIP endpoint.

**No username prefix?**
In the old LiveKit UI, credentials were required and the URI format was:
```
sip:username@livekit-sip-uri.cloud
```
The new LiveKit UI (Feb 2026) doesn't require credentials on the trunk, so we use the URI directly:
```
sip:5tc1esg6moj.sip.livekit.cloud
```

**Priority and Weight explained:**
- Priority: lower number = higher priority. If you had two URIs (failover), priority 1 would be tried before priority 10.
- Weight: used when multiple URIs have the same priority — higher weight = more traffic. With one URI, both values are just set to 10 as standard.

---

### Step 5 — Attach Phone Number to Trunk

**Navigate to:** Phone number configuration page for `+6498869807`

Found via: Left sidebar → Phone Numbers → Active numbers → click `+6498869807` → Configure tab

**What was already there:**
An old ngrok webhook URL from earlier testing:
```
https://b76e5261fe9a.ngrok-free.app
```
This needed to be replaced.

**Changed:**

| Field | Old value | New value |
|---|---|---|
| Configure with | `Webhook, TwiML Bin, Function...` | `SIP Trunk` |
| SIP Trunk | (empty) | `voiceagent-demo-pstn-trunk` |

**Note:** When you select "SIP Trunk" in the dropdown, Twilio automatically shows your available trunks. The `voiceagent-demo-pstn-trunk` we just created appeared immediately.

**Clicked:** Save configuration ✅

**Why this step?**
A Twilio SIP trunk doesn't automatically handle all your phone numbers. You must explicitly tell each phone number "route calls through this trunk". One number can only be attached to one trunk at a time.

---

## Code Change — agent_name Made Configurable

During Chat 8, we identified that hardcoding `"rockgas-agent"` in `agent.py` violated the configurable-by-design principle. Fixed before completing the dispatch rule.

### Three Files Updated

**customers/rockgas/config.yml** — added `agent_name` field:
```yaml
agent:
  name: Sam
  agent_name: rockgas-agent        ← NEW
  welcome_message: "Thank you for calling RockGas. I'm Sam,
    your virtual assistant. How can I help you today?"
  welcome_known_template: "Welcome back {name}! Great to hear
    from you. How can I help you today?"
```

**config.py** — added `agent_name` to `AgentConfig` dataclass:
```python
@dataclass
class AgentConfig:
    name: str
    agent_name: str                 ← NEW
    welcome_message: str
    welcome_known_template: str
```

And wired into `load_config()`:
```python
agent=AgentConfig(
    name=raw["agent"]["name"],
    agent_name=raw["agent"]["agent_name"],    ← NEW
    welcome_message=raw["agent"]["welcome_message"],
    welcome_known_template=raw["agent"]["welcome_known_template"],
)
```

**agent.py** — replaced hardcoded string with config value:
```python
# Before (hardcoded — wrong)
@server.rtc_session(agent_name="rockgas-agent")

# After (configurable — correct)
@server.rtc_session(agent_name=cfg.agent.agent_name)
```

**Verified:**
```bash
uv run python -c "
from config import load_config
cfg = load_config()
print('Agent name:', cfg.agent.name)
print('Agent dispatch name:', cfg.agent.agent_name)
"
# Output:
# Agent name: Sam
# Agent dispatch name: rockgas-agent  ✅
```

**Why this matters:**
New customer = new `config.yml` with `agent_name: mbie-agent`. Zero changes to `agent.py`, `config.py`, or any other code file.

---

## The Two Successful Calls — Log Analysis

### Call 1 — Anonymous Caller (18:38)

```
18:38:22  received job request
          room: call-_+642108430555_kv6JmndCKi3h
18:38:25  Caller phone resolved to: +642108430555
18:38:25  Anonymous caller — using generic greeting
```

**Why anonymous?**
At this point, `known_users.json` had `+6421084300555` (extra digit) not `+642108430555`. Lookup returned None → generic greeting used.

**Conversation:**
```
"Hey. Tell me, do you deliver in Auckland?"
"No. I wanted to know what Christ has made."  ← Deepgram mishearing "Christchurch"
"I wanted to know if you also deliver in Wellington."
"Our Sydney"
"That's all. Thank you, mate."
```

**Session close:**
```
18:39:26  closing agent session due to participant disconnect
          reason: CLIENT_INITIATED  ← caller hung up
```

---

### Call 2 — Known Caller: Saurabh (18:43)

After fixing the phone number in `known_users.json`:

```
18:43:10  Caller phone resolved to: +642108430555
18:43:10  Known caller identified: Saurabh  ← 🎉
```

**Sam greeted Saurabh by name:**
```
"Welcome back Saurabh! Great to hear from you. How can I help you today?"
```

**Conversation:**
```
"Hey."
"I was just looking to check what are the services you guys offer?"
"Do you deliver in Christchurch?"
"do you also deliver in Wellington?"
"Perfect. That's all I needed. Thank you so much."
```

**Session close:**
```
18:44:00  closing agent session due to participant disconnect
          reason: CLIENT_INITIATED  ← caller hung up cleanly
```

---

### What the Log Lines Mean

| Log line | What it means |
|---|---|
| `registered worker` | agent.py connected to LiveKit Cloud, listening for jobs |
| `received job request` | A call came in, LiveKit assigned it to this worker |
| `no warmed process available` | First call — Silero VAD model not preloaded yet |
| `initializing process` | Spawning a new worker process for this call |
| `process initialized (1.84s)` | Worker ready — this is the cold start delay |
| `Caller phone resolved to` | SIP phone extraction succeeded |
| `Known caller identified` | lookup_user() found this number in known_users.json |
| `input stream attached` | Microphone audio from caller is being received |
| `Established new Deepgram STT WebSocket` | Real-time transcription started |
| `received user transcript` | Deepgram transcribed what caller said |
| `CLIENT_INITIATED disconnect` | Caller hung up (vs server-side disconnect) |
| `session closed` | Everything cleaned up correctly |

---

## Naming Decisions Made

This chat involved several important naming debates. Here's the final decision rationale:

| Item | Final Name | Why |
|---|---|---|
| LiveKit SIP trunk | `voiceagent-demo-inbound-pstn` | Platform infrastructure — generic, reused for all demos |
| LiveKit dispatch rule | `rockgas-dispatch` | Customer-specific — one per customer, name identifies who |
| Twilio SIP trunk | `voiceagent-demo-pstn-trunk` | Platform infrastructure — generic, reused for all demos |
| Agent name in dispatch rule | `rockgas-agent` | Customer-specific — must match config.yml exactly |
| `agent_name` in config.yml | `rockgas-agent` | Customer-specific — drives the decorator in agent.py |

**The pattern:**
- Infrastructure = generic name (`voiceagent-demo-*`)
- Customer-specific = customer name (`rockgas-*`)

---

## What's Shared vs Per-Customer

```
┌─────────────────────────────────────────────────────┐
│  SHARED — Set up once, never touch again            │
├─────────────────────────────────────────────────────┤
│  Twilio number: +6498869807                         │
│  Twilio trunk:  voiceagent-demo-pstn-trunk          │
│  Twilio config: number → SIP trunk                  │
│  LiveKit trunk: voiceagent-demo-inbound-pstn        │
│  LiveKit SIP URI: 5tc1esg6moj.sip.livekit.cloud     │
│  Railway deployment (Chat 16)                       │
│  agent.py, config.py, tools.py (zero code changes) │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  PER CUSTOMER — ~15 mins setup per new demo         │
├─────────────────────────────────────────────────────┤
│  customers/{name}/config.yml                        │
│    └── agent_name: {name}-agent                     │
│  customers/{name}/known_users.json                  │
│  customers/{name}/orders.csv (or equivalent data)   │
│  LiveKit dispatch rule: {name}-dispatch             │
│    └── points to {name}-agent                       │
└─────────────────────────────────────────────────────┘
```

**Adding a new customer (e.g. MBIE):**
1. Create `customers/mbie/` folder with 3 files — 10 mins
2. Set `agent_name: mbie-agent` in their config.yml
3. Create LiveKit dispatch rule `mbie-dispatch` → `mbie-agent` — 2 mins
4. Set `ACTIVE_CUSTOMER=mbie` in Railway — 30 seconds
5. Zero code changes, zero redeployment of agent.py

---

## Issues We Hit and How We Fixed Them

### Issue 1 — Old ngrok URL in Twilio Phone Config

**What we found:**
```
A call comes in → URL
https://b76e5261fe9a.ngrok-free.app
```

An old ngrok webhook was configured from earlier testing (before the proper SIP trunk architecture).

**Fix:** Changed "Configure with" dropdown from `Webhook` to `SIP Trunk`, selected `voiceagent-demo-pstn-trunk`, saved.

**Lesson:** Always check existing Twilio phone number configuration before testing. Old webhook configs silently intercept calls.

---

### Issue 2 — LiveKit UI Changed — No Credential Fields

**Expected:** Username/password fields on trunk creation form (as documented in older guides)

**Reality:** The new LiveKit UI (Feb 2026) removed credential fields from the trunk creation form. The "Allowed addresses" field replaced IP-based authentication.

**Fix:** Left "Allowed addresses" empty (accepts from any IP) — Twilio's IP ranges change frequently so IP allowlisting isn't practical.

**Lesson:** LiveKit's UI evolves quickly. Always work from what you see on screen, not from old documentation. This is exactly why we have the LiveKit MCP server — live docs.

---

### Issue 3 — SIP Trunk Dropdown Empty in Twilio Phone Config

**Symptom:** Changed "Configure with" to "SIP Trunk" but the SIP Trunk dropdown showed empty/disabled.

**Cause:** No Twilio SIP trunk existed yet — it needed to be created first.

**Fix:** Created `voiceagent-demo-pstn-trunk` in Elastic SIP Trunking first, then came back to the phone number configuration — trunk appeared in the dropdown automatically.

**Lesson:** Order matters. Create the trunk first, then attach the phone number to it.

---

### Issue 4 — Phone Number Typo in known_users.json

**Symptom:** First call showed "Anonymous caller" despite being Saurabh's number.

**Cause:** `known_users.json` had `+6421084300555` (extra `5` at end) instead of `+642108430555`.

**Fix:** Corrected the phone number in `known_users.json`.

**Second call:** `Known caller identified: Saurabh` ✅

**Lesson:** E.164 format is exact — one wrong digit means no match. Always verify numbers match exactly what Twilio passes through SIP headers. The SIP log shows the exact number: `sip_+642108430555`.

---

### Issue 5 — agent_name Hardcoded in agent.py

**Identified during:** Setting up the dispatch rule — noticed `rockgas-agent` was hardcoded.

**Fix:** Made configurable via `config.yml` → `AgentConfig.agent_name` → `cfg.agent.agent_name` in decorator.

**Lesson:** Any value that differs between customers must come from config.yml. The configurable-by-design rule applies everywhere.

---

## Complete Configuration Reference

### LiveKit Cloud Configuration

| Item | Value |
|---|---|
| Project name | voice-agent-demo |
| Project ID | p_5tc1esg6moj |
| SIP URI | `sip:5tc1esg6moj.sip.livekit.cloud` |
| Inbound trunk name | `voiceagent-demo-inbound-pstn` |
| Inbound trunk ID | `ST_ZrPoUjLXcwSe` |
| Dispatch rule name | `rockgas-dispatch` |
| Dispatch rule ID | `SDR_wPp92Dk3AAiK` |
| Agent name | `rockgas-agent` |
| Room prefix | `call-` |

### Twilio Configuration

| Item | Value |
|---|---|
| NZ phone number | `+6498869807` |
| SIP trunk name | `voiceagent-demo-pstn-trunk` |
| SIP trunk SID | `TK2386c19bbac210029fc90c4b300a2da6` |
| Origination URI | `sip:5tc1esg6moj.sip.livekit.cloud` |
| Priority | `10` |
| Weight | `10` |
| Phone number config | SIP Trunk → voiceagent-demo-pstn-trunk |

### config.yml (RockGas)

```yaml
agent:
  name: Sam
  agent_name: rockgas-agent
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

---

## Adding a New Customer Demo — The Exact Steps

For future reference — everything you need to onboard a new customer:

### 1. Code (10 mins — Cursor)
```bash
mkdir customers/mbie
# Create customers/mbie/config.yml with agent_name: mbie-agent
# Create customers/mbie/known_users.json
# Create customers/mbie/orders.csv (or relevant data)
```

### 2. LiveKit (2 mins — browser)
- Telephony → Dispatch rules → Create new rule
- Rule name: `mbie-dispatch`
- Agent name: `mbie-agent`
- Trunk: `voiceagent-demo-inbound-pstn` (same trunk, already exists)

### 3. Railway (30 seconds)
- Change env var: `ACTIVE_CUSTOMER=mbie`
- Redeploy

### 4. Twilio
- Nothing to change — same trunk, same number, same origination URI

---

## Architecture Reminder — Where Chat 8 Fits

```
Chat P  ✅  Accounts + API keys
Chat 1  ✅  Project scaffold
Chat 2  ✅  Cursor rules
Chat 3  ✅  LiveKit MCP
Chat 4  ✅  config.py — CustomerConfig dataclasses
Chat 5  ✅  RockGas customer data files
Chat 6  ✅  agent.py — Sam is live (console test)
Chat 7  ✅  verify_keys.py — 5/5 services healthy
Chat 8  ✅  SIP pipeline — Sam answers REAL phone calls
═══════════════════════════════════════════════════════
Chat 9  →   Dockerfile — containerise the agent
Chat 10 →   Railway deployment — Sam goes to production
Chat 11 →   Knowledge base documents
Chat 12 →   Order lookup tool (@function_tool)
Chat 13 →   Email capture tool (@function_tool)
Chat 14 →   KB search tool (Qdrant)
Chat 15 →   ingest.py — load KB into Qdrant
Chat 16 →   Full production demo
```

---

## Handoff Into Chat 9

Paste these 3 bullets at the start of your next session:

```
1. SIP pipeline fully configured and tested — Twilio trunk
   (voiceagent-demo-pstn-trunk) forwards calls from +6498869807
   to LiveKit SIP URI (sip:5tc1esg6moj.sip.livekit.cloud).
   LiveKit dispatch rule (rockgas-dispatch) routes to rockgas-agent.
   Two successful real calls completed — Saurabh recognised by name
   on second call after fixing phone number in known_users.json.

2. agent_name made configurable via config.yml — AgentConfig now
   has agent_name field, @server.rtc_session uses cfg.agent.agent_name.
   RockGas config: agent_name: rockgas-agent. All changes committed.

3. Ready for Chat 9: Dockerfile — containerise agent.py for Railway
   deployment. Agent currently only runs locally (uv run agent.py dev).
   Railway needs a Docker image to run it in production 24/7.
```

---

## Golden Rules — Updated After Chat 8

> **SIP trunk = platform infrastructure** — one trunk for all demos, never rename or recreate

> **Dispatch rule = per customer** — one per customer, name it `{customer}-dispatch`

> **Agent name must match exactly** — dispatch rule `agent_name` must equal `@server.rtc_session(agent_name=...)` must equal `config.yml agent_name`

> **E.164 is exact** — one wrong digit = no caller match. Verify against SIP logs (`sip_+64XXXXXXXXX`)

> **Check existing Twilio config** — old webhooks silently intercept calls. Always verify "Configure with" setting.

> **Create trunk before attaching number** — order matters in Twilio

> **Allowed addresses empty = accept from any IP** — correct for Twilio (their IPs change)

> **`agent_name` in config.yml** — any value that differs between customers must come from config

> **Run `uv run agent.py dev` before calling** — agent must be running to answer

> **Warm up before demos** — free plan has 10-20s cold start on first call. Call yourself first.

> **`CLIENT_INITIATED`** — caller hung up normally. `SERVER_INITIATED` or errors = investigate

---

*voice-agent-demo | Chat 8 Learning Notes | 28 February 2026*
