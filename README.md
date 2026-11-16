# @pipeworx/apra-enforcement

Australian Prudential Regulation Authority (APRA) enforcement against banks, insurers and superannuation trustees: the court enforceable undertakings it has accepted (from 2005) and the infringement notices it has served (from 2019), with the Act and section, the conduct, the penalty and the linked media release.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2006+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `apra_enforcement_search(query?, year?, register?, limit?)` -- searches both registers, newest first. The keyword matches the entity name, the conduct and the media release title. `register` is `all`, `enforceable_undertakings` or `infringement_notices`. Each hit: action id, entity, date, Act and section, penalty, conduct summary, expiry, media release. `individuals_dropped` counts what was not returned.
- `apra_enforcement_get(id)` -- one action by its id (such as `in-2024-07-17-onepath-custodians-pty-limited`): the undertaking documents, variations and media releases. An unknown id is `found:false`.

Every response carries `source` and `data_as_of` (the newest action date).

## Auth

Keyless.

## Data sources

- Australian Prudential Regulation Authority (APRA), enforceable undertakings and infringement notices registers. Read live per call, nothing stored.

Privacy: entities only. The undertakings register has its own section for individuals (disqualified directors and trustees); that section is never read into the output, only counted in `individuals_dropped`. A row in the entity sections whose name is written like a person's ("Surname, Given") or has no company marker is dropped and counted too.

Traps worth knowing: a zero-match search means no match in APRA's two registers, not that an entity has a clean record, and ASIC or AUSTRAC action is not here. The date on an undertaking is APRA's acceptance date; an infringement notice carries its issue date. Two actions on one day for the same entity get a numeric suffix on the id.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "apra-enforcement": {
      "url": "https://gateway.pipeworx.io/apra-enforcement/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/apra-enforcement/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2006+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/apra_enforcement_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"OnePath"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/apra_enforcement_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-apra-enforcement` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/apra-enforcement/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "apra-enforcement": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-apra-enforcement"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Apra Enforcement data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
