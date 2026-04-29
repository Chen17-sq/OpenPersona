<h1 align="center">OpenPersona</h1>

<p align="center">
  <b>People. Promises. Events.</b><br/>
  Three primitives. One local graph. Any agent.
</p>

<p align="center">
  <code>v0.0.1</code> · macOS only · <b>private alpha</b> · public release ~6 weeks
</p>

---

What you owe Bob by Friday. When you last saw Sarah. The Q3 deal you
promised Tom on the last call.

OpenPersona extracts that from your real conversations — iMessage,
WeChat, WhatsApp, Telegram, Outlook, your calendar — into a local
graph any AI agent can query through MCP.

Your Persona, your machine, your keys.

> "AI memory" is consensus by now. None of those products know about
> Bob's deck. **We do.**

---

## Use cases

Four moments where OpenPersona earns its keep. Every one is the kind
of "I should've remembered, but I didn't" you've had this quarter.

### "Wednesday night I promised Bob the OnePager. Friday morning I forgot."

> **Bob**, Sequoia partner. WhatsApp, Wednesday 11:43 PM:
> *"Send me the OnePager by EOD Friday? Need the latest numbers from the data room."*
> You: *"Sounds good — Friday EOD."*

Thursday you're heads-down with two other LPs in iMessage. Friday
9 AM you open your laptop — Bob isn't on your inbox or in your head.
He doesn't ping you. Saturday morning he's circling on a different
deal.

What you actually see Friday 9 AM, before email:

> **Today · 3 things**
> 1. **send Bob the OnePager** · *due in 8h · me → Bob*
> 2. Reach out to Sara · *silent 42d, 2 pending*
> 3. GLV quarterly · *2 promises tied to this · 14:00*

You ship at 9:30. Bob replies same day. Different ending.

### "She asked 6 weeks ago. I said 'this weekend.' Twice."

> **Sara**, your 2024 co-founder. iMessage, 42 days ago:
> *"Can you take a quick look at my YC app? Deadline Wednesday."*
> You: *"Sure — this weekend."*

Three weeks later she asked again. You said yes again. You forgot
again. Tomorrow your calendar has *"kickoff 10am"* on it — with her,
but Calendar.app didn't resolve the participant.

You open **The Week**. Kickoff has a red border: *"floating — no
people linked"*. Click `[+ Add person]` → "Sara" → the card flips:

> **Sara** · 1 promise overdue · *42 days* · "review YC app"

You block 30 min that afternoon. Review goes out the night before
the kickoff. She walks in not having to ask. Relationship lives.

### "Five apps, three deals — what did I actually promise?"

You're tracking **Mike** (founder, Q3 pipeline) across two channels
in one week:

| When | Where | What |
|---|---|---|
| Tue 9pm | Telegram | "Will review the deck by Wednesday" |
| Thu 11am | iMessage | "Term sheet to you by Monday" |

Five days later you sit down to follow up. The deck review you
remember (you did it Wednesday). The term sheet has slipped through
your head entirely — different app, different thread. Searching
iMessage finds 180 messages, none of them obviously this.

Promise Grid, Mike's row:

| dir | what | status |
|---|---|---|
| me → Mike | review deck | ✅ done |
| me → Mike | term sheet | 🔴 **overdue 2d** |

You catch it. Term sheet goes out that afternoon — before he asks.

### "Maya just emailed. I'm in Cursor. What did I owe her?"

> **Maya Chen**, BD lead at a portfolio co. Last quarter Q3 sync:
> *"Could you intro me to 3 senior ML candidates over the next month?"*
> You: *"Yeah — let me line them up."*

You introed 1. Forgot the other 2 (you got distracted by a fundraise).
Today Maya emails wanting to sync at 4pm. You're in Cursor coding the
portfolio dashboard. You don't switch apps — you ask Claude:

> *"What did I promise Maya last quarter, and where do I stand?"*

Claude calls OpenPersona's MCP server (`who_owes_me`,
`recent_interactions`), reads your **local** Persona graph, replies:

> **Maya Chen** — 2 ML intros outstanding · *6 weeks pending* · last
> sync was the Q3 BD review where you committed to "3 senior ML
> engineering intros".

You fire off 2 intros in the 5 minutes before the call. Your chat
data never left your Mac. Maya never has to remind you.

### "I haven't seen David in a year. Meeting in 3 hours. Cold."

> **David Liu**, sat at the same portfolio dinner 11 months ago.
> Today his EA put a 1-on-1 on your calendar — no agenda. You don't
> remember what he does. You can't dig through three message apps
> in 5 minutes. You feel slightly embarrassed.

Click `[▸ Brief]` from The Week:

> **Briefing · in 3h**
> **David Liu** · Verge Capital · BD lead
>
> **Top of mind**
> · birthday — May 6 (in 9 days, Taurus)
> · school: Stanford CS '14 — same as you
> · work: co-founded XYZ, acquired by Google 2022
> · tag: VC · tag: AI-infra
>
> **Where you stand**
> · You owe: 2 ML intros (6 weeks pending — same row)
>
> **Last we talked** · 11mo ago, iMessage:
> > *"let's pick up after Q4"* — no follow-through
>
> **Worth bringing up**
> · He mentioned wanting intros to portfolio engineers (last sync,
>   no resolution)

You walk in knowing his backstory, the unfinished thread, and his
birthday. Awkward becomes warm — and the thread that's been
hanging for 11 months finally closes.

The same panel flips to **Recap** mode 2 hours after the meeting:
*"What was discussed"* (chat excerpts ±2h of the meeting time),
*"Loose threads still open"* (anything mentioned that didn't
crystallize into a commitment), feeding low-confidence items
straight into the Inbox for one-click confirm.

---

## Screens

> Screenshots get refreshed on each public-alpha milestone — see
> [`docs/screenshots/README.md`](docs/screenshots/README.md) for
> the capture process. Currently captured at v0 code-complete (~Apr 29 2026).

| | |
|---|---|
| ![Self-View](docs/screenshots/self-view.png) | **Self-View · `localhost:7600/`** — *Today · 3 things* leads, then by-the-numbers, then overdue / due-this-week / recent-people. The first thing you see in the morning. |
| ![The Week](docs/screenshots/the-week.png) | **The Week · `localhost:7600/calendar`** — events grouped by day, foregrounded by people. Floating events have red borders + a `[+ Add person]` flow. Each card has `[▸ Brief]`. |
| ![Briefing panel](docs/screenshots/briefing-panel.png) | **Briefing panel** — slide-out from any event. *Top of mind* (school, work, birthday) · *Where you stand* (open promises both ways) · *Last we talked* · *Worth bringing up* (loose threads). |
| ![Recap panel](docs/screenshots/recap-panel.png) | **Recap panel** — same surface, flipped to past events. *What was discussed* · *Loose threads still open* → feeds the Inbox. |
| ![Inbox edit](docs/screenshots/inbox-edit.png) | **Inbox · `localhost:7600/inbox`** — accept / dismiss / *edit* the wording, deadline, who-promised-whom inline. Optional ☑ schedules a focus block 24 h before the deadline. |
| ![Persona Card](docs/screenshots/persona-card.png) | **Persona Card · `localhost:7600/people/<id>`** — Next-meeting hint + `[+ Add fact]` + `[✨ Enrich with LLM]` at the top, then facts grouped by prefix, then promises both directions. |
| ![The Network](docs/screenshots/the-network.png) | **The Network · `localhost:7600/network`** — force-directed relationship graph. Nodes scale by pending-load. Black centre = `me`. Red = high-load relationships. Click a node → that Persona Card. |

---

## Features — what you see and use

The six surfaces in the local web UI at `localhost:7600`. Each one
renders a slice of your Persona graph; what's underneath is in the
next section.

| Surface | What you do there |
|---|---|
| **Today · 3 things** | The first thing on the Self-View. Top 3 priorities ranked deterministically across overdue / due-today / today's events / silent-contacts-with-pending-promises. No decision fatigue, no "AI ranking" black box. |
| **Promise Grid** | `People × Time` matrix. Red = overdue, yellow = due soon, green = on-track. Click a cell to drill into the underlying promises. |
| **The Week** | This week's events grouped by day, **foregrounded by people, not by hour-of-day**. Person chips + an "N promises due before this meeting" hint on every card. Floating events (no participants resolved) get a 1-click binding flow. Each event has a `[▸ Brief]` panel that flips between **briefing** (future event: top-of-mind facts, last we talked, worth bringing up) and **recap** (past event: what was discussed, loose threads). |
| **Persona Card** | One person at a time: facts grouped by prefix (`bio:` / `tag:` / `preference:` / `date:` / `health:` / `read:` / `note:`), promises in both directions, recent interactions. Every row traces back to the source message. |
| **Inbox** | Low-confidence extractions awaiting your call. Accept · dismiss · or **edit** the wording, deadline, or who-promised-whom inline before keeping it. Optional ☑ schedules a 1-hour focus block 24 h before the deadline. |
| **The Network** | Force-directed graph of People as nodes, Promises + shared Events as weighted edges. Red = high-load relationship (≥ 5 pending). Click a node → that Persona Card. The "deal-flow user" view of who's connected to whom. |

## Capabilities — what runs underneath

The infrastructure that makes those surfaces possible. Most of these
you don't see directly — but losing any of them collapses the
product.

| Capability | What it powers |
|---|---|
| **Multi-collector ingestion** | iMessage + WeChat (live) · WhatsApp + Telegram + Outlook + Notion + Linear (skeletons, real-data tuning pending) · macOS Calendar · your own via the [Collector Protocol](docs/architecture.md). One protocol, every channel you actually use. |
| **Auto-extract daemon** | launchd ingests new messages every 5 min from your wired collectors — no manual `extract` typing. Per-collector state so failures retry without re-scanning. |
| **LLM extractors** | Two: the **commitment extractor** (high-confidence explicit promises stated in a message) and the **expectation extractor** (low-confidence inferred promises — unanswered questions, stale "let me check" offers, cadence breaks). BYO LLM key (DeepSeek / OpenAI / Anthropic / Ollama). |
| **Calendar bidirectional** | `push-calendar` writes promises as macOS Calendar events; `pull-calendar` reads events back into the graph; event reschedules cascade to anchored promises so deadlines move with their underlying meetings. |
| **MCP server** (10 tools) | Claude Desktop / Cursor / any MCP client queries your Persona graph over stdio. 8 read tools + 1 write-via-Inbox tool + 1 briefing-synthesis tool (`event_brief`). The agent-portable layer that turns your Persona into universal AI memory. See [`docs/mcp.md`](docs/mcp.md). |
| **Local-first storage** | SQLite + per-person Markdown on your Mac. Atomic writes + per-path locks. Zero cloud, ever. The graph is `git diff`-able and survives the product. |

---

## What's in your Persona

The three things we compete on — extracted from real conversations:

```
People    — who you know (yes, including yourself, just another row)
Promises  — what you owe and what's owed, with deadlines that move
            when the underlying events do
Events    — meetings, calls, threads — the moments your life actually has
```

Scaffolding around them:

```
Facts      — properties that decorate People (namespaced keys; new
             domains add a prefix, never a table)
Reminders  — five flexible triggers; birthday isn't a feature, it's a
             date: fact materialised into a reminders row
Sources    — every fact traces back to a real message you can re-read
```

That's it. Six tables. Forever.

---

## Architecture — we own the graph, not the cameras

```
surface       SvelteKit UI (localhost:7600) + MCP server (9 tools, stdio)
persona       SQLite + Markdown — atomic, lockable, git-diffable
extractors    LLM-driven: commitment + expectation (inferred), facts,
              entity resolution, event-phrase resolver
─────────── Collector Protocol ───────────
collectors    iMessage · WeChat · WhatsApp · Telegram · Outlook ·
              macOS Calendar (bidirectional) · Paperboy MCP · Granola · yours
```

We extract structured rows from conversations. We **don't** capture
screens, OCR images, or transcribe audio — those plug in via the
Collector Protocol. Bring your own Paperboy / Granola / Whisper. Every
new collector enriches every OpenPersona user's graph; we compete on
the layer above.

The MCP server exposes the graph to Claude Desktop / Cursor / any
MCP-compatible agent — so you can ask "what did I promise Bob?" or
"what's overdue?" without leaving the chat. See [`docs/mcp.md`](docs/mcp.md)
for the 9-tool inventory and Claude Desktop / Cursor wiring.

---

## Currently in private alpha

The code is closed for now while the prompts, extractors, and data
model mature against real usage. Public release follows when:

- 3+ external dogfood users have run it for a week without crashes
- Extractor recall on real (not fixture) data is ≥ 80 %
- The MCP integration is verified live in Claude Desktop + Cursor

Roughly **~6 weeks out** from this writeup, give or take.

### Engineering status

The README isn't aspirational. The private core (separate repo,
not linked here on purpose) is actively developed today:

- **6 UI surfaces · 10 MCP tools · 8 collector adapters · briefing/recap panel · network graph viz · backup/restore + install + doctor + forget CLIs** — every
  capability listed above is built and runs locally on the author's
  Mac. Skeletons are honestly flagged in the Capabilities table
  pending real-data tuning.
- **386 tests passing · ruff clean · pip-audit 0 CVEs · bandit baseline-clean · prompt-baseline locked** —
  green on every commit; security tooling pinned in dev deps;
  prompt files hashed so any change requires a deliberate baseline bump.
- **100+ commits** since first push; daily activity visible on
  [@Chen17-sq](https://github.com/Chen17-sq)'s public contribution
  graph (private repo commits surface there with "private contributions
  enabled" turned on).

What's gated is real-data validation, not the build itself. Source
opens when the extractors hold up against a week of someone's
real iMessage / WeChat history.

### Want early access?

- **Email** — [schen.aldrich@gmail.com](mailto:schen.aldrich@gmail.com)
  with one line about how you'd use it
- **LinkedIn** — [Aldrich Chen](https://www.linkedin.com/in/aldrich17siqi/)
- **WeChat** — `18574843907` (for Chinese-speaking users; mention OpenPersona)
- **Open an issue** here describing your use case — I read every one
  and it shapes the v0.1.0 cut

---

## Privacy & security

- **Zero cloud by default**. Every byte stays on your Mac.
- **BYO LLM key** (OpenAI / Anthropic / DeepSeek / Ollama / LM Studio). Stored in macOS Keychain.
- **Per-source kill switch** — disable any contact, group, or app.
- **One-shot forget** — `openpersona forget --person <id-or-name>` atomically wipes a person from SQLite (people, facts, promises, reminders) + strips them from event participants + deletes the Markdown file. `--before <ISO>` for bulk redaction. Audit-logged.
- **Snapshot + restore** — `openpersona backup` writes a `0600` tar.gz; `openpersona restore` validates archive members and rolls back with auto-quarantine of the live state.
- **One-command launchd install** — `openpersona install` writes both plists (tick + auto-extract) and bootstraps them. `openpersona doctor` is the all-in-one readiness diagnostic.
- **Sensitive content filter** — credit cards, verification codes, password formats dropped at the collector layer, never reach the graph.
- **Localhost-only API** by default; CLI refuses non-loopback bind without `--allow-public`. CSP + `Cache-Control: no-store` + `X-Frame-Options: DENY` on every response. Audit log of every mutation.
- **Persona dirs chmod'd to 0700** on every start; backup tarballs chmod'd to 0600. Markdown sync refuses path-traversal ids.
- **Optional bearer-token auth** (`OPENPERSONA_AUTH_TOKEN`) for the paranoid; constant-time compare; off by default.
- **Per-person rate limit** on the LLM enrich endpoint (1 / 60 s).
- **PII-redaction helper** scrubs API keys / emails / phone numbers from log messages before they hit syslog.

Audit posture:
- **0 CVEs** across 90+ transitive dependencies (`pip-audit`).
- **0 new findings** in the static security scan vs baseline (`bandit`).
- **All ruff checks passing** on every commit.

Full threat model + posture in the source repo at `docs/security.md` (becomes public when v0.1.0 ships).

---

## Roadmap

| | What | Status |
|---|---|---|
| **v0** | 5 UI surfaces · Promise Grid · Calendar push/pull · Inbox edit · MCP (9 tools) · auto-extract daemon · macOS-only | code-complete, awaits real-data dogfood |
| **v0.5** | WhatsApp / Telegram / Outlook collector skeletons · expectation extractor (inferred promises) · event-anchored promise resolution · focus-block scheduling | shipped as skeletons; prompt + thresholds tuned against dogfood data |
| v1 (+2 mo) | Real-data validation pass · few-shot examples in expectation prompt · event-anchored reminders UX · public launch (HN / Twitter / dev blog) | — |
| v2 (+5 mo) | Apple Health collector · Notion / Linear sync · relationship visualization · plugin marketplace · MCP write-tool expansion | — |

The "skeleton" honesty: WhatsApp / Telegram / Outlook collectors,
inferred-promise extractor, event-anchored resolver — all are wired
end-to-end with mocked transports + ~280 tests, but haven't yet run
against real wacrawl output / real Telegram dialogs / real Outlook
mailboxes / real chat windows. Real-data tuning is exactly the
private-alpha work.

See [`docs/roadmap.md`](docs/roadmap.md) for the day-by-day breakdown,
[`docs/product-spec.md`](docs/product-spec.md) for the canonical
product definition.

---

## Contributing

The code is private until v0.1.0, but the **ideas are open**:

- **Collector Protocol** — if you maintain a context-aware product
  (Paperboy, Granola, a meeting tool, a wearable, a custom internal
  tool) and want to feed OpenPersona's graph, the protocol is
  documented in [`docs/architecture.md`](docs/architecture.md) and
  [`docs/extension-protocol.md`](docs/extension-protocol.md). Email
  to co-design the integration.
- **Use-case feedback** — open an issue describing how you'd use
  OpenPersona; this directly shapes the v0.1.0 cut.
- **Bug reports** from private-alpha users welcome (alpha users have
  the source repo URL).

---

## License

The documentation in this repo is MIT. The product code is private
until v0.1.0; the planned public license at that point is MIT.
