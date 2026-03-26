---
name: check-logging
description: Audit logging standards and check for sensitive data in logs
allowedTools: ["View", "Grep"]
---

Check logging practices:

1. Is a structured logger installed? Search `package.json` for `pino`, `winston`, etc.
2. Search for `console.log` in `app/api/` and `lib/` — flag as should-be-structured
3. Search for empty catch blocks: `catch` followed by `{}` or only a comment
4. Search log statements for sensitive variable names: `password`, `token`, `secret`, `apiKey`, `authorization`, `creditCard`
5. Search for PII patterns in log statements:
   - Email patterns (`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+`)
   - IP address patterns (`\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b`)
   - User IDs being logged directly (e.g., `log.*userId`, `console.*user_id`)
6. Check for error logging that includes stack traces

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
LOGGER: [library name or "console only"]
CONSOLE_LOG_COUNT: [number in production code]
EMPTY_CATCH_BLOCKS: [count]
SENSITIVE_DATA_IN_LOGS: [count]
PII_IN_LOGS: [count]
ISSUES:
- [filepath]: [description]
```

Rules:
- 🔴 if sensitive data or PII found in logs, or many empty catch blocks
- 🟡 if using console.log instead of structured logger
- 🟢 if structured logger configured and no sensitive data logged
- Keep output under 200 words
