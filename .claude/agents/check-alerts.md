---
name: check-alerts
description: Check for error monitoring and alerting integration
allowedTools: ["View", "Grep"]
---

Check for error monitoring and alerting:

1. Search `package.json` for: `@sentry/nextjs`, `@sentry/node`, `@axiomhq/js`, `@datadog/browser-rum`, `betterstack`, `logtail`
2. Check for Sentry config files: `sentry.client.config`, `sentry.server.config`, `sentry.edge.config`
3. Check `next.config` for Sentry webpack plugin or similar
4. Search for error reporting calls in catch blocks and error boundaries
5. Check `vercel.json` or CI config for deployment notification webhooks

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
MONITORING_SDK: [name or "none"]
CONFIGURED: [yes|no]
ERROR_BOUNDARIES_REPORT: [yes|no]
DEPLOYMENT_NOTIFICATIONS: [yes|no]
ISSUES:
- [description]
```

Rules:
- 🔴 if no error monitoring SDK installed
- 🟡 if SDK installed but not fully configured (missing error boundary integration)
- 🟢 if monitoring configured and error boundaries report to it
- Keep output under 150 words
