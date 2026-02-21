# Claude Code Agent System Template v2

15-agent system for large-scale Claude Code projects with page-scoped, document-driven workflow. Includes `/auto` orchestrator for automated pipeline execution with parallel sub-agent support.

## Quick Start

### 1. Create Your Project

Click **"Use this template"** on GitHub, then clone your new repository.

### 2. (Optional) MCP Setup

If you use Codex or other MCP servers, copy the example config:
```bash
cp .mcp.json.example .mcp.json
```
Edit `.mcp.json` to match your environment. This file is gitignored (personal config).

### 3. Automated Workflow (Recommended)

Use `/auto` to run the entire pipeline automatically:

```
/auto init

Example input:
"쇼핑몰 프로젝트. 주요 페이지: 로그인, 상품 목록, 상품 상세, 장바구니, 결제.
사용자는 소셜 로그인(Google, Kakao)으로 가입하고, 상품을 검색/필터링하여
장바구니에 담고 결제할 수 있다. 관리자 페이지는 별도 Phase에서 다룬다."
```

Then build pages and release:
```
/auto page login,product-list,cart    ← Spec → Design → API → Build → Review (parallel)
/auto release                         ← Final release gate
```

Or let `/auto` detect the next step automatically:
```
/auto                                 ← Reads INDEX.md status, suggests next action
```

See [Auto Orchestrator](#auto-orchestrator-auto) below for all subcommands.

### 3-alt. Manual Workflow

You can also run each agent individually for fine-grained control.

#### Phase A — Project Init

```
/pm-lead              ← Requirements → SSOT.md + INDEX.md
/pm-review            ← Validate SSOT + INDEX consistency
/design-lead          ← Establish design system (DESIGN_SYSTEM.md)
/design-review        ← Validate design system
/fe-lead              ← FE architecture decisions (DECISIONS/fe-arch.md)
/be-lead              ← BE architecture decisions (DECISIONS/be-arch.md)
```

#### Phase B — Per-Page Build (repeat for each page)

```
/pm-build login           ← Page spec (spec.md)
/pm-review                ← Validate spec consistency
/design-build login       ← Page design (design.md)
/design-review            ← Validate design fidelity
/api-orch login           ← API definition (api.md)
/be-build login           ← BE implementation + tests
/fe-build login           ← FE implementation + tests
/be-review                ← BE code review
/fe-review                ← FE code review
/design-review login      ← Design fidelity check
```

> B-4 (`/be-build`) and B-5 (`/fe-build`) can run in parallel after `api.md` is finalized (B-3).

#### Phase C — Release

```
/review                   ← Final release gate
```

## Agent Structure (15 agents)

```
Auto
  /auto            — Orchestrator: automated pipeline with parallel sub-agents

PM Group
  /pm-lead         — Requirements → pages, SSOT + INDEX
  /pm-build        — Per-page detailed spec
  /pm-review       — Cross-page consistency gate

Design Group
  /design-lead     — Project design guidelines
  /design-build    — Per-page design spec
  /design-review   — Design fidelity gate

Orchestrator
  /api-orch        — Per-page API definition (req/res)

FE Group
  /fe-lead         — FE architecture guardrails
  /fe-build        — Implementation + tests
  /fe-review       — FE code review + test verification

BE Group
  /be-lead         — BE architecture + domain
  /be-build        — Implementation + tests
  /be-review       — BE code review + test verification

Release
  /review          — Final release gate (integrated)
```

## Auto Orchestrator (`/auto`)

The `/auto` command automates the Phase A/B/C workflow, running sequential steps inline and parallel steps via Task sub-agents.

### Subcommands

| Command | Example | Description |
|---------|---------|-------------|
| `init` | `/auto init` | Phase A: PM Lead → PM Review → Design Lead → Design Review → FE Lead + BE Lead (parallel) |
| `page` | `/auto page login,cart` | Phase B full: spec → design → API → build (parallel) → review (parallel) |
| `build` | `/auto build login` | B-4+B-5 only: FE + BE build in parallel (worktree-isolated) |
| `review` | `/auto review login` | Review only: BE Review + FE Review + Design Review in parallel |
| `release` | `/auto release` | Phase C: Release Gate |
| _(none)_ | `/auto` | Auto-detect next action from INDEX.md page status |

### Parallel Execution

`/auto` uses Task sub-agents for independent work:

- **A-3**: `/fe-lead` + `/be-lead` run simultaneously
- **B-4+B-5**: `/be-build` + `/fe-build` run simultaneously per page (worktree-isolated)
- **Review**: `/be-review` + `/fe-review` + `/design-review` run simultaneously per page
- **Cross-page**: Multiple pages build/review in parallel (e.g., 2 pages = 4 build tasks + 6 review tasks)

### Key Principles

- **INDEX.md single writer**: Only the orchestrator updates INDEX.md; sub-agents never touch it
- **Sequential where required**: B-1 → B-2 → B-3 always run in order (output dependency)
- **State-aware skipping**: Already-completed steps are skipped based on page status
- **Failure isolation**: One page failing doesn't block others
- **Review gate**: Max 3 retry iterations per reviewer; escalates to Lead on failure

## Document Structure (2-Tier)

### Tier 1: Global (project-level)

| File | Owner | Purpose |
|------|-------|---------|
| `docs/SSOT.md` | /pm-lead | Project overview + index |
| `docs/pages/INDEX.md` | /pm-lead | Page inventory, routes, status |
| `docs/DESIGN_SYSTEM.md` | /design-lead | Design tokens, methodology |
| `docs/API_STANDARDS.md` | /api-orch + /be-lead | API common rules |
| `docs/DECISIONS/*.md` | shared | Architecture decisions (ADR) |
| `docs/DEPLOY.md` | /review | Env vars, infra, runbook |
| `docs/TEST_PLAN.md` | /review | Test coverage tracking |

### Tier 2: Per-Page (folder-scoped)

```
docs/pages/{page_id}/
├─ spec.md      ← PM Builder (purpose, user stories, AC, states)
├─ design.md    ← Design Builder (layout, components, accessibility)
├─ api.md       ← API Orchestrator (endpoints, req/res, errors)
└─ tests.md     ← Reviewers (test cases, coverage)
```

## Context Recovery (Cold Start)

New session needs only 3~5 files:
1. `docs/SSOT.md` (project overview)
2. `docs/pages/INDEX.md` (page status map)
3. `docs/pages/{page_id}/spec.md` (current task)
4. As needed: design.md, api.md, DESIGN_SYSTEM.md, DECISIONS/

## Review Feedback Loop

When a Reviewer returns **"hold"**, the flow is:
1. Builder fixes issues based on hold reasons
2. Builder re-invokes the **same Reviewer**
3. Re-review scope: **modified diff + hold items only** (not full re-review)
4. Max **3 iterations** — escalate to Lead if unresolved

## Customization

- **Tech stack**: Decisions recorded in `docs/DECISIONS/fe-arch.md` and `docs/DECISIONS/be-arch.md` during Phase A
- **Checklists**: Adjust `CLAUDE.md` FE/BE/Design/Infra checklists
- **Agents**: Modify `.claude/commands/*.md` for your workflow
- **Remove agents**: Delete unused command files and update `CLAUDE.md`
- **Templates**: Edit `docs/pages/TEMPLATE_*.md` for your page doc format
- **MCP**: Copy `.mcp.json.example` to `.mcp.json` and configure
