# OpenPersona MCP Server

Expose your Persona graph (people, promises, facts, events) to any
MCP-compatible agent — Claude Desktop, Cursor, Continue, your own
custom client.

The server is read-only by design: agents can *query* your graph but
not mutate it. Mutations stay behind the FastAPI surface where you see
them in the browser, preserving the human review loop.

## The 9 tools

8 read-only tools query the graph; 1 write tool (`propose_promise`)
drops a row into the Inbox at low confidence for the user to confirm
— writes never auto-publish.

| Tool | What it does | Read/Write |
|---|---|---|
| `find_person` | Fuzzy lookup by id / display name / alias | R |
| `what_did_i_promise` | Outgoing promises (committer = me), filterable by recipient + status | R |
| `who_owes_me` | Incoming promises (committee = me), same filters | R |
| `whats_overdue` | Promises past their deadline, optionally direction-filtered | R |
| `facts_about` | Facts on one person, grouped by key prefix (bio:, tag:, …) | R |
| `recent_interactions` | Last N sources (messages / events) involving a person | R |
| `who_did_i_meet_with` | Event participants in a recent window, ordered by count | R |
| `next_meeting_with` | Next upcoming event with a specific person | R |
| `propose_promise` | Drop a new promise into the Inbox at confidence 0.5 (kind='inferred'). User accepts/dismisses/edits in the UI. | **W → Inbox** |

Inputs are flat strings/ints (no nested objects) so Claude Desktop's
tool-call UI fills them out cleanly. Outputs are JSON dicts/lists with
both `id` and `display_name` paired so the agent can re-query without
losing context.

## Wire it into Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`
(create if missing):

```json
{
  "mcpServers": {
    "openpersona": {
      "command": "openpersona",
      "args": ["mcp"]
    }
  }
}
```

If `openpersona` isn't on Claude Desktop's `PATH` (Claude Desktop
launches with a stripped environment), use the absolute path from
`which openpersona`:

```json
{
  "mcpServers": {
    "openpersona": {
      "command": "/Users/you/.local/bin/openpersona",
      "args": ["mcp"]
    }
  }
}
```

Restart Claude Desktop. The 8 tools should appear in the tool-use UI on
the next conversation start. Try:

> "What did I promise Bob? What's overdue? Who do I have a meeting with
> tomorrow?"

## Wire it into Cursor

Settings → MCP → Add new MCP server, paste the same JSON. Same restart
discipline.

## Wire it into your own client

Anything speaking MCP over stdio works:

```bash
openpersona mcp
# stdio JSON-RPC, blocks
```

Send `{"jsonrpc":"2.0","id":1,"method":"tools/list"}` to enumerate;
`{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"whats_overdue","arguments":{}}}`
to call.

## Privacy

The MCP server reads from your local SQLite (`$OPENPERSONA_ROOT/index.db`).
Zero network calls. Whatever agent you point at it sees only what its
tool calls return — and what those calls return is whatever it asks for.

If you want to whitelist particular people / promises / facts from the
agent's view, the cleanest path right now is to keep them out of the
graph entirely (use a per-source kill switch on the collector). A
finer-grained MCP-level filter is on the v2 list.

## Implementation notes

- Built on `mcp.server.fastmcp.FastMCP` — the high-level decorator API.
  Each tool is a Python function; the SDK auto-generates the JSON
  schema clients see.
- Tool tests live in `tests/test_mcp_server.py` and exercise each
  tool by invoking `mcp.call_tool(name, args)` directly — same path
  stdio JSON-RPC takes, minus the transport.
- The 8-tool budget is intentionally small. Once we see how Claude
  Desktop / Cursor users actually invoke these, low-traffic tools get
  cut and high-leverage queries (e.g. `health_facts_for_me`,
  `mention_count_in_window`) get added. See the `TODO(real-data):`
  comments in `src/openpersona/mcp/server.py`.
