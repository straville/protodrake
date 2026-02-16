# Kata 00: CLAUDE.md Files & Sub-Agent Architecture (Reference)

## Overview

A complete example of layered `CLAUDE.md` files for a team-sized full-stack project, with reusable sub-agent prompt templates. Shows how to structure project knowledge so Claude Code (and its sub-agents) stay focused, consistent, and parallelizable.

The fictional project: **Acme Platform** — Next.js frontend, Express API, PostgreSQL, Redis, Cloudflare Workers edge layer, GitHub Actions CI/CD.

---

## Directory Structure

```
00-CLAUDE.mds-subagents-example/
├── CLAUDE.md                          # Root — project overview, conventions, commands,
│                                      #   git workflow, sub-agent dispatch patterns
├── .claude/
│   └── agents/
│       ├── backend.md                 # Reusable prompt: backend sub-agent
│       ├── frontend.md                # Reusable prompt: frontend sub-agent
│       ├── test.md                    # Reusable prompt: test sub-agent
│       └── infra.md                   # Reusable prompt: infra sub-agent
│
├── src/
│   ├── backend/
│   │   └── CLAUDE.md                  # Backend layer: route→service→DB/cache pattern,
│   │                                  #   Drizzle ORM, Redis cache, driver wrappers
│   └── frontend/
│       └── CLAUDE.md                  # Frontend layer: Next.js App Router, shadcn/ui,
│                                      #   server vs client components, asset rules
├── tests/
│   └── CLAUDE.md                      # Testing: Vitest + Playwright, unit vs integration
│                                      #   vs E2E boundaries, fixture/mock patterns
└── infra/
    └── CLAUDE.md                      # Infrastructure: Cloudflare Workers, Docker,
                                       #   GitHub Actions workflows, deployment
```

---

## Key Files Explained

### `CLAUDE.md` (root)

The single most important file. Every Claude Code session reads this automatically. Contains:

- **Quick reference table** — maps each layer to its path, runtime, and entry point
- **Conventions** — TypeScript strict mode, naming, imports, env var handling, error patterns
- **Database / Cache / Edge rules** — how to use Drizzle, Redis, and Cloudflare Workers
- **Commands** — every `pnpm` script the project uses
- **Sub-Agent Workflow** — when to split into agents, dispatch patterns for features/bugs/migrations, and parallel vs sequential rules

### `.claude/agents/*.md`

Prompt templates for the Task tool sub-agents. Each file scopes one agent to a single layer:

| Agent | Owns | Does not touch |
|-------|------|----------------|
| `backend.md` | `src/backend/` | frontend, infra, test files |
| `frontend.md` | `src/frontend/` | backend, infra, test files |
| `test.md` | `tests/` + colocated `*.test.ts` | implementation code |
| `infra.md` | `infra/` + `.github/workflows/` | application code in `src/` |

Each agent file includes: scope boundaries, pre-flight reading list, responsibilities, and a post-completion checklist (run tests, report changes).

### Subfolder `CLAUDE.md` files

Auto-loaded by Claude Code when working in that directory. They provide layer-specific patterns without bloating the root file:

- **`src/backend/CLAUDE.md`** — route→service→DB layering, transaction patterns, cache read-through, "adding a new endpoint" step-by-step
- **`src/frontend/CLAUDE.md`** — Server Components vs Client Components, data fetching, shadcn/ui usage, "adding a new page" step-by-step
- **`tests/CLAUDE.md`** — test stack, colocated unit tests, integration test DB setup, E2E auth shortcuts, common mistakes
- **`infra/CLAUDE.md`** — worker constraints, Docker multi-stage build, CI workflow table, "making infra changes" checklist

---

## How Sub-Agent Dispatch Works

The root `CLAUDE.md` defines three dispatch patterns:

**Feature spanning multiple layers:**
```
1. Explore agent    → understand current code, find integration points
2. Backend agent    → DB schema, API endpoints, cache logic
3. Frontend agent   → UI components, data fetching          (parallel with 2 if API contract agreed)
4. Test agent       → unit + integration tests
```

**Bug fix:**
```
1. Explore agent → reproduce, find root cause
2. Fix agent     → implement the fix
3. Test agent    → regression test + run suite
```

**DB migration + cache invalidation:**
```
1. Backend agent → schema, migration, queries, cache (single agent, sequential)
2. Test agent    → integration tests against test DB
```

---

## Use This As a Starting Point

Copy the structure into your own project and customize:

1. Replace the fictional tech stack with your actual stack
2. Update conventions to match your team's standards
3. Add/remove agent templates based on your project layers
4. Adjust the dispatch patterns to fit your typical workflows
