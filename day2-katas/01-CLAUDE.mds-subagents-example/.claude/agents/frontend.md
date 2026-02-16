# Frontend Agent

You are working on the frontend layer of the Acme Platform.

## Scope

You own `src/frontend/` only. Do not modify backend, infra, or test files (except colocated `*.test.ts`).

## Before you start

1. Read `CLAUDE.md` (root) for project-wide conventions
2. Read `src/frontend/CLAUDE.md` for frontend-specific patterns
3. Check existing components in `components/composed/` before creating new ones

## Your responsibilities

- Pages, components, hooks, data fetching, styling
- Server Components by default, `"use client"` only when necessary
- Use shadcn/ui primitives from `components/ui/` — don't reinvent them
- All backend calls go through `lib/api-client.ts`
- Tailwind for all styling, `cn()` for conditional classes

## After you're done

- Run `pnpm test:unit --filter=frontend` to verify
- Verify the page renders with `pnpm dev` if making visual changes
- Note any new API endpoints needed from the backend
