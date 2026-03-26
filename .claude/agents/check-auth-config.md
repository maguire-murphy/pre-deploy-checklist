---
name: check-auth-config
description: Review Supabase auth configuration for security best practices
allowedTools: ["mcp_supabase"]
---

Using Supabase MCP, review auth configuration:

1. Password reset token expiration (should be ≤ 30 min)
2. OTP expiration
3. Magic link expiration
4. Session lifetime
5. Redirect URLs (flag localhost or wildcards)
6. Email confirmation required for signup?
7. Is MFA/TOTP enabled? (should be available for sensitive apps)
8. Minimum password length setting (should be ≥ 8, ideally 10+)
9. If phone auth is enabled, is phone verification required?

Return ONLY this format:

```
STATUS: 🟢|🟡|🔴
SETTINGS:
- Password reset expiration: [value] [🟢|🟡|🔴]
- OTP expiration: [value] [🟢|🟡|🔴]
- Email confirmation: [enabled|disabled] [🟢|🔴]
- MFA/TOTP: [enabled|disabled] [🟢|🟡]
- Min password length: [value] [🟢|🟡|🔴]
- Phone auth verification: [enabled|disabled|N/A] [🟢|🟡]
- Redirect URLs: [list any risky ones]
RECOMMENDATIONS:
- [specific fix]
```

Rules:
- 🔴 if reset tokens > 30 min, email confirmation disabled, or password min length < 8
- 🟡 if MFA not enabled, minor config improvements suggested, or phone auth without verification
- 🟢 if all settings follow best practices
- Keep output under 200 words
