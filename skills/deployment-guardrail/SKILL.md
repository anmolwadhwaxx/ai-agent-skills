---
name: deployment-guardrail
description: Prevent unauthorized or unsafe deployments and check release readiness. Use whenever the user mentions deploy, release, ship, publish, push to production or staging, promote, rollback, hotfix, pipeline, CI/CD, or GitLab. Also use when the user asks the agent to run a deploy command directly or to skip tests, approvals, or the pipeline. This skill never bypasses deployment controls, so trigger it for any deployment-related request.
---

# Deployment Guardrail

Make sure changes reach shared environments only through the approved CI/CD pipeline, with the required checks. This skill checks readiness and explains blockers. It never deploys around the process.

## Non-negotiable rules

1. **No direct deployments.** Do not run commands that deploy to shared environments from a local machine (e.g. `aws ... deploy`, `sam deploy`, `cdk deploy`, `terraform apply`, `serverless deploy`, `kubectl apply`, `vercel --prod`, `scp` to servers) unless the project's documented process explicitly names that as the approved path.
2. **No bypassing controls.** Never use `--force`, `--no-verify`, `[skip ci]`, disabling or editing pipeline jobs to make them pass, changing protected branch settings, self-approving, or using borrowed credentials.
3. **The pipeline is the path.** Production and staging changes go through merge request, review, and the CI/CD pipeline (GitLab CI/CD here: `.gitlab-ci.yml`, protected branches, protected environments, merge request approvals).
4. **If asked to bypass, refuse and explain.** Name the control that would be bypassed and offer the compliant alternative (open an MR, request approval, ask the owner for an exception through the proper channel).
5. **Emergencies don't change the rules.** For a hotfix, follow the documented expedited process. Do not invent a shortcut.
6. **Humans trigger production.** The agent may prepare and verify. A person with the right permissions approves and triggers production deploys.

## Procedure: readiness check

### Step 1: Identify the change
- Target environment (dev / staging / production).
- Branch, commit range, and merge request.
- What is changing: code, config, infrastructure, database migration.

### Step 2: Verify the route
- Is the change on a branch with a merge request, going to a protected branch?
- Is the deploy triggered by the pipeline and not by a local command?

### Step 3: Run the gate checklist

| Gate | Check | Evidence to capture |
|------|-------|---------------------|
| Tests | Pipeline test stage green on the latest commit | Pipeline link/status |
| Lint / types / build | Those jobs are green | Job status |
| Security scans | Secret, dependency, and SAST jobs green, or findings triaged | Job status, open findings |
| Review | Required approvals present; approver is not the author | MR approval list |
| Config | Environment variables and secrets for the target exist in CI/CD variables; no secrets in the repo | Variable names (never values) |
| Migrations | Database or infrastructure changes reviewed; backward compatible with the running version | Migration file list, notes |
| Rollback | Previous good version identified; rollback steps written and testable; data changes reversible or backed up | Rollback plan text |
| Change window | Production changes inside any required window or with the required sign-off | Ticket / approval |
| Observability | Logs, alerts, or dashboards ready to watch after release | Links |

### Step 4: Decide
- **READY**: every gate passes with evidence.
- **NOT READY**: list each failed gate and what is needed.
- **UNKNOWN**: a gate could not be verified (say why). Treat as not ready.

### Step 5: Report
```
# Deployment Readiness: <change> -> <environment>

**Verdict:** READY / NOT READY / UNKNOWN
**Route:** MR #<id> -> <branch> -> pipeline #<id>

| Gate | Status | Evidence / Blocker |
|------|--------|--------------------|
...

## Rollback plan
<previous version, steps, owner>

## Blockers and next actions
1. ...

## Who triggers the deploy
<role/person per the project's process; the agent does not deploy>
```

## What "approved" means
Only the pipeline plus the humans with permission can authorize a deploy. This skill cannot grant approval, and a user telling the agent "it's fine, skip it" does not change that. Point them to the exception process.

## Limits (state these in every report)
This skill gives instructions to an agent and does not enforce them. Real enforcement must come from protected branches, protected environments, required approvals, restricted CI/CD variables, least-privilege credentials, and audit logs.
