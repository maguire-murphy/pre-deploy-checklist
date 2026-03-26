---
name: input-validation
description: Enforce input validation patterns when writing API routes or server actions
globs: ["**/api/**", "**/actions/**"]
---

When writing or editing any API route or server action:
1. Always use Zod for request body validation
2. Validate at the entry point, before business logic
3. Use `.trim()` on string inputs, `.max(N)` on all strings
4. Return 400 with `error.flatten()` on validation failure
5. Never pass unsanitized input to `.rpc()` calls
6. Wrap `JSON.parse` in try/catch
