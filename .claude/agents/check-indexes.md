---
name: check-indexes
description: Audit database index coverage against actual query patterns
allowedTools: ["mcp_supabase", "View", "Grep"]
---

Two-phase index audit:

**Phase 1 — Database (via Supabase MCP):**
- List all indexes: `SELECT tablename, indexname FROM pg_indexes WHERE schemaname = 'public';`
- List foreign keys without indexes
- Check `pg_stat_user_tables` for tables with high `seq_scan` relative to `idx_scan`

**Phase 2 — Codebase (via Grep):**
- Search for `.eq(`, `.order(`, `.filter(`, `.match(`, `.in(` in the codebase
- Extract the column names being queried

Cross-reference: flag queried columns that lack indexes.

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
INDEXES_FOUND: [number]
GAPS:
- [table].[column] — used in [.eq()/.order()/etc], no index
FIX_SQL:
- CREATE INDEX idx_[table]_[column] ON [table]([column]);
```

Rules:
- 🔴 if foreign keys lack indexes or high-traffic columns are unindexed
- 🟡 if minor gaps found
- 🟢 if all queried columns are indexed
- Keep output under 300 words
