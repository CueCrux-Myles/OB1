# VaultCrux Complement Mode

Run OB1 and VaultCrux MCP servers side-by-side. Your AI client (Claude Code, Claude Desktop, Cursor) sees both tool sets — OB1 for quick thought capture, VaultCrux for deep semantic retrieval, trust routing, and knowledge management.

## Prerequisites

- A running OB1 instance (Supabase or self-hosted) with your `MCP_ACCESS_KEY`
- A running VaultCrux instance with Memory Core MCP enabled
- Claude Code, Claude Desktop, or any MCP-compatible client

## Setup

### Claude Code

Add both servers to your Claude Code MCP settings (`~/.claude/settings.json` or project `.claude/settings.json`):

```json
{
  "mcpServers": {
    "ob1": {
      "type": "url",
      "url": "https://YOUR-PROJECT.supabase.co/functions/v1/ob1?key=YOUR_MCP_ACCESS_KEY"
    },
    "vaultcrux-memory-core": {
      "command": "npx",
      "args": ["tsx", "apps/memory-core-mcp/src/server.ts"],
      "cwd": "/path/to/VaultCrux",
      "env": {
        "VAULTCRUX_MEMORY_CORE_API_BASE": "http://localhost:14333",
        "MCP_STDIO_API_KEY": "YOUR_VAULTCRUX_API_KEY",
        "MCP_STDIO_TENANT_ID": "YOUR_TENANT_ID"
      }
    }
  }
}
```

### Claude Desktop

1. Open **Settings > Connectors > Add custom connector**
2. Add OB1: paste your OB1 MCP endpoint URL with `?key=` parameter
3. Add VaultCrux Memory Core: use the HTTP endpoint (default port 8093)

## How It Works

With both servers connected, your AI client has access to:

| OB1 Tools | VaultCrux Memory Core Tools |
|-----------|---------------------------|
| `search_thoughts` — semantic vector search | `query_memory` — multi-lane retrieval with trust scoring |
| `list_thoughts` — filtered listing | `list_topics` — topic groups with freshness metadata |
| `thought_stats` — aggregate stats | `get_versioned_snapshot` — point-in-time knowledge state |
| `capture_thought` — quick capture | `check_claim` — verify claims against stored knowledge |

The AI naturally routes to the right tool based on your request. Quick captures go to OB1; deep queries go to VaultCrux.

## Optional: Unified Proxy

If you prefer a single MCP connection, VaultCrux can proxy OB1's tools. Set these environment variables on your VaultCrux Memory Core MCP server:

```bash
OB1_URL=https://YOUR-PROJECT.supabase.co/functions/v1/ob1
OB1_KEY=YOUR_MCP_ACCESS_KEY
```

This adds `ob1_search_thoughts`, `ob1_list_thoughts`, `ob1_thought_stats`, and `ob1_capture_thought` alongside VaultCrux's native tools.

## Migration

If you later want to migrate OB1 data into VaultCrux, see the [VaultCrux Migration integration](../../integrations/vaultcrux-migration/README.md).
