# Testing

## What we test, and where
| Layer | Tool | Location | Must cover |
| --- | --- | --- | --- |
| Domain rules | Vitest | `src/lib/domain/**/*.test.ts` | Every stage dependency, date rule (pass, fail, override), gate requirement and compliance status case — aim for 100% |
| Permissions | Vitest | `src/lib/permissions.test.ts` | Every action key × every role |
| Database and RLS | Vitest against local Supabase | `tests/rls/` | Per table: cross-branch read/write denied, spoofed branch denied, soft-deleted rows hidden, forbidden operations fail; audit log immutable; sensitive tables per role |
| End-to-end | Playwright | `tests/e2e/` | Each milestone's "done when" test; applicant pages at a phone viewport |

## Rules
- **Synthetic data only.** Use obviously fake names and numbers. Never copy real staff data into tests, fixtures or screenshots.
- Tests ship in the same pull request as the code.
- A bug fix starts with a failing test that reproduces it.
- Never weaken or delete a failing test to make a build pass; fix the code or discuss.

## How to run
To be filled in during milestone M1 (commands, local Supabase start and reset, seeding).
