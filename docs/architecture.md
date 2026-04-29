# Architecture

## OpenPersona is a layer, not an app

OpenPersona owns the **Persona graph + extraction + surface** — what we
build is the layer that turns raw messages and events into a queryable
graph of "what people promised whom, when, and who they are."

We **do not build collectors for visual / multimodal sources**. Instead
we publish a `Collector Protocol` and let other products (Paperboy,
future Granola integrations, custom internal tools) plug in. OCR, VLM,
screen-context, audio transcription — those belong to specialized
collectors, not the OpenPersona core.

This is the inverse of building a vertical product: **anyone making a
context-aware product becomes a potential contributor of a collector**,
and every new collector enriches every OpenPersona user's graph.

## 4-layer stack

```
┌────────────────────────────────────────────────────┐
│  Surface (we build)                                │
│  - SvelteKit UI (localhost:7600)                   │
│  - MCP server (v1) — any agent can query           │
└────────────────────────────────────────────────────┘
                  ▲ queries
┌────────────────────────────────────────────────────┐
│  Persona Graph (we build, the core asset)          │
│  - SQLite: people / facts / promises /             │
│    events / reminders / sources                    │
│  - Markdown: per-person files                      │
│  - Bidirectional sync (atomic write + per-path lock)│
└────────────────────────────────────────────────────┘
                  ▲ writes
┌────────────────────────────────────────────────────┐
│  Extractors (we build, pure functions)             │
│  - commitment / facts / entity_resolver            │
│  - litellm wrapper (explicit timeout + retries)    │
└────────────────────────────────────────────────────┘
                  ▲ raw events (Message / Event / Source)
┌────────────────────────────────────────────────────┐
│  Collectors (we publish a protocol; anyone can     │
│              implement)                            │
│                                                    │
│  Built-in (we maintain):                           │
│    - WeChat (via wechat-cli)                       │
│    - iMessage (macOS chat.db)              [W1+]   │
│    - Calendar (EventKit, read + write)     [W4]    │
│                                                    │
│  External (anyone can implement the protocol):     │
│    - Paperboy MCP — OS activity, screen text,      │
│      mouse clicked_text, transcriptions, OCR/VLM   │
│    - Granola / Otter — meeting transcripts         │
│    - Apple Health — self facts (weight, sleep)     │
│    - your custom collector                         │
└────────────────────────────────────────────────────┘
```

## Collector Protocol (the boundary)

A collector is anything that emits a stream of three record types:

| Record | What | Required fields |
|--------|------|-----------------|
| `Source` | One traceable origin (a message, an event, a captured frame) | `id`, `type`, `external_id`, `excerpt`, `occurred_at` |
| `Message` | Conversational text — IM, email, transcription | `Source` fields + `chat_id`, `sender_handle`, `sender_display`, `text`, `is_self` |
| `Event` | A discrete occurrence (meeting, call, focus block) | `Source` fields + `participants[]`, `start_iso`, `end_iso`, `summary` |

A collector implementation is either:
1. A Python module exposing `iter_messages(since, until) -> Iterable[Message]` etc.
2. A CLI / MCP that emits the records as JSONL on stdout / a tool call.

Either is fine; OpenPersona has adapters for both. The contract is the
record schema, not the transport.

## Dependencies (one-directional)

```
collectors → extractors → store ← reminders_engine
api depends on store + reminders_engine
ui depends only on api (never touches SQLite directly)
```

## Invariants

- **`extractors` are pure functions**. No I/O. Tests use mocked LLM.
- **`me` is a normal row** in `people`. No code path special-cases self.
- **New domains = new fact key prefix**. Never new tables.
- **Markdown is source of truth for human-editable fields**. SQLite is the index.
- **Every fact / promise traces to a `source` row**.
- **Collectors don't import from extractors / store / surface**. They
  produce records and stop. This keeps the protocol independently
  versionable from internals.

## Dependencies (must stay one-directional)

```
collectors → extractors → store ← reminders_engine
api depends on store + reminders_engine
ui depends only on api (never touches SQLite directly)
```

## Invariants

- **`extractors` are pure functions**. No I/O. Tests need only mocked LLM.
- **`me` is a normal row** in `people`. No code path special-cases self.
- **New domains = new fact key prefix**. Never new tables.
- **Markdown is source of truth for human-editable fields**. SQLite is the index.
- **Every fact / promise traces to a `source` row**.
