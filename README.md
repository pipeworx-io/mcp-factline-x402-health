# @pipeworx/factline-x402-health

Health of x402 sellers and facilitators: Factline's nightly seller probes and hourly facilitator probes, proxied live per request.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `x402_facilitator_health(facilitator?)` -- all tracked x402 facilitators with current status (up/down/unknown), 7-day uptime, latency and supported networks, or one by id, name or URL. Refreshes hourly.
- `x402_seller_health(query?, tag?, network?, limit?)` -- x402 sellers ranked by 30-day payers, each with a health reading (healthy/degraded/unhealthy/insufficient_data/unprobed) from nightly probes. At most 100 per call; narrow with query, tag or network to reach the rest.
- `x402_seller_lookup(origin)` -- one seller by origin, host or URL: per-day probe history, uptime, listed payTo wallets, health reading. Use before paying a seller.

Every response carries `source`, `cite_as`, `cadence`, `data_as_of` and `warnings`.

## Auth

Keyless. Factline offered these feeds free of charge for agent use.

## Data sources

- Factline, x402 Seller Health Index -- nightly probes of sellers listed in public x402 catalogues.
- Factline, x402 Facilitator Health -- hourly probes of x402 facilitators.

Cite as "Source: Factline (factline.dev)". Cadence: sellers nightly, facilitators hourly; `data_as_of` is the latest probe, and a `warnings` entry appears when it is older than the cadence implies.

## Reading the results

- Absence is not health. `found:false` or `count:0` means unknown or unmatched, never "fine". The pack says so in the payload.
- The seller feed lists at most 100 sellers per request with no paging (about 1,000 tracked); `coverage_note` states the cut.
- `health` and `status_now` are this pack's reading of Factline's numbers (rules in `health_basis` / `status_basis`), not Factline's own verdict. A facilitator that requires an API key is probed on `/supported` only (`verify_tested: false`).
- Seller probes check that a sample endpoint answers a valid payment challenge; no payment is made and nothing is said about the quality of what the seller sells.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "factline-x402-health": {
      "url": "https://gateway.pipeworx.io/factline-x402-health/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/factline-x402-health/mcp` returns the tools in the table
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

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/x402_facilitator_health \
  -H 'Content-Type: application/json' \
  -d '{"facilitator":"payai"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/x402_facilitator_health`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "factline-x402-health": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-factline-x402-health"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-factline-x402-health
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Factline X402 Health data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
