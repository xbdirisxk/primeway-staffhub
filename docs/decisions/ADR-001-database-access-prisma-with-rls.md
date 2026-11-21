# ADR-001: Database access through Prisma, run as the signed-in user

- **Status:** accepted
- **Date:** 2026-11-20
- **Features affected:** all data features (F2 onwards)

## Context
The spec requires branch separation in two layers: server code and Postgres row-level security (RLS). Prisma, Zak's preferred ORM, normally connects as a privileged database user, and privileged users skip RLS — so the second layer would silently disappear.

## Options considered
1. **supabase-js from the server with the user's session** — RLS applies automatically, but means a second query style alongside Prisma and weaker typing for complex queries.
2. **Prisma, accept one layer** — simplest, but any missed `where branch_id = …` leaks another branch's data. Not acceptable for health and DBS data.
3. **Prisma inside a per-request transaction that switches to the `authenticated` role and sets the user's verified claims** — keeps Prisma and keeps RLS.

## Decision
Option 3. Every web-app query runs through `withUser(ctx, fn)`, which opens a transaction, sets `request.jwt.claims`, `app.branch_id` and `app.client_ip` with transaction-local `set_config(..., true)`, runs `set local role authenticated`, then executes the queries.

## Why
Keeps the stack Zak knows, keeps two independent layers of protection, and gives audit triggers the actor and IP for free.

## Consequences
- The raw Prisma client lives only in `src/lib/db/privileged.ts`; ESLint blocks importing it in the web app. The worker and scripts use it.
- RLS policies, grants, helper functions and triggers are written in SQL migrations — Prisma does not manage them.
- Every query is in a transaction; keep request transactions short. Use the Supabase transaction pooler (`pgbouncer=true`) for the app and a direct connection for migrations.
- RLS tests must cover every table. Revisit if performance suffers or Prisma changes how it handles roles.
