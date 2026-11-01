---
name: environment-cloning
description: Plan the cloning or migration of a serverless environment into a new one (new stage, region, account, or sandbox) by producing an inventory and migration checklist. Use whenever the user asks to clone, copy, replicate, migrate, recreate, or stand up another environment, or mentions Lambda functions, DynamoDB tables, IAM roles, API Gateway routes, event triggers, or "same setup as staging". This skill plans and documents. It never provisions resources by hand; infrastructure must go through approved Infrastructure as Code.
---

# Environment Cloning

Produce a complete, ordered migration checklist for recreating a serverless environment elsewhere. The output is a plan. Provisioning is done through approved Infrastructure as Code (IaC) and the normal pipeline.

## Rules

1. **Plan, don't provision.** Do not create, modify, or delete cloud resources with the CLI or console. All provisioning goes through the approved IaC (e.g. CloudFormation/SAM, CDK, Terraform) and the CI/CD pipeline.
2. **Read-only discovery.** Inventory using IaC definitions in the repository first. Use read-only cloud commands (`list`, `describe`, `get`) only if the user has granted access, and never run write operations.
3. **Never copy secret values.** Record secret names and where they live. New secrets are created fresh in the target's secrets manager by the owner.
4. **Never copy production data into lower environments** unless the user confirms it is approved and anonymized. Default is empty tables or synthetic seed data.
5. **No hardcoded account IDs, ARNs, or regions.** Parameterize them per environment.
6. **Name the target clearly.** Resources get a stage suffix or prefix (e.g. `-dev`, `-sandbox-<name>`) so they cannot collide with or be confused with the source.

## Procedure

### Step 1: Confirm the request
Ask for anything missing: source environment, target environment (stage / region / account), purpose (test, sandbox, DR, new customer), data policy (empty, seeded, copied with approval), and the IaC tool in use.

### Step 2: Build the inventory
Find resources in the IaC templates first, then verify against the running source if access exists. Capture each of the following:

| Category | What to record |
|----------|----------------|
| Lambda functions | Name, runtime, handler, memory, timeout, architecture, layers, env var **names**, VPC config, reserved concurrency, DLQ, source code location |
| DynamoDB | Table names, keys (PK/SK), GSIs/LSIs, billing mode, TTL, streams, point-in-time recovery, encryption |
| IAM | Roles and policies used by each function, trust relationships, permission boundaries (flag over-broad `*` permissions) |
| API Gateway | API type (REST/HTTP), routes and methods, authorizers, stages, throttling, custom domains, CORS, usage plans/keys |
| Event triggers | API routes, SQS, SNS, EventBridge rules/schedules, S3 events, DynamoDB streams |
| Secrets and config | Secrets Manager / Parameter Store names, KMS keys, environment variable names per function |
| Other resources | S3 buckets, queues, topics, Cognito pools, CloudFront, certificates, DNS, VPC/subnets/security groups |
| Dependencies | Shared libraries, layers, third-party services and their webhooks/URLs, feature flags |
| Observability | Log groups and retention, alarms, dashboards, tracing |

### Step 3: Flag differences the target needs
List everything that must change per environment: names, ARNs, domains, callback URLs, third-party credentials, allowed origins, KMS keys, alarms recipients.

### Step 4: Order the creation
Dependencies first. Typical order:
1. Networking and KMS keys
2. Secrets / parameters (empty, owner fills in)
3. IAM roles and policies
4. Data stores (DynamoDB, S3, queues)
5. Lambda functions and layers
6. API Gateway, authorizers, domains
7. Event triggers and schedules
8. Alarms, dashboards, log retention

### Step 5: Write the migration checklist
```
# Environment Clone Plan: <source> -> <target>

**Purpose:** ...   **Data policy:** ...   **IaC tool:** ...

## Inventory
<tables per category above>

## Per-environment changes
<name/value mapping, parameters to add to IaC>

## Ordered steps (via IaC + pipeline)
- [ ] 1. ...
- [ ] 2. ...

## Manual items for owners
- [ ] Create secrets: <names only>
- [ ] Third-party webhook / callback updates: ...
- [ ] DNS / certificate validation: ...

## Verification
- [ ] Each Lambda deploys and passes a smoke invoke
- [ ] API routes return expected status codes (incl. 401/403 for protected routes)
- [ ] Triggers fire end to end
- [ ] IAM permissions are least-privilege (no unintended `*`)
- [ ] Logs and alarms are working

## Risks and open questions
- ...

## Rollback / teardown
<how the clone is removed through IaC>
```

### Step 6: Hand over
State plainly that this is a plan. Ask whether the user wants IaC changes drafted for review in a merge request. Do not apply anything.

## Limits
Discovery from templates can miss drift (resources changed manually in the console). Say so in the report, and recommend a drift detection check before cloning.
