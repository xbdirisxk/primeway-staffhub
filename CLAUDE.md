# Primeway StaffHub — instructions for Claude Code

## What this project is
Staff recruitment and compliance web app for Primeway Care, a UK domiciliary care company. It keeps carers CQC-ready (Regulations 18 and 19) and holds special category data (health, DBS) under UK GDPR. Separate from CareHub, Primeway's client system.

- **Source of truth:** `docs/specs/staffhub-spec-v3-ai.md` (build spec). Human version: `docs/specs/staffhub-spec-v3.md`.
- If the code, the docs and the spec disagree, **stop and ask** — do not pick one silently.
- Build **Phase 1 only** unless told otherwise. Items marked TBC in the spec are open: ask, don't decide.

## Who you are working with
Zak builds this alone with you. He is a self-taught TypeScript/Next.js developer: strong on frontend, knows backend concepts but not in depth. He must be able to explain every change before it is committed.
- Before writing code, explain the approach and how it works in 2–3 short paragraphs, then give complete, working code.
- Explain backend, database, RLS and security decisions in plain words: what, why, and what would go wrong otherwise.
- Keep changes small and reviewable (one 2–3 hour session or less).
- Stay in the React/Next.js ecosystem. Do not add new languages or frameworks. Do not use React Query, Drizzle, AWS, Vercel, analytics or third-party scripts.
- British English, sentence case in UI text and docs.

## How every task runs
1. **Plan mode first** for anything bigger than a one-line fix.
2. **Interview before planning:** restate the goal, list your assumptions, ask up to 5 questions about real unknowns, and flag security or privacy risks.
3. **Plan contents:** files to change, tables and RLS policies affected, tests to add, docs to update, and which features are affected (check `docs/features/README.md`).
4. Implement only after Zak approves the plan. Tests ship with the change, never later.
5. Before saying "done": `npm run lint`, `npm run typecheck`, `npm test` all pass (commands are listed below once set up).
6. **Use the agents:**
   - `architect` — before any feature, structural change, new library or cross-feature change.
   - `db-rls-reviewer` — after any change to `prisma/`, `supabase/`, `src/lib/db/` or anything touching tables, policies, grants or triggers.
   - `security-reviewer` — before every pull request, and after changes to auth, server actions, route handlers, links, uploads, storage, email or config.
7. Update the docs (rules below). Record any decision with trade-offs as an ADR in `docs/decisions/`.
8. **End of session:** append a short entry to `docs/progress.md`: done, next, open questions.

## Git rules
- **Never commit or push to `main`.** Work on a branch: `feat/…`, `fix/…`, `chore/…`, `docs/…`, `test/…`.
- Commit or push only when Zak asks. Conventional commit messages (`feat: …`, `fix: …`). One logical change per commit.
- Changes reach `main` only through a pull request using `.github/pull_request_template.md`.
- Never force-push, never skip hooks (`--no-verify`), never rewrite `main`'s history.

## Architecture rules (do not break)
- The web app reaches the database **only through `withUser()`** (`src/lib/db/with-user.ts`), which runs each query as the signed-in user so RLS applies. The privileged Prisma client is for the worker and scripts only; ESLint blocks it elsewhere.
- Business rules (stage dependencies, date rules, gates, compliance status, permissions) live in plain TypeScript in `src/lib/domain/` and `src/lib/permissions.ts`, with unit tests. No business rules inside pages or components.
- Every server action and route handler: verify the session on the server, require MFA (`aal2`), confirm the user's assignment to the current branch, call `can()`, validate input with Zod. Server actions are public endpoints — treat them that way.
- The browser never talks to the database or storage directly. Supabase's Data API stays off for app tables.
- Every table: RLS enabled, soft delete (`deleted_at`, `deleted_by`), audit trigger. No hard deletes from the app.
- Date rules are shared Zod schemas used by both forms and server.
- Compliance status is computed when read, never stored.

## Security and privacy rules
- Never read, print or commit `.env` files, keys or tokens. The service-role key and privileged database URL exist only in the worker's environment.
- **Synthetic data only.** No real names, NI numbers, dates of birth, addresses or documents in code, seeds, tests, fixtures, logs, screenshots or prompts.
- Never log personal data. Sentry runs with PII scrubbing.
- Validate everything on the server; never trust the client.
- Uploads: quarantine → virus scan → staff review. Signed URLs live 60 seconds or less. Every sensitive view or download is audited.
- Agreement tick boxes are never pre-ticked; records store who, what, when and where and are insert-only.
- If you notice a security or privacy problem anywhere — even outside the current task — say so straight away.

## Project layout (target — confirm in the architecture ADRs)
```
src/app/            routes, layouts, server actions
src/components/     UI (shadcn/ui based)
src/lib/domain/     business rules (pure TS, unit-tested)
src/lib/db/         withUser(), privileged client (restricted)
src/lib/validation/ shared Zod schemas
src/lib/permissions.ts
worker/             background service (pg-boss, ClamAV, backups)
prisma/             Prisma schema
supabase/           local config, SQL migrations (RLS, triggers, grants)
tests/              Playwright end-to-end tests
docs/               see docs/README.md
```

## Commands
To be filled in during milestone M1. Keep this list accurate.
- `npm run dev` —
- `npm run lint` —
- `npm run typecheck` —
- `npm test` —
- `npm run test:e2e` —

## Docs rules
- Every feature has `docs/features/<feature>.md` (copy `_TEMPLATE.md`). Update it in the same change as the code, including its **Impact if changed** section.
- `docs/features/README.md` holds the feature list and dependency map. Update it when dependencies change.
- Architecture → `docs/architecture/`, decisions → `docs/decisions/`, testing → `docs/testing/`, running and maintaining the system → `docs/maintainer-guide/`, security → `docs/security/`.
- Document environment variable **names** only, never values.
