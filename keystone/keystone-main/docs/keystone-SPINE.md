# Keystone Local-First Spine

## Principle

Keystone's running desktop app is the primary source of truth. The local SQLite
spine stores definitions needed across views: feature glossary, models and
presets, non-secret configuration, node library, file types, datasets, filters,
statuses, and lifecycles.

Cloud synchronization is a **mirror**, never a prerequisite for normal local
operation. Provider keys and other secret values stay in local Electron
safeStorage/local files; `secret_refs` only names a local secret reference.

## Local API

| UI / capability | Endpoint | Local source |
|---|---|---|
| Whole state snapshot | `GET /api/spine/snapshot` | `spine_*` SQLite tables |
| Feature glossary | `GET /api/spine/glossary` | `spine_registry` |
| Provider model catalog + presets | `GET /api/spine/models` | `spine_model_catalog`, `spine_model_presets` |
| Flow node library | `GET /api/spine/nodes` | `spine_node_library` |
| Global filter definitions | `GET /api/spine/filters` | `spine_global_filter_definitions` |
| Non-secret configuration | `GET/PUT /api/spine/config/{scope}/{key}` | `spine_configurations` |

## Local Tables → Supabase Mirror

| SQLite | Supabase |
|---|---|
| `spine_registry` | `keystone.registry` |
| `spine_status_definitions` | `keystone.status_definitions` |
| `spine_lifecycle_definitions` | `keystone.lifecycle_definitions` |
| `spine_file_type_definitions` | `keystone.file_type_definitions` |
| `spine_model_catalog` | `keystone.model_catalog` |
| `spine_model_presets` | `keystone.model_presets` |
| `spine_configurations` | `keystone.configurations` |
| `spine_node_library` | `keystone.node_library` |
| `spine_node_presets` | `keystone.node_presets` |
| `spine_datasets` | `keystone.datasets` |
| `spine_global_filter_definitions` | `keystone.global_filter_definitions` |

## Sync Rules

1. Seed and write local SQLite first.
2. Build a normalized snapshot through `/api/spine/snapshot`.
3. Mirror only user-approved non-secret metadata into the `keystone` Supabase schema.
4. Resolve conflicts by monotonic `updated_at`, preserving local changes when offline.
5. Do not sync API keys, OAuth tokens, or raw credential strings. Store only `secret_refs`.

## Current Seeded Definitions

- 22 provider defaults plus embedding-model entries (`spine_model_catalog`)
- Default model preset for each provider (`spine_model_presets`)
- Ten Flow Canvas node types from the actual `backend/bernie.py` library
- Six catalog/content/task datasets, including 48-hour freshness for live listings and Keepa data
- Five global filter definitions, including marketplace, brand, team, project, and freshness
- File definitions for Excel, CSV/TSV, Markdown, PDF, Word, JSON, NDJSON, and JSONL
- All features parsed from the canonical `frontend/sidebar.js` registry

## Cloud Target

Supabase project `dfvylthyfrcarucyqgru` now contains the `keystone.*` mirror
tables with RLS enabled. Backend/server credentials are required before an
application sync job writes to those tables.
