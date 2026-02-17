# Claude Code Agent System Template

7-agent system for Claude Code projects with document-driven workflow.

## Quick Start

1. **Use this template** → "Use this template" button on GitHub
2. Edit `docs/SSOT.md` with your project requirements
3. Run `/pm` to initialize specs

## Agent Structure

```
/pm          — Spec Owner + Conflict Resolver
/deploy      — Infra/Env Auditor + Deployment Gate
/review      — PR/Release Gate Owner
/fe-lead     — FE Architecture + Security Guardrails
/fe-build    — FE Implementation + Unit Tests
/be-lead     — BE Architecture + Security Guardrails
/be-build    — BE Implementation + Integration Tests
```

## Workflow

```
1. /pm           → Define specs, AC, API contract
2. /fe-lead      → FE architecture decisions
   /be-lead      → BE architecture decisions
3. /fe-build     → Implement frontend
   /be-build     → Implement backend
3.5 /deploy      → Verify env vars, infra config
4. /review       → PR Gate (code + infra)
5. /review       → Release Gate
```

## Documents

| File | Owner | Purpose |
|------|-------|---------|
| `docs/SSOT.md` | /pm | Requirements, AC, priorities |
| `docs/API_CONTRACT.md` | /pm | API interface contract |
| `docs/DECISIONS.md` | shared | Architecture decisions |
| `docs/TEST_PLAN.md` | /review | Test coverage tracking |
| `docs/DEPLOY.md` | /deploy | Env vars, infra, runbook |

## Customization

- **Tech stack**: Edit `docs/DEPLOY.md` env var table for your stack
- **Checklists**: Adjust `CLAUDE.md` FE/BE/Infra checklists
- **Agents**: Modify `.claude/commands/*.md` for your workflow
- **Remove agents**: Delete unused command files and update `CLAUDE.md`
