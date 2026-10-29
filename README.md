# @pipeworx/kosis-kr

Korean national statistics (KOSIS) — browse, search and pull time series from
Statistics Korea's official statistical database of ~100,000 tables covering
population, prices, employment, trade and industry.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `kosis_stat_list({ parentListId, vwCd, _apiKey })` — walk the table directory one level at a time.
- `kosis_search({ searchNm, orgId, startCount, resultCount, _apiKey })` — find tables by keyword; returns the `orgId` + `tblId` pair the data tool needs.
- `kosis_stat_data({ orgId, tblId, itmId, objL1, objL2, prdSe, newEstPrdCnt, startPrdDe, endPrdDe, _apiKey })` — observations from one table.

## Auth

Platform key (`PLATFORM_KOSIS_KEY`) with BYO override via `?_apiKey=`.
Free registration at <https://kosis.kr/openapi/> — sign up, request OpenAPI
access per service, key issued immediately (no manual approval step).

## Data sources

- <https://kosis.kr/openapi/statisticsList.do> — table directory.
- <https://kosis.kr/openapi/statisticsSearch.do> — keyword search over tables.
- <https://kosis.kr/openapi/Param/statisticsParameterData.do> — observations.

## Traps

- **KOSIS reports an invalid or missing key as HTTP 200** with a body of
  `{"err":"11","errMsg":"유효하지않은 인증KEY입니다."}`. Every status-code
  check reads that as success. This pack inspects `err` on a 200 and raises.
- **`prdSe` must match what the table actually publishes.** Asking a monthly
  table for `"Y"` returns zero observations with no error.
- Korean-language search terms match far more tables than English ones.
- The success payload is **unverified** — no fleet key exists. Field names come
  from the published parameter guide; every tool also returns `raw`, so a
  caller with a real key gets the untouched payload if a name here is stale.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "kosis-kr": {
      "url": "https://gateway.pipeworx.io/kosis-kr/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/kosis-kr/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/kosis_stat_list`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-kosis-kr` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/kosis-kr/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "kosis-kr": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-kosis-kr"]
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
ask_pipeworx({ question: "your question about Kosis Kr data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
