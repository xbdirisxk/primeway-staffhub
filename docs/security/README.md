# Security and privacy

Full rules: spec sections 7, 10, 10a, 13 and `CLAUDE.md`. The `security-reviewer` agent and the security-guidance plugin check against these.

## What we protect
Staff identity documents, Right to Work, DBS and health data (UK GDPR special category), applicant data arriving through public links, the audit trail CQC relies on, and branch separation.

## Main threats and controls
| Threat | Control |
| --- | --- |
| One branch sees another's data | Server-side branch checks + RLS on every table (two layers, ADR-001); RLS tests |
| Staff exceed their role | `can()` on every action; role checks in RLS for sensitive tables |
| Guessing record ids (IDOR) | All lookups through `withUser()` inside branch scope |
| Stolen password | Mandatory TOTP 2FA, idle and absolute timeouts |
| Link brute force or sharing | Long hashed tokens, DOB check, lockout, rate limits, short expiry, link-bound cookie |
| Malicious upload | Type by magic bytes, size limit, quarantine, ClamAV scan, no inline rendering |
| Leaks through logs, email or monitoring | No personal data in logs, emails or Sentry; scrubbing on |
| Leaked secrets | Secrets only in DigitalOcean env; service-role key only in the worker; `.env` never read by Claude Code or committed |
| Tampering with evidence | Append-only audit log; insert-only agreements; signed copies cannot be replaced |
| Lost files | Daily database backups; weekly file backups to Spaces; tested restores |

## If something goes wrong
A personal data breach that risks people's rights must be reported to the ICO within 72 hours of becoming aware of it. The incident steps live in `docs/maintainer-guide/` (to write before go-live).
