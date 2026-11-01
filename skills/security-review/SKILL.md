---
name: security-review
description: Review code for security problems and report findings with suggested fixes, without silently rewriting code. Use whenever the user asks for a security review, audit, or "is this safe", before a merge request or release, or when code touches API endpoints, authentication, secrets, logging, user input, CORS, or dependencies. Trigger even if the user only says "check this for issues" and the change involves any of those areas.
---

# Security Review

Review code for common security problems and report them. This skill is advisory: it finds and explains issues and proposes fixes. It does not apply fixes unless the user explicitly asks.

## Ground rules

1. **Report, don't rewrite.** Default to read-only. Show a suggested fix as a snippet in the report. Only edit files if the user says "apply the fixes" (and then only the ones they approve).
2. **Never print a secret in full.** If you find a key or token, show only the first 4 characters and mask the rest (e.g. `AKIA****`). Never copy secrets into your report, commit messages, or logs.
3. **Say what you checked and what you did not.** An empty result is not proof of safety.
4. **Stay in scope.** Review the files or diff the user pointed to. If none given, review the current branch diff against the default branch.

## Procedure

### Step 1: Define scope
- Identify the language, framework, and what changed (`git diff <default-branch>...HEAD --stat`).
- List the entry points in scope: API routes, handlers, server actions, background jobs.

### Step 2: Run the checklist
Work through every category below. For each, record: Pass, Finding, or Not applicable.

| # | Category | What to look for |
|---|----------|------------------|
| 1 | Secrets | Hardcoded API keys, tokens, passwords, private keys, connection strings; `.env` files committed; secrets in client-side bundles (e.g. a private key in a `NEXT_PUBLIC_*` variable); secrets in test fixtures or comments |
| 2 | Authentication | Endpoints reachable without login that should not be; weak or missing session/token validation; tokens that never expire |
| 3 | Authorization | Missing role/ownership checks (user A reading user B's data, i.e. IDOR); trusting client-sent user IDs or roles; admin routes without admin checks |
| 4 | Input validation | Unvalidated request body, query, params, headers, file uploads; injection risks (SQL/NoSQL, command, path traversal); unsafe HTML rendering (XSS) |
| 5 | Sensitive data in logs | Logging tokens, passwords, full request bodies, PII, full error objects containing secrets |
| 6 | CORS and headers | `Access-Control-Allow-Origin: *` on authenticated APIs; credentials allowed with a wildcard origin; missing security headers where relevant |
| 7 | Dependencies | Known-vulnerable or abandoned packages; unpinned versions; suspicious new packages. Run the project's audit command if available (`npm audit`, `pip-audit`, etc.) and report its output |
| 8 | Error handling | Stack traces or internal details returned to clients; swallowed errors on auth paths |
| 9 | Data exposure | Responses returning more fields than needed; verbose API errors; public storage buckets referenced in config |

### Step 3: Rate each finding
- **Critical**: exploitable now, exposes secrets or data, or bypasses auth.
- **High**: likely exploitable with modest effort.
- **Medium**: weakens defenses; exploitable under certain conditions.
- **Low**: hygiene or defense in depth.

### Step 4: Write the report
Use exactly this structure:

```
# Security Review: <scope>

**Scope reviewed:** <files/diff>
**Not reviewed:** <what was out of scope or could not be checked>
**Summary:** <N> critical, <N> high, <N> medium, <N> low

## Findings
### [SEVERITY] <short title>
- **Where:** <file:line>
- **Issue:** <what is wrong and why it matters>
- **Suggested fix:** <description + minimal snippet>
- **Verify by:** <how to confirm it is fixed>

## Checklist results
<table of the 9 categories: Pass / Finding / N/A>

## Recommended next steps
<ordered list, highest severity first>
```

### Step 5: Hand back control
End by asking which findings the user wants fixed. Do not start editing before they answer.

## If a real secret is found
1. Flag it as Critical at the top of the report.
2. Tell the user to **rotate the secret** immediately. Removing it from the code is not enough, because it stays in git history.
3. Suggest moving it to the approved secrets manager or CI/CD protected variables.

## Limits (state these in every report)
This review is a set of instructions followed by an AI agent. It can miss issues. It does not replace automated scanning (SAST, secret scanning, dependency scanning in CI), peer review, or a professional security assessment for sensitive systems.
