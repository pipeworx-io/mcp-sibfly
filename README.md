# mcp-sibfly

SibFly — satellite-measured ground motion (subsidence / uplift) for any US address.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/sibfly_ground_motion \
  -H 'Content-Type: application/json' \
  -d '{"address":"1600 Pennsylvania Avenue NW, Washington, DC 20500"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/sibfly_ground_motion`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

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

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-sibfly
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

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
