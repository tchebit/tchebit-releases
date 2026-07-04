---
name: tchebit-mcp-setup
description: Install and configure the Tchê Bit news MCP (tchebit-news-mcp-client) for Claude Code or Claude Desktop — downloads the right binary, verifies its checksum, and wires it up with the user's MCP key. Use when the user asks to set up, install, or configure the Tchê Bit MCP, add tchebit to Claude, search Tchê Bit news/newsletters from Claude, or mentions an MCP key from tchebit.com / Account / MCP Keys.
---

# Tchê Bit MCP setup

Installs `tchebit-news-mcp-client` — semantic search over Tchê Bit news,
newsletters, and premium editions, run locally as an MCP server over stdio.
Repo: https://github.com/tchebit/tchebit-releases

Work through these steps in order. Ask the user for anything you can't
detect yourself; don't guess at their MCP key or install location.

## 1. Confirm the user has an MCP key

Every tier — including Free — needs one. If they don't have one yet, tell
them: connect your wallet at tchebit.com, click your wallet address (top
right) to open the dropdown, go to **Account**, and create a key under
**MCP Keys**. Wait for them to paste it before continuing. Treat
the key as a secret — don't print it back in full, don't log it, don't put
it in a shell history-visible command if you can avoid it (prefer writing it
into a config file directly over `export KEY=...` on the command line).

## 2. Detect platform and download the binary

Determine OS + arch (e.g. `uname -sm`), then map to the release asset name:

| OS      | Arch    | Asset suffix           |
|---------|---------|-------------------------|
| Linux   | x86_64  | `linux-x86_64`          |
| Linux   | aarch64 | `linux-aarch64`         |
| macOS   | arm64   | `darwin-aarch64`        |
| macOS   | x86_64  | `darwin-x86_64`         |
| Windows | x86_64  | `windows-x86_64.exe`    |

Download the matching `tchebit-news-mcp-client-<suffix>` and `SHA256SUMS.txt`
from the latest release:

```bash
curl -fsSL -o tchebit-news-mcp-client \
  https://github.com/tchebit/tchebit-releases/releases/latest/download/tchebit-news-mcp-client-<suffix>
curl -fsSL -o SHA256SUMS.txt \
  https://github.com/tchebit/tchebit-releases/releases/latest/download/SHA256SUMS.txt
```

Verify the checksum before doing anything else with the binary:

```bash
sha256sum --ignore-missing -c SHA256SUMS.txt
```

If verification fails, stop and tell the user — do not proceed with an
unverified binary.

Pick an install location (ask the user, or default to
`~/.local/bin/tchebit-news-mcp-client` on Linux/macOS), move the binary
there, and `chmod +x` it.

## 3. Configure the MCP client

The binary takes no required arguments — running it with no subcommand
serves over stdio by default (`serve` and `sync` are the only two
subcommands; `serve` is what runs automatically). Don't add `serve` as an
arg — it's not something separate to install, just the default behavior.

**Claude Code** (preferred if this skill is running inside Claude Code):

```bash
claude mcp add tchebit -- <absolute-path-to-binary>
```

Then set `TCHEBIT_API_KEY` for that server — if `claude mcp add` in the
user's installed version doesn't support an inline env flag, fall back to
writing the project or user MCP config file directly with:

```json
{
  "mcpServers": {
    "tchebit": {
      "command": "<absolute-path-to-binary>",
      "env": { "TCHEBIT_API_KEY": "<their key>" }
    }
  }
}
```

**Claude Desktop**: edit `claude_desktop_config.json` (macOS:
`~/Library/Application Support/Claude/claude_desktop_config.json`; Windows:
`%APPDATA%\Claude\claude_desktop_config.json`; Linux:
`~/.config/Claude/claude_desktop_config.json`) and merge in the same
`mcpServers.tchebit` block as above. Preserve any existing `mcpServers`
entries — don't overwrite the whole file.

`TCHEBIT_API_URL` doesn't need to be set — it defaults to
`https://newsletter-api.tchebit.com` (production).

## 4. Smoke test

Restart Claude Code / Claude Desktop if required for the new MCP server to
load, then call the `news_status` tool. It should report a cache entry count
(0 is fine on first run — the cache fills in on the next `sync`). If it
instead reports an auth error, the key is wrong or wasn't picked up by the
env config — re-check step 3, not step 2.

## Hard limits (tell the user if relevant)

`/feed/semantic-sync` (the client's local-cache refresh) is rate-limited per
tier: Free 10/hour, Premium 120/hour, Pro 240/hour. This is not a per-search
limit — search runs offline against the local cache. Free tier does not sync
premium editions (The Setup / Money Flow / Scorecard); Premium and Pro do.
