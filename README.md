<h1 align="center">OpenPersona</h1>

<p align="center">
  <b>An AI that remembers your people for you.</b>
</p>

<p align="center">
  The open-source, AI-native relationship memory layer —<br/>
  across <b>iMessage</b> · <b>WhatsApp</b> · <b>Telegram</b> · <b>WeChat</b> · <b>Slack</b> · <b>Gmail</b> ·<br/>
  and any chat you live in.
</p>

<p align="center">
  <code>v0.4.2</code> · macOS · <b>private alpha</b> · public source ~v0.5
</p>

---

## Why this exists

I built a feature graveyard.

Two weeks ago, OpenPersona was a *"local-first commitment graph"* — extract every promise,
fact, and event from your IM history into a queryable database, then visualise it. I built
30+ CLI commands. 5 dashboard views (Today / Inbox / Grid / Self View / Search). 5
collectors. A launchd daemon. A menubar app. A 15-tool MCP server. SvelteKit + FastAPI +
SQLite + Markdown round-trip. 700+ tests. It looked impressive on paper.

I barely opened it.

Two weeks of dogfood is short, but the signal was already loud: of 30+ CLI commands, I used
4. I never voluntarily opened Inbox or Grid. Of 200 extracted promises, half were noise.
The product's most damning flaw: **the author wasn't using it.**

The bug wasn't quality. It was *category*. I had built a SaaS-style CRM for a job that
doesn't have a SaaS shape. Real relationships aren't promises and facts in a table —
they're attention, texture, time. The "open the dashboard daily" pattern is the graveyard
of wellness apps and BI tools. I had walked straight into it.

So I deleted half the codebase and pivoted the frame. The data layer stayed (SQLite +
Markdown round-trip is the one piece worth keeping). Everything else got rebuilt around
one sentence:

> **OpenPersona doesn't ask you to open it. It remembers your people for you.**

The product is now mostly invisible. It lives in surfaces you already use:

- Your **lock screen** — a one-line glance, all day
- Your **push notifications** — voice-matched, AI-written, only when it matters
- Your **Apple Calendar event description** — pre-meeting brief auto-injected before you arrive
- A **natural-language chat hotkey** (⌘⇧Space) — ask anything about anyone

Dashboards still exist as backstop. They're hidden by default. **If you find yourself
opening this app daily, I've failed at the job.**

---

## What it does

Four surfaces. That's the product.

### 🔒 Lock screen widget
A single line of text on your lock screen. *"Maya · silent 21d."* *"Vincent at 14:00."*
Glanceable. Updated continuously. Native macOS WidgetKit (Swift).

### 📱 Push notifications
The product speaks when it has something to say.

- **Morning push (8 am)** — *"Today · 3 things."* Concrete, actionable, ranked deterministically.
- **Pre-meeting brief (5 min before)** — *"Vincent @ Cathay · last talked 9d · 'intro to Eileen' open."*
- **Drift detection** — when an inner-circle relationship goes silent past your baseline.

Every notification is **voice-matched**: written by AI in your tone, generated per-recipient,
never templated. One tap reveals a draft reply ready to send through the platform's native
app.

### 📅 Apple Calendar event auto-prep
For any calendar event with a known person, OpenPersona writes a pre-meeting brief into
the **event's description field** N hours before the meeting. You open Calendar.app (your
existing workflow); the brief is already there. No new app to check. No tab to remember.

### 🔍 Natural-language chat (⌘⇧Space)
A global hotkey opens an LLM chat anchored to your relationship graph. Ask anything:

```
> who's at Cathay?
> what did Bob say about the deck last week?
> draft a reply to Maya in my voice
> who should I reach out to this week?
```

Replaces every traditional search bar, filter, and dashboard query.

---

## Where the data comes from

OpenPersona reads conversations from the IM platforms you actually live in.

| Platform | Status | Notes |
|---|---|---|
| **iMessage** | ✅ shipping | Direct `chat.db` read · SMS spam filter · service-aware |
| **WeChat** | ✅ shipping | Via [`wechat-cli`](https://github.com/Chen17-sq/wechat-cli) subprocess |
| **WhatsApp** | ✅ v0.4 | `wacrawl` live + Export-Chat .txt offline path (`OPENPERSONA_WHATSAPP_EXPORT_DIR`) |
| **Telegram** | ✅ v0.4 | Telethon live + Telegram Desktop result.json offline path (`OPENPERSONA_TELEGRAM_EXPORT_JSON`) |
| **Gmail** | ✅ v0.4 | Stdlib `imaplib` IMAP path · Gmail API via stdlib OAuth (`op auth gmail`) |
| **Outlook** | ✅ v0.4 | Microsoft Graph live + .eml directory offline (`OPENPERSONA_OUTLOOK_EML_DIR`) |
| **Slack** | ✅ v0.4 | Workspace export / `slackdump` offline (`OPENPERSONA_SLACK_EXPORT_DIR`) |
| **Discord** | ✅ v0.4 | Official data export + DiscordChatExporter JSON (`OPENPERSONA_DISCORD_EXPORT_DIR`) |
| **Apple Health** | ✅ v0.4 | Export ZIP → `health:*` facts on `me` (`op apple-health-sync`) |
| **Your platform** | 🤝 anytime | [Collector Protocol](docs/architecture.md) — 3 record types, 2 functions, ship a PR |

The collector layer is **one protocol** — three record types (`Source`, `Message`,
`Event`), two functions to implement (`iter_messages`, `iter_events`). Anything that
emits messages can plug in. Every new collector enriches every OpenPersona user's graph;
we compete on the layer above.

---

## Honesty: cost, privacy, what we send to which AI

OpenPersona is local-first **storage**. LLM extraction calls a cloud provider you choose.
We're explicit about this because the alternative is dishonest.

- **Storage**: every byte of your graph stays on your Mac. SQLite + per-person Markdown,
  `0700` directory permissions, atomic writes.
- **LLM calls**: per-message extraction sends the message text to the provider you've
  configured (DeepSeek default, OpenAI / Anthropic / Ollama supported).
- **Cost** (DeepSeek V4 Flash, default): roughly **$0.50–$5 per month** depending on IM
  volume + AI-native depth (per-meeting briefs, voice-matched pushes, weekly persona
  refresh).
- **Fully local mode**: pass `--local-only` to route everything through Ollama (Qwen
  32B or similar). Slower; quality varies; your data never leaves your Mac.
- **Sensitive content filter**: credit cards, OTP codes, password formats are dropped at
  the collector layer before any LLM sees them.
- **Per-person killswitch + bulk forget**: `op forget --person <id>` atomically removes a
  person from every table, audit-logged.

**Posture (private alpha)**: 0 CVEs across 90+ transitive deps · `bandit` baseline clean
· `ruff` green on every commit · **982 tests** across collectors / extractors / store /
API / MCP / CLI / agent layer / SPA / CSP / inline-edit / widget xcodeproj generator /
schema index audit / Gmail OAuth flow / MCP write-gate / schema versioning / WeChat
group classification. Full threat model ships in the alpha bundle and moves to
`docs/security.md` at public source open.

---

## Try it — for private alpha users

If you have a private alpha invite, you got a `.whl` plus an `install.sh` — drop both in
a folder, run:

```bash
bash install.sh
op quickstart
```

Within 30 seconds of `quickstart` finishing, you'll get your first push.

Day-to-day commands (full set in `op --help`):

| Command | What it does |
|---|---|
| `op remember` | Run extraction (incremental — picks up only new messages) |
| `op recall <person>` | Generate a brief for any person on demand |
| `op forget --person <id>` | Atomic delete across every table |
| `op merge <src> <dst>` | Merge two person rows when entity resolution split one |
| `op dedupe` | Find and merge duplicate people / promises |
| `op clean` | Sweep noise out of the graph |
| `op doctor` | Comprehensive readiness diagnostic |

For Claude Desktop / Cursor integration:

```bash
op mcp   # stdio MCP server — see docs/mcp.md
```

---

## For developers — OpenPersona as a layer

OpenPersona is **not just a product, it's a layer others can plug into**.
Three integration shapes are first-class:

### MCP (recommended for AI agents)
The MCP server (**17 tools** — 8 read + 6 write + 1 brief synth + 2
archive) gives Claude Desktop / Cursor / any MCP-compatible agent
direct access to your relationship graph + the local IM message
corpus via stdio. See [`docs/mcp.md`](docs/mcp.md).

### HTTP API (Agent Native, for everything else)

| Endpoint | Purpose |
|---|---|
| `GET  /api/changes?since=<iso>` | Poll the change feed since cursor |
| `GET  /api/messages?person_id=…` | Query the **local IM archive** — read full history without re-pulling chat.db |
| `GET  /api/schema/{entity}` | Per-entity JSON Schema for code-gen |
| `POST /api/webhooks` | Register a callback URL — delivery daemon now active |
| `POST /api/bulk/promises` | Multi-row insert with retry-safe `idempotency_key` |
| `GET  /api/openapi.json` | Full OpenAPI 3.1 spec |
| `GET  /api/docs` | Interactive Swagger UI |

### Collector Protocol (for new IM platforms)
Three record types, two functions to implement. Anything that emits
messages can plug in. See
[`docs/architecture.md`](docs/architecture.md).

---

## Why private alpha — and what flips it public

Source is currently closed because the v1 pivot is in active flight. The shape stabilises
**~v1.1** (~1 month), at which point this README's repo opens.

What's gating it:

- **Real-data validation of v1 surfaces** — pre-meeting brief / push voice / chat hotkey
  need a week of someone's real iMessage + WeChat history to tune. Closed alpha lets me
  iterate without churning a public commit log.
- **Lock screen widget** — Swift / WidgetKit work in progress. v0.4 ships the Xcode
  project generator (`op widget-xcodeproj`); signing / notarisation is the remaining gate.
- **Gmail OAuth completion** — IMAP path ships in v0.4; the OAuth refresh-token flow is
  the polish gate for full Gmail API. WhatsApp / Telegram / Outlook / Slack / Discord all
  shipped v0.4 offline-export paths.

What's already public:
- This repo (`Chen17-sq/OpenPersona`) tracks **strategy + roadmap + contact**. Everything
  in [`docs/`](docs/) is real and up to date.
- Daily commits visible on [@Chen17-sq](https://github.com/Chen17-sq)'s public contribution
  graph (private repo commits surface there).

What's gated is real-data validation, not the build itself. **Source opens when the
extractors hold up against a week of someone's real iMessage / WeChat / Gmail history.**

### Want early access?

- **Email** — [schen.aldrich@gmail.com](mailto:schen.aldrich@gmail.com)
  with one line on how you'd use it
- **LinkedIn** — [Aldrich Chen](https://www.linkedin.com/in/aldrich17siqi/)
- **WeChat** — `18574843907` (mention OpenPersona)
- **Open an issue** here describing your use case — I read every one and it shapes the
  v1.1 cut

---

## Architecture

```
surface       Lock screen widget (Swift, v0.4 xcodeproj generator) · Push (macOS native) ·
              Calendar event injection · ⌘⇧Space chat hotkey · Person page (Bauhaus, hidden
              by default) · MCP server (17 tools)

intelligence  Streaming extract (per-message LLM judgment) · Per-person tone calibration ·
              Voice-matched AI push generation · Behavioural inference (last contact, cadence,
              drift) · Implicit feedback observer (workflow interception, no buttons) ·
              Wave E contact metadata (bio / relationship / preference / tag / date / school)

persona       SQLite + per-person Markdown — atomic, lockable, git-diffable, round-trip safe ·
              raw_messages archive (every message persisted, agents query via /api/messages
              or MCP query_messages) · inner-circle tier persistence (top-20 / next-30,
              recomputed nightly from attention_score)

────── Collector Protocol ──────
collectors    iMessage · WeChat · WhatsApp (live + offline) · Telegram (live + offline) ·
              Gmail (IMAP + API stub) · Outlook (Graph + .eml) · Slack (offline) · Discord
              (offline) · Apple Health (export ZIP) · macOS Calendar (bidirectional) ·
              iCal URL (Google / iCloud / Outlook) · your own
```

We extract structured rows from conversations. We **don't** capture screens, OCR images,
or transcribe audio — those plug in via the Collector Protocol. Bring your own meeting AI
or wearable.

The MCP server exposes the graph to Claude Desktop / Cursor / any MCP-compatible agent —
ask *"what did I promise Bob?"* without leaving the chat. See [`docs/mcp.md`](docs/mcp.md)
for the tool inventory.

---

## Roadmap

| | What | Status |
|---|---|---|
| **v0** *(retired)* | 5 dashboard surfaces · Promise Grid · Calendar push/pull · Inbox edit · 15-tool MCP · auto-extract daemon | superseded by v0.1 pivot |
| **v0.1** | Pivot — lock-widget plumbing · push · calendar injection · chat hotkey backend · streaming extract · commitment-strength three-tier · AI-narrated portrait · voice-matched drafts · observer + correlator | shipped |
| **v0.2** *(now, private alpha)* | UX polish (Toast / EmptyState / `?` / optimistic UI / Person-page Svelte rewrite) · setup wizard + sources table · `op morning-push` daemon · terminal-notifier backend · **Agent Native HTTP API** (/changes, /schema, /webhooks, /bulk + idempotency) · status banner · **Master mode** (audit panel · conflict detection · staleness scorer · `op identity-merge` cross-source phone match · groups + group_members schema · conflict-resolve endpoint with master-review UI) | shipped |
| **v0.3** *(shipped)* | **raw_messages archive** (persistent local IM corpus) · `op backfill` for full-history pulls · `/api/messages` agent retrieval · `op storage` report · webhook delivery daemon (v0.2 registrations now fire) · Ollama local-mode (`op setup --only ollama`) · group-chat LLM extraction (topic / convener / lurkers / mutual-intro) | shipped |
| **v0.4** *(shipped)* | **Wave E contact metadata extractor** (bio / relationship / preference / tag / date / school / note with hallucination guard) · **inner-circle tier persistence** (top-20 inner / next-30 close from `attention_score`, daily 04:00 recompute) · **Person Page raw-messages viewer** (lazy-loaded archive timeline) · **MCP archive tools** (`query_messages` + `message_archive_stats` for external agents) · **6 new collector offline paths** (WhatsApp Export-Chat .txt / Telegram result.json / Gmail IMAP / Outlook .eml dir / Slack export / Discord export) · **schema index audit** (+6 hot-path indexes) · **Apple Health sync CLI** (`op apple-health-sync`) · **widget xcodeproj generator** (`op widget-xcodeproj`) | shipped |
| **v0.4.1** *(shipped)* | Wave E **wired into weekly tick** (Sundays 04:30 over inner-circle) · `op metadata-extract` / `op tier-recompute` / `op doctor-schema` / `op collectors-status` CLI commands · **Gmail OAuth** end-to-end (`op auth gmail` runs the loopback flow; Gmail API path live, stdlib only — no `google-auth-oauthlib` dep) · IMAP pagination + max-threads warning · `scripts/publish-pypi.sh` with pre-flight CHANGELOG check | shipped |
| **v0.4.2** *(now, shipped)* | External-audit response: **WeChat `@chatroom` group detection** (root-cause for 16k msgs all stamped chat_kind='private') · commit-pipeline **quality floor** (drops `what`<5 chars, intent-without-time, audit-curated reject set) · **noise-people regex** (catches `刚刚`/`吴+`/test-fixture leaks/embedded group hints) · **MCP write-tool security gate** (`OPENPERSONA_MCP_WRITES_REQUIRE_REVIEW=1` short-circuits direct writes; audit.log forensic trail always-on) · **dedicated daemon-writer connection** (`db.daemon_writer()` with explicit BEGIN IMMEDIATE) · **schema-versioning framework** (`schema_versions` + linear migrations) | shipped |
| **v1.0** *(+3 mo)* | Cross-device sync · plugin marketplace · Lock-screen widget signing & distribution · PyPI publish · **source opens here** | — |

[`docs/roadmap.md`](docs/roadmap.md) for the day-by-day breakdown.
[`docs/product-spec.md`](docs/product-spec.md) for the canonical product definition.

---

## Contributing

Once source opens (~v1.0), the three highest-leverage contributions will be:

1. **A new collector** — implement [`Collector Protocol`](docs/architecture.md) for any
   IM / email / chat platform you live in. Three record types, two functions, real test
   coverage already in place. v0.4 ships offline-export paths for WhatsApp / Telegram /
   Gmail / Outlook / Slack / Discord — extending these to live API mode (e.g. Gmail
   OAuth, Slack Web API) is a great next step.
2. **An agent integration** — OpenPersona ships an Agent Native HTTP API
   (`/api/changes` polling, `/api/schema` introspection, `/api/webhooks` push,
   `/api/bulk` retry-safe writes). Build a CLI / browser extension / mobile app on
   top, link it back here.
3. **AI-native voice tuning** — prompts living in the codebase. PRs improving voice
   consistency, language detection, per-person tone matching, or commitment-strength
   classification welcomed.

If you maintain a context-aware product (meeting AI, wearable, custom internal tool) and
want to feed OpenPersona's graph: open an issue, we'll co-design.

## License

MIT (applies once source opens).
