# AI Agent Skills

Six reusable skills that make AI coding agents more consistent. A skill is a reusable set of instructions that teaches an AI agent how to perform a specific task the same way every time.

| Skill | Purpose |
|-------|---------|
| `security-review` | Reviews code for secrets, auth gaps, unsafe logging, weak validation, CORS issues, and vulnerable dependencies. Suggests fixes, does not auto-change. |
| `dev-setup` | Starts the app locally: install, env vars, port, health check (Next.js). |
| `deployment-guardrail` | Blocks direct or unauthorized deploys; checks CI/CD, approvals, tests, config, rollback readiness. |
| `environment-cloning` | Produces a migration checklist for cloning a serverless environment through approved IaC. |
| `quality-gate` | Checks naming/structure, tests/lint/format/types, and error handling/docs/config. |
| `ui-development` | Decides reuse vs. create for components; enforces design-system, accessibility, and responsive standards. |

## Structure

```
skills/
  <skill-name>/
    SKILL.md        # YAML frontmatter (name, description) + instructions
```

## Install

Each agent tool looks for skills in its own folder. Common examples (check your tool's current docs):

- Claude Code: `.claude/skills/<skill-name>/SKILL.md` in a project, or `~/.claude/skills/` for personal use.
- Other agents: copy the `SKILL.md` content to wherever that tool reads custom instructions.

Copy the `skills/*` folders into that location.

## Customize before use

1. `dev-setup`: edit the "Project configuration" block (commands, port, health URL).
2. `deployment-guardrail`: adjust the gate table to your pipeline and approval rules.
3. `environment-cloning`: set your IaC tool and naming conventions.
4. `quality-gate`: confirm the commands match your scripts.
5. `ui-development`: point it at your component library and design tokens.

## Important

Skills are instructions, not guarantees. An agent can still miss something or ignore a rule. Real enforcement comes from tests, permissions, CI/CD policies, branch protection, and infrastructure controls. Skills make agents consistent. Guardrails make them safe.

## Contributing

1. Branch from `main`, edit a `SKILL.md`, open a merge request.
2. Keep each `SKILL.md` under 500 lines; one skill per folder.
3. Never include company-specific names, secrets, account IDs, or internal URLs.
