# The Promise Model: Two Kinds × Three Anchors

A `Promise` in OpenPersona has two orthogonal dimensions beyond the
obvious `(committer, committee, what)` triple:

1. **Kind** — *how the promise came to exist*: stated explicitly, or
   inferred from patterns.
2. **Anchor** — *when the promise is due*, with three precedence
   levels: event > time > open.

## Two kinds — by origin

OpenPersona models commitments along **one axis** that's easy to miss:
not just *what* was promised, but *how* the promise came to exist.

| Kind | Origin | Example | When implemented |
|------|--------|---------|------------------|
| **`explicit`** | Stated verbatim in a message | "I'll send you the deal flow notes tonight"<br/>"deck by Friday" | **v0** — the `commitment_extractor` |
| **`inferred`** | Derived from patterns, silence, cadence — never literally stated | "Bob's question is 3 days unanswered"<br/>"Sara cadence — overdue for a check-in"<br/>"last meeting said 'let's catch up next month' — month is here" | **v1** — the `expectation_extractor` |

Both live in the same `promises` table. The `kind` column distinguishes
them. The downstream surface (Promise Grid) renders them with the same
colors but different visual weight (explicit = solid, inferred = dashed).

## Why one table, not two

Conceptually, a user sees **"things I owe"** as one mental category. The
distinction between "I said I'd do this" and "the system thinks I should
do this" is a **provenance** detail, not a kind difference. Splitting
into two tables would force every consumer (Promise Grid, MCP `whats_overdue()`,
the reducer) to double-query. Instead: one table, one query, an optional
filter on `kind`.

## How v1's `expectation_extractor` works (locked design)

A two-stage pipeline:

```
Stage 1 — prefilter (cheap, deterministic, runs every tick):
  - Unanswered-question detector: thread has a "?" from the other
    party, no reply from `me` in > N hours (N = relationship-tier
    dependent, e.g. close friends 24 h, work contacts 48 h).
  - Cadence-violation detector: a recurring_interval reminder that's
    overdue, OR a relationship's median contact gap is exceeded.
  - Stalled-thread detector: a thread that contained explicit
    promises, all resolved, but the conversation died before the
    natural follow-up.
  - Calendar-followthrough detector: a meeting happened, action items
    were mentioned in chat, none have been ticked off.

Stage 2 — LLM-on-pattern (only for prefilter hits, ~5-20/day):
  - Input: the matched pattern + the last 30 days of messages with
    that person, condensed.
  - Question to model: "Is this actually a real expectation, or is
    it noise (small talk, no longer relevant, already handled via
    another channel)?"
  - Output: `inferred` ExtractedPromise with confidence < 0.7 by
    construction (these are guesses, not statements).
```

## User feedback loop

`inferred` promises **never auto-publish**. They land in the **Inbox**:

```
┌─────────────────────────────────────────────────────┐
│ The system thinks you should:                       │
│  ⚪ Reply to Bob's 4-25 question (silent 3 days)    │
│  [✓ Add to Grid]   [✗ Not needed]   [✏️ Edit]       │
└─────────────────────────────────────────────────────┘
```

A `✗` dismissal is a **negative training signal** — it's logged with
the pattern that triggered the inference. The next time the same
pattern fires for this person, the prefilter weights it lower. After
3 dismissals on the same pattern × person, that pattern is
permanently disabled for that person (user can re-enable in
settings).

Accepting (`✓`) flips `kind = 'explicit'` and persists like a normal
promise. The history of "this was originally inferred" is kept in the
audit log but doesn't surface in the UI.

## Sharpness invariant — does this fit?

Yes. The unit of meaning is still `(person × time × fact_or_action)`.
We're only enriching the *origin* of the action, not adding a new
dimension. Schema change is one column.

If a future request looked like:

> "Add a new kind: `negotiating` for promises that are mid-discussion
> but not fully agreed."

That would also fit — same column, new value. **No new tables, no new
extractor primitives, no new UI surface.** The Inbox handles "draft"
states naturally.

## What does NOT belong even with this expansion

- ❌ **Goals / OKRs without a promise shape** — those are facts (e.g.
  `health:run_freq = 3/week`), not promises.
- ❌ **Calendar events themselves** — those are events, not promises.
- ❌ **Reminders to take medication** — that's a `reminders` row with
  a `recurring_interval` trigger; no commitment surface attached.

The `kind` column extends Promise's expressiveness without leaking it
into adjacent primitives.

---

## Three anchors — by deadline shape

A promise's "when" is a layered concept. Time is the coarsest signal;
event is richer; some promises are open with no deadline at all yet.
The schema records them so that the UI can render the most meaningful
one available.

| Anchor | `when_event_id` | `when_iso` | Example | Implemented |
|--------|-----------------|------------|---------|-------------|
| **event** | ✅ FK events.id | usually NULL | "deck before the GLV quarterly"<br/>"ask about his wife next time I see Bob"<br/>"sync with me after Faye finishes the review" | **v1** — needs event collector + linker |
| **time** | NULL | ✅ ISO 8601 | "by Friday 5pm"<br/>"tonight at 9" | **v0** — current `commitment_extractor` |
| **open** | NULL | NULL | "soon"<br/>"in a bit" | v0 (confidence usually < 0.7) |

`when_phrase` is always retained verbatim, regardless of which anchor
resolved.

### Why event-anchored matters

A bare timestamp loses the *why*. "Friday 5pm" might be Friday because
of a meeting, because of someone's flight, because of a quarterly close.
Anchoring to the event keeps the meaning. Concrete benefits:

1. **UI clarity** — Promise Grid renders "before the GLV quarterly"
   instead of "2026-05-01T17:00", which the user actually thinks in
   terms of.
2. **Event-cancel propagation** — if the GLV meeting moves, every
   promise anchored to it shifts automatically (no orphaned deadlines).
3. **Better natural-language queries** — "what did I promise for the
   GLV meeting?" works as a structured query, not a fuzzy text search.
4. **Post-meeting follow-up** — when an event ends, all `after_event`
   reminders attached to it fire; promises anchored to it switch to
   `pending` if not yet `done`.

### v0's choice: time-anchored only, schema ready for events

v0's `commitment_extractor` only emits `when_iso`. The schema
already has `when_event_id` so v1's event-linker can write to it
without migration. v0 UI treats event-anchored promises (when they
arrive in v1) the same as time-anchored, just with a different render
label.

### Cancel / move semantics (v1)

When an event is cancelled or rescheduled:

- Cancelled: anchored promises become `status='unsure'` and land in the
  Inbox: "GLV quarterly was cancelled — are these 3 promises still in play?"
- Rescheduled: anchored promises auto-update their effective deadline
  to the new event time. No user action needed.

Time-anchored promises don't have this property — they need manual
re-scheduling. This is why event-anchoring is preferred when available.

---

## Putting it together: the full Promise schema

```python
Promise = (
    # ── identity ────────────────────────────────────────────────
    committer: Person,           # 'me' or another person
    committee: Person,           # 'me' or another person
    what:      action,           # the verb phrase

    # ── anchor (event > time > open) ────────────────────────────
    when_event_id: Event | None, # v1 — event-anchored
    when_iso:      datetime | None, # v0 — time-anchored
    when_phrase:   str,          # verbatim, always retained

    # ── kind (explicit vs inferred) ─────────────────────────────
    kind:       'explicit' | 'inferred',  # v0 only emits explicit

    # ── lifecycle ───────────────────────────────────────────────
    status:     'pending' | 'done' | 'overdue' | 'cancelled' | 'unsure',
    confidence: float,           # 0..1; below 0.5 dropped

    # ── provenance ──────────────────────────────────────────────
    source_id:  Source | None,   # the message this came from
)
```

Every column has a clear product reason. None is decoration.
