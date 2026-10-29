# mcp-sibfly

SibFly — satellite-measured ground motion (subsidence / uplift) for any US address.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

| Tool | Description |
|------|-------------|
| `sibfly_ground_motion` | Satellite-measured GROUND MOTION for a US address — is the land under a property SINKING or rising (subsidence / uplift / heave), in mm/year and in/year, from NASA Sentinel-1 InSAR. Use for property foundation risk, land subsidence, sinkhole/settlement concern, real-estate due diligence, or "is this address sinking?". Returns vertical velocity (mm/yr + in/yr) with uncertainty, seasonal amplitude, total motion, a plain-language assessment, confidence, and provenance. With a SibFly key (_apiKey) this is a full paid report ($0.40 from your credits); without a key it returns a FREE teaser (coverage + confidence + would_cost_usd). Out-of-coverage addresses return coverage:false and are never billed. |
| `sibfly_coverage` | FREE preflight: does SibFly have satellite ground-motion coverage for this US address, and what would a full report cost? No key required. Returns coverage (true/false), confidence, and would_cost_usd. Use this before sibfly_ground_motion to check a property is covered without spending credits. |
| `sibfly_timeseries` | Historical ground-motion time series for a US address — the per-date satellite measurements of vertical displacement over time (how the land has moved, month by month). Use to see the TREND of subsidence/uplift, not just the current velocity. series_type "measured" = real per-pass observations (premium frames); "modeled" = fitted trend on real acquisition dates. Needs a SibFly key for the full series ($0.40); keyless returns a coverage teaser. |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "sibfly": {
      "url": "https://gateway.pipeworx.io/sibfly/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/sibfly/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/sibfly_ground_motion \
  -H 'Content-Type: application/json' \
  -d '{"address":"1600 Pennsylvania Avenue NW, Washington, DC 20500"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/sibfly_ground_motion`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-sibfly` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/sibfly/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "sibfly": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-sibfly"]
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
ask_pipeworx({ question: "your question about Sibfly data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
