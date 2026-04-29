# OpenPersona — Technical Roadmap

The pacing layer. Cross-links: [product-spec.md](product-spec.md) for
the *what*, [architecture.md](architecture.md) for the *how*.

> Updated 2026-04-28 — v0 split into **Milestone A (CLI alpha)** +
> **Milestone B (public launch)**. W1 + most of W2 came in ahead of
> schedule; A de-risks B by getting real-user feedback before any UI
> work starts.

---

## v0 — two milestones, ~4 weeks · the wedge

```
4/28 today  ──────────────────────────────────────────►
       │             │                                   │
       │ +5 days     │ +3 weeks                          │
       ▼             ▼                                   ▼
   Milestone A   Milestone B                        v1 starts
   CLI alpha    public v0 launch
   ~5/3 (Sun)   ~5/19–5/26
```

**Why split**: the original W1–W4 monolith assumed UI was on the
critical path. It isn't — the data layer (extractors + SQLite +
Markdown) is already shippable as a CLI product to a small dogfood
circle. Shipping that 5 days from now buys two things: (1) real-user
feedback to shape Milestone B's UI, (2) a handful of testimonials /
screencasts to lead the public launch with.

---

## ✅ W1 (the brain) — done

| Day | Deliverable | Commit |
|-----|-------------|--------|
| Day 1 | bootstrap (pyproject, paths, schema, prompts, smoke tests) | `d88a310` |
| Day 2-3 | commitment extractor end-to-end on fixture (mock LLM) | `e1558f2` |
| Day 4 | real LLM smoke (DeepSeek): 3/3 precision, 3/4 recall, Chinese verbatim | `9928208` |
| Day 5 | facts extractor (mirror commitment extractor pattern) | shipped |

W1 acceptance criteria met: ≥ 30 promises + ≥ 5 facts on real data ✅, precision ≥ 90 % ✅.

## ✅ W2 (the asset) — mostly done, ahead of schedule

| Day | Deliverable | Commit |
|-----|-------------|--------|
| Day 1-2 | `store.db` connection layer (atomic write + per-path lock — vendored from OpenChronicle PRs #11/#12) | `8d5de1c` |
| Day 2-3 | CRUD for people / promises / facts / sources | `8d5de1c` |
| Day 4 | `markdown_sync.py` — SQLite → per-person markdown (one-way) | `f58f4c3` |
| Day 5 | `entity_resolver.py` — basic display_name match | shipped (basic) |

53 / 53 tests green ✅. Pending W2 work rolls into Milestone B (B1):

- Markdown → SQLite reverse parser (round-trip)
- Reminders engine triggers + `compute_next_fire()`
- Events table CRUD
- entity_resolver hardening (cross-app handle merge)

---

## 🟢 Milestone A — CLI alpha (5 days · 4/29 → 5/3)

**Goal**: 5–10 dogfood users running OpenPersona daily on real
iMessage + WeChat data. Markdown / CLI is the surface — no UI yet.

**What "done" looks like**: a 5-minute screencast of
`openpersona init && openpersona extract --since "7d ago" && openpersona sync`
producing per-person Markdown with promises + facts on the user's own
machine.

| Day | Date | Deliverable | Notes |
|-----|------|-------------|-------|
| **A1** | Wed 4/29 | iMessage collector — wrap [`niftycode/imessage_reader`](https://github.com/niftycode/imessage_reader) (Python, MIT, pip-installable) | Add `since_date` SQL filter; patch attributedBody plist parse for iOS 16+ if upstream's coverage is thin |
| **A2** | Thu 4/30 | iMessage end-to-end on real `chat.db` | Full Disk Access UX + helpful error message; smoke against last 7 days |
| **A3** | Fri 5/1 | wechat-cli live read (replace fixture path) | Subprocess wrapper exists (`collectors/wechat.py`); wire `since` arg, parse JSON, persist |
| **A4** | Sat 5/2 | `docs/quickstart.md` + `openpersona init` polish | One-page getting-started; explicit FDA / Keychain / LLM-key flow |
| **A5** | Sun 5/3 | screencast + dogfood blast | 5-min screencast → wechat-cli group + 5 personal DMs; collect feedback in private GitHub Discussion |

**Milestone A acceptance criteria** (gate to B):

- 3+ non-self users (besides me) run `extract` end-to-end on their own
  machine without manual code changes within 7 days of A5
- Zero crashes from real-data ingestion; all errors land in a recoverable
  state with actionable error messages
- A user (any one) reports a real promise the system caught that they'd
  forgotten — the existence proof for the wedge

**No-go list for Milestone A** (defer to B, do not feature-creep):

- Any UI / FastAPI server
- Calendar write
- Inbox / accept-dismiss flow
- Reminders engine / notifications
- MCP server
- WhatsApp / Telegram / Outlook collectors

---

## 🔴 Milestone B — public v0 launch (~15 working days · 5/4 → 5/19–5/26)

**Goal**: README's promised flow — iMessage + WeChat → Promise Grid →
macOS Calendar — with a UI that screenshots well, ready for HN /
Twitter (English + Chinese) / Zhihu long-form launch.

Milestone A's dogfood feedback drives which UI surfaces are actually
worth building first. We re-prioritize B2 based on what users ask for
in the CLI period.

### B1 · Persona graph completion (~2 days)

| Day | Deliverable |
|-----|-------------|
| B1.1 | Markdown → SQLite reverse parser (round-trip test: `sync_all() → parse_md() → SQL == SQL`) |
| B1.2 | Reminders engine: `compute_next_fire()` for `absolute` + `recurring_yearly` (other 3 trigger kinds → v1) |

### B2 · UI surface (~6 days)

| Day | Deliverable |
|-----|-------------|
| B2.1 | SvelteKit + Tailwind + shadcn-svelte skeleton at `web/` |
| B2.2 | FastAPI `/api/persona/me` + `/api/promises` + `/api/people/{id}` |
| B2.3-4 | **Promise Grid** (the killer surface) — People × Time matrix, color states (red=overdue, yellow=soon, green=on track) |
| B2.5 | **Persona Card** — facts / dates / reminders / promises / interactions |
| B2.6 | **Persona Self-View** — first screen, your-own digest |

### B3 · Calendar loop (~4 days)

| Day | Deliverable |
|-----|-------------|
| B3.1 | EventKit via pyobjc — write Promise → macOS Calendar event (no Swift needed) |
| B3.2 | Inbox UI: low-confidence promises + accept/dismiss/edit flow |
| B3.3 | macOS notification scheduler (pyobjc UNUserNotificationCenter) |
| B3.4 | install script + dogfood-fix-loop |

### B4 · Launch (~3 days)

| Day | Deliverable |
|-----|-------------|
| B4.1 | demo screencast (refreshed from A5) + README hero update + screenshot of Promise Grid |
| B4.2 | tag `v0.1.0`; flip repo to **public** |
| B4.3 | post Show HN / Zhihu long-form / Twitter (English + Chinese); respond to comments all day |

**Milestone B acceptance criteria**: end-to-end live: real iMessage + WeChat →
Grid → user clicks ✅ → macOS Calendar shows event → notification
fires next day. Stretch: HN front page or 100+ stars within launch week.

---

## v1 — +2 months · the agent layer

| Block | Source / Reason |
|-------|-----------------|
| **WhatsApp collector** | Wrap [`steipete/wacrawl`](https://github.com/steipete/wacrawl) (Go binary, MIT, JSON output). Reads `~/Library/Group Containers/group.net.whatsapp.WhatsApp.shared/`. Subprocess pattern same as wechat-cli — ~1 day integration. |
| **Telegram collector** | telethon (Python) with local cache mode, or read Telegram Desktop's `tdata/` if exposed |
| **Outlook collector** | Microsoft Graph API (mailbox sync) or local OLM file parse |
| **MCP server** (8 tools: `who_did_i_meet_with`, `what_did_i_promise`, `whats_overdue`, etc.) | Claude Desktop / Cursor / any MCP client gets your Persona |
| **Event-anchored promises** | "before the GLV quarterly" → links to events.id; cancel/move propagation |
| **expectation_extractor** | Inferred promises (silence / cadence / unanswered question detection) → Inbox |
| **Auto-extraction loop** | tick every 5 min on new messages; not manual `extract` |

Launch checkpoint: HN second push *"now your Persona works in Claude Desktop"*.

---

## v2 — +5 months · the ecosystem

| Block | Reason |
|-------|--------|
| **Apple Health collector** | Self facts auto (weight / sleep / workouts) — first non-IM collector |
| **Notion / Linear bidirectional sync** | Promises ↔ tasks; events ↔ docs |
| **Relationship graph visualization** | the "people network" the deal-flow user actually wants |
| **Hosted sync layer (opt-in)** | for users with multiple Macs; never default-on |
| **Plugin marketplace for collectors** | the long tail (Slack, Discord, Twitter DMs, custom internal tools) |

---

## Cross-cutting tracks

These run alongside the milestone cadence rather than in any single
block.

| Track | Cadence |
|-------|---------|
| Prompt-quality regression suite | every commit that touches `extractors/prompts/*` |
| Privacy audit | end of each milestone; reviews collector outputs vs schema |
| Distribution narrative | refresh ahead of each milestone |
| Documentation | every new feature ships with doc update in same PR |
| Test coverage | invariant: every PR keeps `pytest` green and `ruff` clean |

---

## Risk register

| Risk | Mitigation |
|------|------------|
| **iMessage attributedBody format change (iOS 27?)** | Depend on `niftycode/imessage_reader`; if breaks, fork + patch typedstream parser inline |
| WeChat encryption format changes | Depend on wechat-cli; we maintain that adapter |
| WhatsApp Group Container path moves | Depend on `wacrawl`; subprocess boundary insulates us |
| LLM cost spikes for heavy users | Batch extraction; per-call timeout + cap (shipped W1 Day 4) |
| Apple revokes EventKit access | Fall back to writing `.ics` files the user imports manually |
| User accidentally puts secrets in fixture | `.gitignore` covers `data/`; fixtures must be hand-curated |
| Maintainer-of-one bottleneck | Doc-first culture; every decision in `docs/` so a contributor can land their first PR without asking |
| **A→B drag**: Milestone A feedback wants UI changes that break B's plan | A's gate is "3+ users running daily," not "user-driven UI redesign." Treat A feedback as informing B2's *order*, not its *shape*. |
