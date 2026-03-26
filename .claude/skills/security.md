---
name: security
description: Enforce security patterns across the codebase
globs: ["**/*.ts", "**/*.tsx"]
---

When writing any code:
1. Never use `Access-Control-Allow-Origin: *` on authenticated endpoints
2. Never prefix sensitive keys with `NEXT_PUBLIC_`
3. Use the `anon` key client-side, `service_role` only server-side
4. Apply rate limiting to public-facing endpoints
5. Set security headers: X-Content-Type-Options, X-Frame-Options, Referrer-Policy
6. Verify `.env.local` is in `.gitignore`
