# Security guidance for Primeway StaffHub

Context: UK care company staff records — identity, Right to Work, DBS and health data (UK GDPR special category). Multiple branches that must never see each other's data.

- The web app must access the database only through `withUser()` in `src/lib/db/with-user.ts`. Flag any import of the privileged Prisma client (`src/lib/db/privileged.ts`) outside `worker/` and `scripts/`.
- Every server action and route handler must verify the session server-side, require MFA (`aal2`), re-check the user's branch assignment, call `can()` from `src/lib/permissions.ts`, and validate input with Zod before any database call.
- Any record id taken from a URL, form or query must be read inside the user's branch scope. Flag direct lookups by id without branch scoping (IDOR).
- Never log, email, or send to Sentry: names, dates of birth, NI numbers, addresses, document contents, DBS or health details, link tokens.
- Secure link tokens: at least 32 random bytes, store only the SHA-256 hash, compare in constant time (`crypto.timingSafeEqual`), enforce expiry, lockout after 5 wrong DOB attempts, rate limit per link and IP.
- Link-session cookies must use the `__Host-` prefix with `Secure`, `HttpOnly`, `SameSite=Strict`.
- Uploads: verify type by magic bytes, enforce size limits, write only to the `quarantine` bucket at a server-chosen path, never serve a file before the virus scan marks it clean. Signed URLs must expire in 60 seconds or less.
- Agreement checkboxes must never default to checked and the signing-preference dropdown must have no default; the server must reject submissions without an explicit tick and choice. `agreement` and `audit_log` rows are insert-only.
- No `NEXT_PUBLIC_` variable may contain a service-role key, database URL or any secret.
- SQL: Prisma `$queryRaw` / `$executeRaw` only as tagged templates; never `$queryRawUnsafe` with user input.
- SQL migrations: every new table needs RLS enabled, branch-scoped policies, no delete grant to `authenticated`, and the audit trigger.
- Never disable Next.js server action origin checks or CSRF protections. Never use `dangerouslySetInnerHTML`.
- Seeds, tests and fixtures must use synthetic data only.
