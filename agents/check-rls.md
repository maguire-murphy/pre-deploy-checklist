---
name: check-rls
description: Audit Row-Level Security policies on all Supabase tables
allowedTools: ["mcp_supabase", "Grep"]
---

Check RLS on every table in the public schema using Supabase MCP.

Also grep client-side code (`app/`, `components/`, `public/`) for `service_role` references — this key bypasses RLS and must never appear in browser-accessible code.

For each table:
1. Is RLS enabled?
2. Are there policies? How many?
3. Do SELECT policies filter by `auth.uid()`?
4. Any `USING (true)` policies?
5. Do UPDATE/DELETE/INSERT policies exist? (tables with permissive SELECT but no restrictive write policies are a red flag)
6. Are write policies properly scoped with `auth.uid()` checks?

Return ONLY a summary in this exact format:

```
STATUS: 🟢|🟡|🔴
TABLES_CHECKED: [number]
ISSUES:
- [table]: [issue description]
- [table]: [issue description]
FIX_SQL:
- [SQL statement]
```

Rules:
- 🔴 if any table has RLS disabled, no policies, missing write policies, or `service_role` found in client code
- 🟡 if policies exist but have broad access (`USING (true)`) or SELECT-only without write policies
- 🟢 if all tables have proper RLS with uid filtering on both read and write operations
- Keep output under 300 words
