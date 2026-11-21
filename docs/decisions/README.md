# Architecture decision records

One file per decision: `ADR-NNN-short-title.md`, from `ADR-000-template.md`. Never edit an accepted ADR's decision — write a new one that supersedes it.

| ADR | Decision | Status |
| --- | --- | --- |
| [ADR-001](ADR-001-database-access-prisma-with-rls.md) | Database access through Prisma, run as the signed-in user (`withUser()`) | accepted |

Expected next (from the architecture session): migration tooling and where SQL lives; folder structure; local development with the Supabase CLI; how the worker is deployed; email provider.
