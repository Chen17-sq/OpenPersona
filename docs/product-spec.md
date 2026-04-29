# OpenPersona — Product Specification

> Status: v0.0.1 · macOS only · pre-alpha

This is the canonical product definition. When new features are
proposed, we test them against the principles here and the
[sharpness invariant](#sharpness-invariant). Engineering details live
in [architecture.md](architecture.md), [promise-kinds.md](promise-kinds.md),
and [extension-protocol.md](extension-protocol.md).

---

## 1. What OpenPersona is

**A layer that turns your real conversations into structured rows
any AI agent can query: People, Promises, Events.**

What you owe Bob by Friday. When you last saw Sarah. The Q3 deal you
promised Tom on the last call. Concrete units of a relational life —
extracted from real activity, stored on your machine, exposed via MCP.

| Primitive | What it captures |
|-----------|------------------|
| **Person** | who you know — including yourself (`me`) |
| **Promise** | bilateral commitments — what's owed, by whom, by when |
| **Event** | discrete moments — meetings, calls, message threads |

Facts / Reminders / Sources are scaffolding around the three; they
don't have UI surfaces of their own.

> "AI memory" is consensus by now. The reason it all feels the same:
> every product stops at "remember stuff said inside this app." We
> model the three things that actually matter.

The graph lives as plain SQLite + Markdown on your machine, sourced
from your real activity (WeChat, iMessage, calendar, screen, audio
via collectors), exposed to any agent through MCP — owned by you,
portable across every AI you'll ever use.

---

## 2. Target user (one persona, narrow)

**Senior IM-heavy professional on macOS**: founder / VC / BD / sales /
consultant / lawyer — anyone whose deal-flow lives in chat threads.

| Trait | Description |
|-------|-------------|
| Where work happens | iMessage + WhatsApp + WeChat + Telegram + Outlook + calendar — multi-channel, no single source |
| Volume | 5+ active group threads + 20+ active 1:1 chats across two or more apps per week |
| Pain | "I forgot what I owe whom"; "the relationship graph only lives in my head"; "I need to switch deal contexts 10 times a day across 3 apps" |
| Already paying for | Notion / Granola / Superhuman / Folk — none reach into IM and stitch across channels |
| Unmet need | A surface that auto-extracts "what was promised, by whom, to whom, when" from every channel they actually use |

**Not for**: students, casual users, anyone whose work isn't
deal-flow-shaped, or anyone who lives entirely in one app already
(Granola covers that case for meeting-only people).

---

## 3. The wedge (the one thing v0 must do)

**Promise Graph** — extract every commitment from your IM into a
`(committer, committee, what, when, source)` tuple, lay them out on a
People × Time grid, write them to your calendar in one click.

The 30-second demo:

> "Look — 47 commitments extracted from your last 7 days of iMessage
> + WeChat. Red = overdue. Click ✅ → it's in your macOS Calendar."

---

## 4. The four surfaces (v0 — only these four)

1. **Persona Self-View** — your own facts, fitness goals, recent
   activity, the 5 people you talked to most.
2. **Promise Grid** — `People × Time` matrix; red = overdue, yellow =
   soon, green = on track. Each cell links to a Promise Card.
3. **Persona Card** — one person at a time: facts / dates / reminders /
   promises (you owe / they owe) / recent interactions.
4. **Inbox** — low-confidence extractions waiting for your 1-click
   confirm; in v1, also surfaces *inferred* promises (the "you should
   reach out" / "Bob hasn't replied in 3 days" pattern).

No settings page, no dashboard, no team collab in v0.

---

## 5. The Persona model — three primary primitives + three supporting

### Primary (the things we compete on)

```
People       — anyone you know, including 'me' (self is one row)
Promises     — bilateral commitments with 2 kinds × 3 anchors
Events       — meetings / calls / message threads / focus blocks
```

These three are **the user-facing reality** of OpenPersona. Every
surface (Persona Card, Promise Grid, Inbox) renders one or more of
these. Every MCP tool query returns combinations of these. Every
extractor's output reduces to writing these.

### Supporting (scaffolding around the primary three)

```
Facts        — static knowledge that decorates a Person
                 (namespaced by key prefix; new domains = new prefix)
Reminders    — flexible triggers (5 kinds) that schedule actions on
                 People / Promises / Events
Sources      — provenance for every Fact / Promise / Event so anything
                 traces back to a real message or capture
```

These three exist *for* the primary three. They have no UI surface of
their own — they show up *inside* a Persona Card, *next to* a Promise
in the Grid, *behind* an Event's "open original message."

### Self is just a Person

Self is `person.id = 'me'`. **No code path special-cases self.** Your
fitness goals are facts on `me`, your reminders to call mom are
reminders on `me`. Same primitives.

### 5.1 Promise — 2 kinds × 3 anchors

| | Explicit | Inferred (v1) |
|---|---|---|
| **Origin** | Stated in a message | Derived from patterns / silence |
| **Example** | "I'll send you the deck tonight" | "Bob hasn't replied in 3 days" |
| **Confidence** | usually 0.7+ | usually 0.5–0.7 |
| **UX** | Auto-published to Grid | Lands in Inbox first |

| | Event-anchored (v1) | Time-anchored (v0) | Open |
|---|---|---|---|
| **When** | "before the GLV quarterly" | "Friday 5pm" | "soon" |
| **Schema** | `when_event_id` | `when_iso` | both NULL |
| **UX** | "before the GLV quarterly" displayed | timestamp displayed | "(no deadline)" |

Full model: [promise-kinds.md](promise-kinds.md).

### 5.2 Facts — namespaced keys, never new tables

Static knowledge about a person, organised by `key` prefix:

| Prefix | Purpose | Examples |
|--------|---------|----------|
| `bio:` | structured profile | `bio:company`, `bio:role`, `bio:spouse` |
| `tag:` | labels | `tag:VC`, `tag:AI`, `tag:morning_person` |
| `preference:` | soft preferences | `preference:food`, `preference:communication` |
| `date:` | recurring dates → auto-materialize yearly reminders | `date:birthday`, `date:anniversary` |
| `health:` (v1) | self-tracking | `health:weight`, `health:run_freq` |
| `read:` (v1) | reading list | `read:current_book` |
| `note:` | free-form bullets | `note:hobbies`, `note:trivia` |

**A new "domain" is a new key prefix, never a new table.** The UI
auto-groups facts by prefix into labeled sections. Users can invent
their own prefixes; the product extends without code changes. Full
rules: [extension-protocol.md](extension-protocol.md).

### 5.3 Reminders — five trigger kinds, one table

```
absolute            — fixed datetime
recurring_yearly    — auto-materialized from date:* facts
recurring_interval  — every N days, anchored to last_meeting | last_fire
after_event         — fires when event_kind happens (e.g. next_meeting_with)
before_event        — fires N hours/days before event_id occurs
```

A new trigger type = one new `compute_next_fire(...)` function. **No
schema change.** Birthdays are not a special feature — they're a
`recurring_yearly` reminder auto-materialized from a `date:birthday`
fact, deleted when the fact is.

---

## 6. What we explicitly do NOT build

| ❌ Out of scope | Reason |
|----|---|
| **OCR / VLM / screenshots / audio capture** | Collector layer — Paperboy / Granola / others bring it via the Collector Protocol |
| Calendar app | macOS Calendar / Notion Calendar exist — we **write** to them |
| CRM | Folk / Attio exist — we **extract** the relationships |
| IM client | WeChat / iMessage stay where they are — we **read** local DBs |
| Meeting transcription | Granola / Otter exist |
| General agent | We're the **layer** agents query, not the agent |
| Generic todos / chores | Reminders.app handles those — we only do (person × time) |
| Team collaboration / sharing | v0 single-user only |
| iOS / Android | v0 macOS only |
| Cloud sync (default) | Zero bytes leave your machine in v0 |

The full anti-bloat list is enforced by the [sharpness invariant](#sharpness-invariant).

---

## 7. Architecture — a layer, not an app

```
┌──────────────────────────────────────────────────┐
│  Surface (we build)                              │
│  - SvelteKit UI (localhost:7600)                 │
│  - MCP server (v1)                               │
└──────────────────────────────────────────────────┘
                    ▲
┌──────────────────────────────────────────────────┐
│  Persona Graph (we build, the core asset)        │
│  - SQLite: people / facts / promises / events /  │
│    reminders / sources                           │
│  - Markdown: per-person files                    │
└──────────────────────────────────────────────────┘
                    ▲
┌──────────────────────────────────────────────────┐
│  Extractors (we build, pure functions)           │
│  - commitment / facts / entity_resolver          │
└──────────────────────────────────────────────────┘
                    ▲
┌──────────────────────────────────────────────────┐
│  Collector Protocol (we publish; anyone can      │
│  implement)                                      │
│                                                  │
│  Built-in (v0): iMessage, WeChat, Calendar       │
│  Built-in (v1): WhatsApp, Telegram, Outlook      │
│  External:      Paperboy MCP, Granola, your own  │
└──────────────────────────────────────────────────┘
```

Full architecture, dependency rules, invariants:
[architecture.md](architecture.md).

---

## 8. Sharpness invariant

> Before adding any feature, ask:
>
> **Does this feature reduce to `(Person, Promise, Event)`, where
> `Person` may be `me`?**
>
> Time is a coordinate on Event, not a primary axis.
> Memory is the byproduct, not the goal.
>
> Yes → it fits, and likely lands as either a new fact key prefix
> (decorating People), a new trigger kind (scheduling on the
> primaries), or a new collector (feeding the primaries).
> No → it doesn't belong here.

Examples:

| Proposal | In or out? | How |
|---|---|---|
| Track my weight over time | ✅ in | `health:weight` facts |
| Reminder for Bob's birthday | ✅ already in | `date:birthday` → recurring_yearly reminder |
| Track current book | ✅ in | `read:current` fact |
| Tag people by relationship | ✅ in | `tag:colleague` |
| Auto-write meetings to calendar | ✅ in | EventKit collector + write |
| Buy milk reminder | ❌ out | not Persona-shaped |
| Pomodoro timer | ❌ out | bespoke entity |
| Free-form journal | ❌ out (revisit v2) | needs different design |

---

## 9. Privacy stance

| Concern | Position |
|---|---|
| Cloud upload | **Zero bytes** by default. Hosted sync layer is opt-in for v2 only. |
| LLM API key | BYO; stored in macOS Keychain, never in config files. Ollama / LM Studio fully supported (offline). |
| Sensitive content | Credit cards / verification codes / password formats are filtered at the **collector layer**, never reach the graph. |
| Per-source kill switch | Disable any contact / group / app at any time. |
| Forget right | `openpersona forget --person <id>` removes from SQLite + Markdown + sources. |
| LLM audit | Every LLM call logged locally (prompt summary + tokens + cost), inspectable. |

---

## 10. Distribution thesis (v0 launch)

| T+ | Channel | Reason |
|---|---|---|
| Day 0 | Hacker News Show | Dev / tech-Twitter reach; iMessage angle plays well |
| Day 0 | wechat-cli user group (425⭐ existing) | Warmest audience for the WeChat collector specifically |
| Day 1 | Twitter (English + Chinese) / V2EX | Tech crowd, both audiences |
| Week 1 | Long-form blog (knowable / Substack / Zhihu) | The "three primitives" thesis; concrete demo |
| Week 2 | Direct DMs to founders / VCs / BD using IM heavily | Persona-aligned audience |
| v1 | HN second push (with WhatsApp + Telegram + MCP) | "Now your Persona works across every IM you use" |

**Positioning line for HN**: "OpenPersona — your AI memory you carry
between agents. Local-first. Reads every IM you actually use."

---

## 11. Success metrics for v0 (4 weeks out)

| Metric | Pass bar |
|---|---|
| End-to-end: iMessage + WeChat → Promise Grid → Calendar | works on demo machine |
| Commitment extractor recall on real 7-day data | ≥ 80 % vs hand-labeled |
| Commitment extractor false-positive rate | ≤ 10 % |
| GitHub stars after launch week | ≥ 100 |
| Hacker News front page (any time) | yes / no |
| User dogfooding: founder uses it for own deals | daily for ≥ 1 week |

Stretch: a real reviewer (not the maintainer) shipping a PR back into
upstream within 2 weeks of launch.
