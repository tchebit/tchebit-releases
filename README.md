# tchebit-releases

Public release binaries for Tchê Bit client MCP servers.

## tchebit-mcp (client edition)

> Renamed 2026-07-16: `tchebit-news-mcp` → **`tchebit-mcp`** (binary
> `tchebit-news-mcp-client` → `tchebit-mcp-client`). Old releases (v0.1.0–v0.1.2)
> still carry the old binary names — unaffected, historical only. New tags
> publish under the new names.

Semantic search over Tchê Bit news, newsletters, and premium editions, plus
live market data and (Premium/Pro) perps agent tools — embeds and caches
locally, no server-side search per query. Download the binary for your
platform from the [Releases](../../releases) page.

### 1. Get an MCP key

Every tier — including **Free** — needs a Tchê Bit account. Connect your
wallet at [tchebit.com](https://tchebit.com), click your wallet address
(top right) to open the dropdown, go to **Account**, and create a key under
**MCP Keys**. Every logged-in account gets a Free-tier key by default; no
separate signup is required to try it.

### 2. Download and verify

Grab the binary for your platform from the
[latest release](../../releases/latest), then verify it against
`SHA256SUMS.txt` in the same release:

```bash
sha256sum -c --ignore-missing SHA256SUMS.txt
chmod +x tchebit-mcp-client-<platform>
```

### 3. Configure your environment

```
TCHEBIT_API_URL   default https://newsletter-api.tchebit.com
TCHEBIT_API_KEY   required — your MCP key
```

### 4. Install in Claude

**Claude Desktop** — add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "tchebit": {
      "command": "/absolute/path/to/tchebit-mcp-client",
      "env": {
        "TCHEBIT_API_KEY": "<your MCP key>"
      }
    }
  }
}
```

(No `args` needed — the binary serves over stdio by default when run with no
subcommand; `serve` and `sync` are the only subcommands, and `serve` is what
runs automatically.)

**Claude Code**:

```bash
export TCHEBIT_API_KEY=<your MCP key>
claude mcp add tchebit -- /absolute/path/to/tchebit-mcp-client
```

Using Claude Code? [`SKILLS/tchebit-mcp-setup`](SKILLS/tchebit-mcp-setup) is a
Claude Code skill that walks you through all four steps above — copy it into
your project's `.claude/skills/` and ask Claude to set up the Tchê Bit MCP.

### Toolsets — enabling a subset of tools

Tools are grouped into three families: `news`, `market`, `perps`. By default
the server exposes all three (subject to your tier — see below). To limit
which families a client loads, either pass `--toolsets` on the binary or set
the equivalent env var:

```bash
tchebit-mcp-client --toolsets news,market
# or
TCHEBIT_MCP_TOOLSETS=perps ./tchebit-mcp-client
```

This is useful for agents that should only ever see one family — e.g. a
trading agent scoped to `perps` can't accidentally call a news tool, and
vice versa. The filter applies to both tool listing and tool calls: a tool
outside the enabled set is neither listed nor callable.

### Hard limits per tier

`news`/`market` sync and `perps` calls are on **separate rate budgets** —
a perps agent polling frequently can't starve your news sync, and vice
versa. `news` search itself runs offline against the local cache (embeds
your query, cosine-search, no network call); `perps` tools recompute from
freshly fetched data on every call, so each call does hit the network.

| Tier    | `/feed/semantic-sync` requests | Premium editions (The Setup / Money Flow / Scorecard) | `market` tools | `perps` tools | `perps` requests |
|---------|--------------------------------|--------------------------------------------------------|-----------------|----------------|--------------------|
| Free    | 30 / hour                       | Not included                                            | Asset price only | Not included    | — |
| Premium | 120 / hour                      | Included                                                | Full             | scan / oracle / unwind | 120 / hour |
| Pro     | 240 / hour                      | Included                                                | Full             | all five (adds depth + position) | 600 / hour |

The client caches everything it syncs, so the `/feed/semantic-sync` ceiling
only matters for how often it refreshes — a normal Claude session doing many
searches against an already-synced cache makes zero additional requests.
`perps` tools have no local cache (every call fetches live data), so that
budget is sized for active polling instead.

### Available tools

**`news`** — `news_status`, `semantic_search`, `search_news`,
`search_newsletters`, `search_premium`, `get_content`.

**`market`** (Free: asset price only; Premium/Pro: full) —
`get_asset_price` (free), `get_funding_table` (Premium/Pro),
`get_macro_read` (Premium/Pro).

**`perps`** (Premium/Pro only, agent support tools — all figures computed
client-side from live Hyperliquid + Pyth data with freshness/staleness
guards; never returns stale data silently) —
- `perps_scan_funding` — funding-rate opportunity scan, net-APY after fees (Premium+)
- `perps_oracle_price` — Pyth oracle vs. Hyperliquid mark, dislocation check (Premium+)
- `perps_unwind_signal` — funding-decay/exit signal over recent epochs (Premium+)
- `perps_check_depth` — order-book slippage/max-size check (Pro only)
- `perps_position_health` — margin/liquidation-distance check (Pro only)
