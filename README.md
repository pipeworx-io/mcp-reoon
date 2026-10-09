# @pipeworx/reoon

Verify a single email address via Reoon Email Verifier (reoon.com), with an
explicit catch-all flag in "power" mode. Part of `enrich-waterfall`'s
`enrich_verify_email` ordered verifiers (fleet #2860).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1745+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `reoon_verify(email, mode?, _apiKey)` — status, catch-all (power mode
  only), score, disposable/role/disabled/spamtrap/free flags.

## Auth

BYO only. Pass your Reoon API key as `_apiKey` (sent as the `key` QUERY
PARAM, not a header — this is how Reoon's own docs specify it). Create a key
at https://emailverifier.reoon.com/ -> API Settings. No platform key (D4).

## Data sources

- `https://emailverifier.reoon.com/api/v1/verify` — single email verify
  (quick/power mode).

## Verification notes (fleet #2860, 2026-10-08, no vendor key held)

- **Edge-probed live with an invalid key**: HTTP 401
  `{"reason":"Invalid API key.","status":"error"}` — confirms the endpoint,
  that `key` is read as a query param, and that a bad key fails cleanly.
- **Response shape from vendor docs** (https://www.reoon.com/articles/api-documentation-of-reoon-email-verifier/,
  read 2026-10-08) — no successful live call made (no key held):
  - quick: `{ email, status: valid|invalid|disposable|spamtrap,
    is_valid_syntax, is_disposable, is_role_account, mx_accepts_mail,
    is_spamtrap, is_free_email, verification_mode: "quick" }`
  - power: `{ email, status: safe|invalid|disabled|disposable|inbox_full|
    catch_all|role_account|spamtrap|unknown, overall_score,
    is_safe_to_send, is_valid_syntax, is_disposable, is_role_account,
    is_catch_all, is_deliverable, is_disabled, is_spamtrap,
    verification_mode: "power" }`
  **Only power mode exposes `is_catch_all`** — this pack defaults `mode` to
  `"power"` so enrich_verify_email always gets the field it needs; quick
  mode reports `catch_all: null` rather than guessing.
- **Not verified (no key held):** exact scoring thresholds behind
  `overall_score`; whether `status: "unknown"` (power mode) ever coincides
  with `is_catch_all: true` or is a separate signal.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "reoon": {
      "url": "https://gateway.pipeworx.io/reoon/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/reoon/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1745+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/reoon_verify`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "reoon": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-reoon"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-reoon
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Reoon data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
