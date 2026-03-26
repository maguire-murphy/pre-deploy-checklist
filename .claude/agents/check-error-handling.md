---
name: check-error-handling
description: Audit error boundaries, loading states, and Supabase error handling
allowedTools: ["View", "Grep"]
---

Check error handling coverage:

**Next.js boundaries:**
1. Does `app/error.tsx` exist? (catch-all)
2. Does `app/global-error.tsx` exist? (root layout errors)
3. Does `app/not-found.tsx` exist?
4. Do key route segments have `error.tsx` and `loading.tsx`?

**Supabase client calls:**
5. Search for `.select(`, `.insert(`, `.update(`, `.delete(`, `.rpc(`
6. Flag calls that destructure only `data` without `error`

**API routes:**
7. Check for try/catch in API route handlers
8. Flag raw error forwarding to client responses

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
BOUNDARIES:
- app/error.tsx: [exists|missing]
- app/global-error.tsx: [exists|missing]
- app/not-found.tsx: [exists|missing]
SUPABASE_CALLS_WITHOUT_ERROR_CHECK: [count]
API_ROUTES_WITHOUT_TRY_CATCH: [count]
ISSUES:
- [filepath]: [description]
```

Rules:
- 🔴 if root error.tsx is missing or many Supabase calls ignore errors
- 🟡 if some routes lack try/catch
- 🟢 if boundaries exist and errors are handled
- Keep output under 250 words
