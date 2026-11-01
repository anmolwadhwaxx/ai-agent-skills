---
name: quality-gate
description: Run a pre-merge quality check on code the agent wrote or changed, covering naming and structure, tests/lint/format/type checks, and missing error handling, documentation, and configuration. Use whenever the user asks to review, check, clean up, finalize, or "make this PR-ready", before opening a merge request, after finishing a feature or bugfix, or when the agent has just generated code. Trigger even if the user only says "are we done?".
---

# Quality Gate

A self-check the agent runs before calling work done. It covers exactly three areas. Run all three, report honestly, and do not mark work complete while a required check is failing.

## Rules

1. **Run the real tools.** Use the project's own scripts (`package.json` scripts, `Makefile`, CI config). Do not claim a check passed unless you ran it and saw the result.
2. **Never weaken the gate.** Do not disable lint rules, delete or skip failing tests, lower coverage thresholds, add `// @ts-ignore`, or edit config to make checks pass. If a check seems wrong, say so and ask.
3. **Report, then fix only on approval,** except for purely mechanical formatting the project's formatter applies, which may be applied if the user asked for cleanup.
4. **Scope to the change.** Check the files touched, plus project-wide commands where the tooling requires it.
5. **Be specific.** Every finding includes a file and line and a suggested fix.

## Procedure

### Step 1: Discover the project's tooling
Read the project config to find the commands. Examples (use what exists):
- Tests: `npm test`, `pytest`, `go test ./...`
- Lint: `npm run lint`, `ruff check`, `eslint .`
- Format: `npm run format:check`, `prettier --check .`, `black --check .`
- Types: `npm run typecheck`, `tsc --noEmit`, `mypy .`
- Build: `npm run build`
If a tool is not configured, record "not configured". Do not invent one.

### Step 2: Check 1, Naming conventions and project structure
- Files, folders, components, functions, variables, constants, and types follow the project's documented or evident conventions (casing, suffixes, test file placement).
- New files live in the correct directory for their kind (components, hooks, services, utils, API routes, tests).
- No duplicate utilities where one already exists; no dead files, stray `console.log`, commented-out code, or leftover TODOs without tickets.
- Imports follow project patterns (aliases, no deep relative paths if the project forbids them).

### Step 3: Check 2, Tests, lint, format, and types
Run each available command and record the result:
- Unit tests pass. New or changed logic has tests, including edge cases and failure paths. Bug fixes have a regression test.
- Lint passes with zero errors (warnings listed).
- Formatting check passes.
- Type check passes with no new `any`, suppressions, or non-null assertions without reason.
- Build succeeds, if relevant.

### Step 4: Check 3, Error handling, documentation, and configuration
- **Error handling:** failure paths handled for network calls, parsing, I/O, and external services; errors are logged without sensitive data and surfaced to the user appropriately; no empty `catch` blocks; async errors are not unhandled.
- **Documentation:** public functions/modules have clear docs where the project expects them; README or docs updated for new commands, endpoints, or behavior; changelog entry if the project keeps one.
- **Configuration:** new environment variables added to `.env.example` and documented; no hardcoded URLs, ports, or credentials; defaults are safe; config validated at startup where the project does that.

### Step 5: Verdict and report
```
# Quality Gate: <change>

**Verdict:** PASS / PASS WITH WARNINGS / FAIL

| Area | Check | Result | Notes |
|------|-------|--------|-------|
| 1 Naming & structure | Conventions | ... | ... |
| 1 Naming & structure | Placement / duplication | ... | ... |
| 2 Automated | Tests | pass/fail/not configured | <counts> |
| 2 Automated | Lint | ... | ... |
| 2 Automated | Format | ... | ... |
| 2 Automated | Types | ... | ... |
| 3 Completeness | Error handling | ... | ... |
| 3 Completeness | Documentation | ... | ... |
| 3 Completeness | Configuration | ... | ... |

## Issues to fix (highest impact first)
1. <file:line> <problem> -> <suggested fix>

## Not checked
<anything skipped or not configured, and why>
```

Verdict rules: any failing test, type error, lint error, or build failure is **FAIL**. Only warnings or "not configured" items give **PASS WITH WARNINGS**.

## Limits
This gate catches common, mechanical problems. It does not replace code review, security scanning in CI, or product testing.
