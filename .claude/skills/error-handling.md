---
name: error-handling
description: Enforce error handling patterns in React components and API routes
globs: ["**/*.tsx", "**/api/**"]
---

When writing components or API routes:
1. Always destructure both `data` and `error` from Supabase calls
2. Never expose raw errors to the UI — use user-friendly messages
3. Wrap API route handlers in try/catch, log the error, return a clean 500
4. Use `error.tsx` and `loading.tsx` at route segments that fetch data
5. Every async operation needs error handling — no unhandled promises
