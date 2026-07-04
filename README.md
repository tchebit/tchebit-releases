# tchebit-releases

Public release binaries for Tchê Bit client MCP servers.

## tchebit-news-mcp (client edition)

Semantic search over Tchê Bit news, newsletters, and premium newsletters —
embeds and caches locally, no server-side search per query. Download the
binary for your platform from the [Releases](../../releases) page.

### 1. Get an MCP key

Every tier — including **Free** — needs a Tchê Bit account. Connect your
wallet at [tchebit.com](https://tchebit.com), then create a key under
**Settings → MCP Keys**. Every logged-in account gets a Free-tier key by
default; no separate signup is required to try it.

### 2. Download and verify

Grab the binary for your platform from the
[latest release](../../releases/latest), then verify it against
`SHA256SUMS.txt` in the same release:

```bash
sha256sum -c --ignore-missing SHA256SUMS.txt
chmod +x tchebit-news-mcp-client-<platform>
```

### 3. Configure your environment

```
TCHEBIT_API_URL     default https://newsletter-api.tchebit.com
TCHEBIT_API_TOKEN   required — your MCP key
```

### 4. Install in Claude

**Claude Desktop** — add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "tchebit": {
      "command": "/absolute/path/to/tchebit-news-mcp-client",
      "env": {
        "TCHEBIT_API_TOKEN": "<your MCP key>"
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
export TCHEBIT_API_TOKEN=<your MCP key>
claude mcp add tchebit -- /absolute/path/to/tchebit-news-mcp-client
```

Using Claude Code? [`SKILLS/tchebit-mcp-setup`](SKILLS/tchebit-mcp-setup) is a
Claude Code skill that walks you through all four steps above — copy it into
your project's `.claude/skills/` and ask Claude to set up the Tchê Bit MCP.

### Hard limits per tier

Requests below are for `/feed/semantic-sync` — the client's local cache
refresh, not a per-search budget. Search itself runs offline against the
local cache (embeds your query, cosine-search, no network call).

| Tier    | `/feed/semantic-sync` requests | Premium editions (The Setup / Money Flow / Scorecard) |
|---------|--------------------------------|--------------------------------------------------------|
| Free    | 10 / hour                       | Not included                                            |
| Premium | 120 / hour                      | Included                                                |
| Pro     | 240 / hour                      | Included                                                |

The client caches everything it syncs, so these ceilings only matter for how
often it refreshes — a normal Claude session doing many searches against an
already-synced cache makes zero additional requests.

### Available tools

`news_status`, `semantic_search`, `search_news`, `search_newsletters`,
`search_premium`, `get_content`.
