# VaultCrux Migration

Migrate your OB1 thoughts into VaultCrux for deep semantic retrieval, trust routing, freshness scoring, contradiction detection, and the full VaultCrux knowledge management stack.

## What Happens During Migration

1. VaultCrux connects to your OB1 instance (Supabase or direct Postgres)
2. Thoughts are pulled incrementally, paginated by `created_at`
3. Each thought is normalised into a VaultCrux memory record
4. Content is re-embedded using VaultCrux's embedding model (768-dim Nomic) — OB1's 1536-dim OpenAI embeddings are **not** transferred
5. Deduplication prevents re-importing the same thought on subsequent syncs
6. Subsequent syncs only fetch new thoughts (cursor-based incremental)

## Field Mapping

| OB1 | VaultCrux |
|-----|-----------|
| `id` | `sourceConversationId` |
| `content` | `content` (re-embedded at 768d) |
| `created_at` | `timestampSource` |
| `metadata.topics` | `topics[]` |
| `metadata.people` | `entities[]` |
| `metadata.type` | `metadata.ob1.type` |
| `metadata.action_items` | `metadata.ob1.action_items` |
| `metadata.dates_mentioned` | `metadata.ob1.dates_mentioned` |

All imported records have `shareability: "owner_only"` and `sourcePlatform: "ob1"`.

## Setup

### Option A: API Pull (Supabase users)

Create a VaultCrux connector via the API:

```bash
curl -X POST http://localhost:14333/v1/memory/connectors \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_VAULTCRUX_API_KEY" \
  -H "x-tenant-id: YOUR_TENANT_ID" \
  -d '{
    "provider": "ob1",
    "mode": "api_key_pull",
    "apiKey": "YOUR_OB1_MCP_ACCESS_KEY",
    "syncFrequency": "daily",
    "config": {
      "ob1Mode": "supabase",
      "ob1Url": "https://YOUR-PROJECT.supabase.co/functions/v1/ob1"
    }
  }'
```

### Option B: Direct Postgres (self-hosted users)

```bash
curl -X POST http://localhost:14333/v1/memory/connectors \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_VAULTCRUX_API_KEY" \
  -H "x-tenant-id: YOUR_TENANT_ID" \
  -d '{
    "provider": "ob1",
    "mode": "api_key_pull",
    "apiKey": "unused",
    "syncFrequency": "daily",
    "config": {
      "ob1Mode": "direct_pg",
      "ob1Url": "postgres://user:pass@host:5432/openbrain"
    }
  }'
```

### Option C: Manual Upload (JSON export)

Export your thoughts as a JSON array and upload:

```bash
curl -X POST http://localhost:14333/v1/memory/connectors/upload \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_VAULTCRUX_API_KEY" \
  -H "x-tenant-id: YOUR_TENANT_ID" \
  -d '{
    "provider": "ob1",
    "data": [
      {"id": "abc-123", "content": "My first thought", "metadata": {"topics": ["test"]}, "created_at": "2026-03-01T00:00:00Z"}
    ]
  }'
```

## Trigger Manual Sync

```bash
curl -X POST http://localhost:14333/v1/memory/connectors/CONNECTOR_ID/sync \
  -H "x-api-key: YOUR_VAULTCRUX_API_KEY" \
  -H "x-tenant-id: YOUR_TENANT_ID"
```

## Verify Migration

Compare counts:

1. OB1: use `thought_stats` tool or query `SELECT count(*) FROM thoughts`
2. VaultCrux: query memory records with `source_platform = 'ob1'`

```bash
curl "http://localhost:14333/v1/memory/topics" \
  -H "x-api-key: YOUR_VAULTCRUX_API_KEY" \
  -H "x-tenant-id: YOUR_TENANT_ID"
```

## Complement Mode

Migration is not exclusive — you can keep OB1 running alongside VaultCrux. See the [VaultCrux Complement recipe](../../recipes/vaultcrux-complement/README.md) for parallel MCP setup.
