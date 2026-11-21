# F#: Feature name

| Phase | Status | Spec sections | Last updated |
| --- | --- | --- | --- |
| 1 | planned | §… | YYYY-MM-DD |

## Purpose
What this feature does and why Primeway needs it (CQC or GDPR reason if any).

## Who can do what
| Action | Admin | Branch Manager | HR |
| --- | --- | --- | --- |
| … | | | |
Permission keys used (`src/lib/permissions.ts`): …

## How it works
The main flows, step by step. Add a Mermaid sequence diagram for anything with more than three steps.

## Data
| Table | Purpose | Sensitive? | RLS policies |
| --- | --- | --- | --- |
| … | | | |
Relationships, enums and soft-delete notes.

## Code map
- Routes and pages: `src/app/...`
- Server actions / route handlers: …
- Domain rules: `src/lib/domain/...`
- Validation schemas: `src/lib/validation/...`
- Components: …
- Worker jobs: …
- Migrations: …

## Rules and validation
Business rules, date rules, error messages shown to users.

## Security and privacy
Threats specific to this feature and how they are handled. What is audited.

## Tests
| Type | File | What it proves |
| --- | --- | --- |
| Unit | | |
| RLS | | |
| End-to-end | | |
How to run just these tests: …

## Dependencies
- **Depends on:** F… (what it needs from each)
- **Used by:** F… (what they rely on from this feature)

## Impact if changed
"If you change X, also check Y." Be specific — tables, functions, screens and tests that would break.

## Maintainer notes
Configuration, environment variable names, constants, known issues, how to support users with this feature.

## Change log
| Date | Change | PR |
| --- | --- | --- |
