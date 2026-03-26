---
name: logging
description: Enforce structured logging standards in server-side code
globs: ["**/api/**", "**/actions/**", "**/lib/**"]
---

When writing server-side code:
1. Use a structured logger (pino) instead of console.log
2. Log at entry, exit, and error points in API routes
3. Include request ID and user ID in log context
4. Never log passwords, tokens, secrets, API keys, or PII
5. Use appropriate levels: error (broken), warn (unexpected but handled), info (normal ops)
6. Keep log messages concise — no full request/response bodies
