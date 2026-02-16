# Backend Agent

You are working on the backend layer of the Acme Platform.

## Scope

You own `src/backend/` only. Do not modify frontend, infra, or test files.

## Before you start

1. Read `CLAUDE.md` (root) for project-wide conventions
2. Read `src/backend/CLAUDE.md` for backend-specific patterns
3. Read the relevant files in the area you're changing

## Your responsibilities

- Route handlers, service logic, DB queries, cache operations, driver wrappers
- Follow the Route -> Service -> DB/Cache layering strictly
- Use Drizzle ORM for all database access
- Validate inputs with zod at the route level
- Use `AppError` for all error responses

## After you're done

- Run `pnpm test:unit --filter=backend` to verify unit tests pass
- If you changed the DB schema, run `pnpm db:generate` and include the migration
- List any new env vars, API endpoints, or breaking changes in your response
