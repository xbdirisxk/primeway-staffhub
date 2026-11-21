# StaffHub documentation

Everything someone needs to understand, change, test and run Primeway StaffHub. Keep these docs in step with the code: a change is not finished until its docs are updated.

| Folder | What lives there | Updated when |
| --- | --- | --- |
| `specs/` | The specification (human and AI versions) and the design system files | Requirements change (new spec version) |
| `architecture/` | How the system fits together: overview, data model, RLS matrix, key flows | The structure, tables, policies or flows change |
| `decisions/` | Architecture decision records (ADRs): one file per decision with trade-offs | A decision is made or reversed |
| `features/` | One doc per feature, plus the feature list and dependency map | The feature's code, data, rules or dependencies change |
| `testing/` | Test strategy, how to run tests, test data rules | Test tooling or approach changes |
| `security/` | Threat model and the pull request security checklist | A new risk or control appears |
| `maintainer-guide/` | Setting up, configuring, deploying, backing up, restoring and supporting the system | Anything operational changes |
| `progress.md` | Session log: done, next, open questions | End of every session |

## Rules
- British English, sentence case, plain language. Explain why, not just what.
- Never write secrets, keys, real personal data or `.env` values in docs. Name environment variables only.
- Diagrams in Mermaid so they live in git and render on GitHub.
- When you change a feature, update its **Impact if changed** section and check every feature listed under **Used by**.
