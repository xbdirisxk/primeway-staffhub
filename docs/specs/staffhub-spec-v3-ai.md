# Primeway StaffHub — Build Spec v3 (AI version)

> Source of truth for Claude Code. Derived from "Primeway StaffHub – Developer Specification v3" (Word, 20 Nov 2026). If this file and the Word spec disagree, stop and ask Zak.
> Builder: Zak Osman (solo, with Claude Code). Keep changes small and explainable; Zak must understand every change before committing.

## 0. How to use this file

- `MUST` = required. `SHOULD` = default unless Zak decides otherwise. `TBC` = open item; do not invent an answer, use the stated default and leave a `// TBC:` comment.
- Build **Phase 1 only** unless told otherwise. Phase 2/3 items are listed so the schema leaves room for them; do not build them.
- Security is not a later step. Every Phase 1 milestone ships with its permission checks, RLS policies, audit coverage and tests.
- Use synthetic data only. Never put real names, NI numbers or documents in seeds, fixtures, tests or logs.
- British English, sentence case in all UI text.

## 1. Product summary

Web app for Primeway Care (UK domiciliary care) that tracks every carer from application to first independent shift, enforces CQC Regulation 19 checks through two approval gates, and keeps evidence of who did each check and when. Separate from CareHub (Primeway's client system). Single company, multiple fully separated branches.

## 2. Phases

### Phase 1 — Foundation and recruitment (beta, one pilot branch, real data)
P1 scope:
1. Project setup, design system, Sentry (EU), CI with tests.
2. Schema, RLS on every table, soft delete columns, audit triggers.
3. Supabase Auth: email + password, mandatory TOTP 2FA, idle timeout, invites.
4. Branches, 3 roles, user–branch assignments, branch switcher.
5. Worker profile, document types (Admin-managed), staff document uploads.
6. 16 stages: hard dependencies, stage states and colours, target times.
7. Date rules (shared Zod) + override with reason.
8. Simple induction, training, shadowing, competency records (stage evidence).
9. Gate 1 and Gate 2 with "Why blocked?"; compliance status computed on read.
10. Secure links (application + manual document request), full link security, Pending review queue, transactional email.
11. Worker service: pg-boss, ClamAV scan of quarantine uploads, link expiry, weekly Storage → Spaces backup.
12. Online agreement (tick + dropdown) and scanned paper signatures (section 10a). E-signatures are Phase 3.
13. Worker history tab (from audit log), basic branch dashboard, Admin-only branch transfer.
14. Go-live prerequisites (non-code, track them): DPIA, applicant privacy notice, retention periods.

### Phase 2 — Ongoing compliance (after beta hiring) — DO NOT BUILD YET
Renewals + expiry reminders with automatic document request links; training matrix; supervisions, appraisals, spot checks, medication observations; confidential HR notes; leavers; CQC worker file together with its one-click PDF, and other PDFs generated from data; audit log search UI; overrides report; full dashboard tiles; CSV exports; bin UI, restore, 30-day purge; 3 fixed templates (offer letter, contract, reference request) filled from worker data and printed for paper signature; migration of existing staff.

### Phase 3 — After completion — DO NOT BUILD YET
AI (MCP connector, in-app assistant, scheduled AI tasks); built-in e-signatures (section 10a, paper route stays); smart date reading; DBS certificate scan with auto-deletion after Gate 2; SMS fallback; daily manager summary email; template editor; settings UI for link expiry, lockout limit, reminder periods.

## 3. Stack (fixed)

| Concern | Choice |
| --- | --- |
| Language | TypeScript (strict) |
| Web | Next.js App Router, server components + server actions, on DigitalOcean App Platform (LON) |
| DB access | Prisma, via `withUser()` only (section 7) |
| DB | Supabase Postgres, eu-west-2, Pro plan |
| Auth | Supabase Auth + `@supabase/ssr` (cookie sessions), TOTP MFA |
| Files | Supabase Storage, private buckets `quarantine` and `documents` |
| Background | Node.js worker (DO App Platform worker component) + pg-boss + ClamAV (clamd) |
| Backup | DigitalOcean Spaces (LON), private bucket |
| Email | Provider with UK/EU processing + DPA (TBC); wrap behind `lib/email` interface |
| UI | Tailwind, shadcn/ui, tokens from the prototype's CSS (`docs/specs/design/prototype/index.html`, blue accent) — section 14 |
| Forms / validation | React Hook Form + Zod (schemas shared by client and server) |
| PDFs | pdf-lib (generated PDFs in Phase 2; signature stamping in Phase 3) |
| Tables | TanStack Table |
| Tests | Vitest (domain rules, permissions, RLS), Playwright (E2E) |
| Monitoring | Sentry EU region, `sendDefaultPii: false`, scrub request bodies |

Do not add: React Query, Drizzle, AWS services, Vercel, analytics or any third-party script.

## 4. Roles and permissions

Roles: `ADMIN`, `BRANCH_MANAGER`, `HR`. Store roles as data; implement checks only through `lib/permissions.ts` (`can(user, action, context)`), never inline `role ===` checks. The schema must allow adding `RECRUITMENT` and `READ_ONLY` later without migration of existing data.

| Action key | ADMIN | BRANCH_MANAGER | HR |
| --- | --- | --- | --- |
| `branch.access` | all branches | assigned (one) | assigned (one or more) |
| `branch.create` | ✓ | ✗ | ✗ |
| `worker.read` / `worker.write` | ✓ | ✓ | ✓ |
| `stage.complete` | ✓ | ✓ | ✓ |
| `gate.approve` | ✓ | ✓ | ✓ |
| `daterule.override` | ✓ | ✓ | ✓ |
| `link.send` / `link.cancel` | ✓ | ✓ | ✓ |
| `agreement.record` (record paper signature) | ✓ | ✓ | ✓ |
| `review.decide` (approve/reject uploads) | ✓ | ✓ | ✓ |
| `sensitive.read` (health, DBS, equal-opps aggregate) | ✓ | ✓ | ✓ |
| `user.manage` | all branches | own branch | ✗ |
| `worker.transfer` | ✓ | ✗ | ✗ |
| `settings.manage` (courses, document types) | ✓ | ✗ | ✗ |
| `worker.history.read` | ✓ | ✓ | ✓ |
| `record.delete` (soft) | ✓ | ✓ | ✓ |

Phase 2 keys (reserve names, do not implement): `audit.search` (ADMIN), `bin.restore`, `bin.purge` (ADMIN), `export.csv`, `pdf.generate`. Phase 3: `signature.send`, `signature.cancel`.

Every server action / route handler MUST: (1) verify session with Supabase server-side (`getUser`/`getClaims`, never trust the cookie payload alone); (2) require MFA level `aal2`; (3) resolve current branch and confirm the user is assigned to it; (4) call `can()`; (5) run DB work inside `withUser()`.

## 5. Branch model

- Every branch-scoped table has `branch_id uuid not null references branch(id)`.
- Current branch is chosen in the branch switcher, stored server-side in the session (signed, http-only cookie), re-validated against assignments on every request.
- No cross-branch views, ever. Admin switches branch to see another branch.
- Courses and document types: `branch_id null` = shared; non-null = branch-specific. Admin only.
- Transfer (ADMIN only): one transaction that updates `worker.branch_id` and all dependent rows' `branch_id`, inserts `worker_transfer` (from, to, reason, by, at), sets `worker.transfer_review_pending = true`. Receiving BRANCH_MANAGER or HR clears it via "Mark reviewed" (logged).

## 6. Domain rules (pure TypeScript in `lib/domain/`, 100% unit-tested)

### 6.1 Stages
```
1  APPLICATION_RECEIVED
2  INTERVIEW_COMPLETED
3  CONDITIONAL_OFFER
4  REFERENCES_REQUESTED
5  REFERENCES_APPROVED
6  IDENTITY_VERIFIED
7  RIGHT_TO_WORK_CHECKED
8  DBS_SUBMITTED
9  DBS_RECEIVED_REVIEWED
10 CONTRACT_COMPLETED          (completes when a scanned paper-signed contract is uploaded and approved)
11 CORPORATE_INDUCTION
12 MANDATORY_TRAINING
13 SHADOWING_COMPLETED
14 COMPETENCY_ASSESSED
15 GATE_2_APPROVED             (set only by gate approval, never by "complete stage")
16 FIRST_INDEPENDENT_SHIFT     (date entered manually)
```

### 6.2 Hard dependencies (the only ordering enforced)
```ts
const HARD_DEPS: Record<Stage, Requirement[]> = {
  INTERVIEW_COMPLETED:      [stage('APPLICATION_RECEIVED')],
  CONDITIONAL_OFFER:        [stage('INTERVIEW_COMPLETED')],
  REFERENCES_APPROVED:      [stage('REFERENCES_REQUESTED')],
  DBS_RECEIVED_REVIEWED:    [stage('DBS_SUBMITTED')],
  CONTRACT_COMPLETED:       [stage('CONDITIONAL_OFFER')],
  SHADOWING_COMPLETED:      [gate('GATE_1')],
  COMPETENCY_ASSESSED:      [trainingFor('related course')], // per competency type
  GATE_2_APPROVED:          [gate2Requirements()],
  FIRST_INDEPENDENT_SHIFT:  [gate('GATE_2')],
};
```
All other stages can be started and completed in any order. Enforce on the server; the UI only mirrors it.

### 6.3 Stage state (derived) and colour
| State | Colour | Rule |
| --- | --- | --- |
| `LOCKED` | locked style | any hard dep unmet |
| `NOT_STARTED` | grey | no data |
| `IN_PROGRESS` | amber | started or waiting on third party, within target time |
| `OVERDUE` | red | started and `now > started_at + TARGET_DAYS[stage]` |
| `FAILED` | red | rejected reference, DBS outcome requiring review, competency `NOT_COMPETENT` |
| `COMPLETE` | green | completed with evidence |

`TARGET_DAYS` constants (TBC values): references 14, DBS 28, others null (never overdue). Keep red for action-needed only.

### 6.4 Gates
- **Gate 1 – Approved for shadowing** (TBC: stages 1–12 complete). Also TBC: allow DBS Adult First instead of stage 9 — implement as a disabled flag `ALLOW_DBS_ADULT_FIRST_FOR_GATE1 = false`.
- **Gate 2 – Approved for independent work**: Gate 1 approved; stage 13 complete with decision `COMPLETE`; `COMPETENT` outcome for every service in the approval; every mandatory document type approved (none pending review); no expired document; no `FAILED` stage.
- `evaluateGate(worker, gate) => { ok: boolean; missing: { item: string; owner: 'office' | 'applicant' | 'referee' | 'dbs'; link?: string }[] }` powers both the lock and the "Why blocked?" panel.
- Approval record: approver, role, branch, date, comments, restrictions, services competent to provide. Gate requirements cannot be overridden.
- TBC (separation of duties): may the user who completed a check approve the gate? Default: allowed, but record both.

### 6.5 Compliance status (computed on read, never stored)
Order of evaluation:
1. `employment_status = LEAVER` → `LEAVER` (grey)
2. any expired document / failed check / missing renewal → `BLOCKED` (red)
3. Gate 2 approved → `INDEPENDENT` (green)
4. Gate 1 approved → `SHADOWING` (blue-teal #0A6A8C on #DDEFF7)
5. else → `IN_PROGRESS` (amber)

Implement once in TS (`computeComplianceStatus`) and once as a SQL view/function used for list filtering; a test MUST assert both agree on the same fixtures. Employment status (`ACTIVE | INACTIVE | SUSPENDED | LEAVER`) is stored separately.

### 6.6 Date rules (shared Zod, used by forms AND server)
| Rule id | Must be true |
| --- | --- |
| `INTERVIEW_AFTER_APPLICATION` | interview ≥ application |
| `OFFER_AFTER_INTERVIEW` | offer ≥ interview |
| `REF_RECEIVED_AFTER_REQUESTED` | reference received ≥ requested |
| `DBS_CERT_AFTER_APPLICATION` | new DBS: certificate ≥ DBS application |
| `DBS_UPDATE_CHECK_AFTER_APPLICATION` | Update Service: check date ≥ application (certificate may be older) |
| `INDUCTION_AFTER_OFFER` | induction > offer |
| `SHADOWING_AFTER_INDUCTION_TRAINING` | shadowing > induction and > mandatory training |
| `COMPETENCY_AFTER_TRAINING` | competency > related training |
| `FIRST_SHIFT_AFTER_ALL` | first shift > Gate 2 and > DBS, RTW, references, induction, training, shadowing, competency |
| `EXPIRY_AFTER_ISSUE` | any document expiry > issue |
| `NEXT_REVIEW_AFTER_LAST` | next review > last assessment |

Rules apply only when both dates exist. Override: any role, mandatory reason (min 10 chars), inserts `date_override` (rule id, field, original value, reason, user, at), shows "Overridden" badge. Correcting a typo is a normal edit (audited), not an override.

## 7. Database access and RLS (critical)

Prisma connects as a privileged role and bypasses RLS. Therefore:

- The web app MUST access the DB only through `withUser(ctx, fn)`:
```ts
// lib/db/with-user.ts (shape, not final code)
export async function withUser<T>(ctx: RequestCtx, fn: (tx: Prisma.TransactionClient) => Promise<T>) {
  return prisma.$transaction(async (tx) => {
    await tx.$executeRaw`select set_config('request.jwt.claims', ${JSON.stringify(ctx.claims)}, true)`;
    await tx.$executeRaw`select set_config('app.branch_id', ${ctx.branchId}, true)`;
    await tx.$executeRaw`select set_config('app.client_ip', ${ctx.ip}, true)`;
    await tx.$executeRaw`set local role authenticated`;
    return fn(tx);
  });
}
```
  `ctx.claims` come only from server-verified Supabase auth. All `set_config` use `is_local = true` (transaction scope) so nothing leaks between pooled connections.
- Export the raw `prisma` client only from `lib/db/privileged.ts`, imported only by the worker and scripts. Add an ESLint `no-restricted-imports` rule blocking it in `app/` and `lib/` (except `lib/db/with-user.ts`).
- Connection: Supabase pooler (transaction mode) with `pgbouncer=true` for the app; direct connection for migrations.
- RLS, policies, triggers, grants and views live in SQL migrations (Prisma migrate with hand-written SQL). Prisma schema mirrors tables only.
- Do not expose app tables through Supabase's Data API: disable the Data API or keep tables in schema `app` that is not in exposed schemas; revoke all on app tables from `anon`. Grant only what `authenticated` needs.
- Policy helpers (SQL, `security definer`, `stable`, `search_path` pinned):
  - `app.current_user_id()` → `auth.uid()`
  - `app.current_branch_id()` → `current_setting('app.branch_id', true)::uuid`
  - `app.has_branch_access(b uuid)` → user is ADMIN, or has an assignment to `b`
  - `app.has_role(r text[])`
- Standard policy for branch tables: `branch_id = app.current_branch_id() and app.has_branch_access(branch_id) and deleted_at is null` (select/update). Insert `with check` the same. No hard `delete` grant to `authenticated` (soft delete only).
- Sensitive tables (`health_declaration`, `dbs_record`, `equal_opportunities`, later `confidential_note`) have their own policies using `app.has_role(...)`, even though all three roles pass today — this is what lets a Recruitment role be added later.
- Tests (Vitest against a local Supabase): for each table prove (a) user in branch A cannot read/write branch B, (b) unassigned branch denied even if `app.branch_id` is spoofed, (c) soft-deleted rows invisible, (d) `audit_log` cannot be updated/deleted by `authenticated`.

## 8. Schema outline (Phase 1)

Conventions: `uuid` PK default `gen_random_uuid()`; `created_at/created_by/updated_at/updated_by`; soft delete `deleted_at timestamptz null, deleted_by uuid null` on every deletable table; unique constraints as partial indexes `where deleted_at is null`; enums as Postgres enums; all timestamps `timestamptz`, dates `date`; UK date display `dd MMM yyyy`.

| Table | Key columns / notes |
| --- | --- |
| `branch` | name, code, active |
| `staff_user` | id = `auth.users.id`, full_name, email, active, invited_by |
| `user_branch_role` | user_id, branch_id (null for ADMIN = all), role enum |
| `worker` | branch_id, employee_no, names, dob, ni_number, contact, address, emergency contact, job_title, employment_type, hours, line_manager_id, start_date, first_independent_shift_date, employment_status, transfer_review_pending, entered_by_staff (bool), source (`LINK` / `STAFF`) |
| `worker_stage` | worker_id, stage enum, started_at, completed_at, completed_by, status_detail, evidence notes; unique (worker_id, stage) |
| `document_type` | branch_id null=shared, name, requires_issue_date, requires_expiry_date, reminder_days int[], mandatory_for_gate2 |
| `document` | worker_id, document_type_id, storage_path, issue_date, expiry_date, number, review_status (`PENDING_SCAN`/`PENDING_REVIEW`/`APPROVED`/`REJECTED`/`INFECTED`), reviewed_by, reviewed_at, reject_reason, source |
| `reference` | worker_id, referee details, requested_at, received_at, outcome, approved_by |
| `right_to_work` | worker_id, visa_type, sponsor, expiry_date, check_date, follow_up_dates |
| `dbs_record` (sensitive) | worker_id, certificate_no, issue_date, level, outcome, update_service bool, update_check_date, application_date, checked_by, checked_at — **no file in Phase 1** |
| `health_declaration` (sensitive) | worker_id, answers jsonb, submitted_at |
| `equal_opportunities` (sensitive) | applicant-optional answers; never joined into profile queries; aggregate reporting only |
| `application_form` | worker_id, data jsonb (Zod-validated), employment_history with gap explanations, submitted_at |
| `training_course` | branch_id null=shared, name, validity_months, active |
| `training_record` | worker_id, course_id, completed_on, expires_on, provider, method, certificate_document_id |
| `induction_record` | worker_id, modules, trainer, completed_on, test_score |
| `shadowing_record` | worker_id, shift dates, supervisor, hours, feedback, decision (`COMPLETE`/`MORE_NEEDED`) |
| `competency_assessment` | worker_id, type, assessor, assessed_on, outcome (`COMPETENT`/`PARTLY`/`NOT_COMPETENT`), comments, actions, reassess_on, evidence_document_id |
| `gate_approval` | worker_id, gate (`GATE_1`/`GATE_2`), approved_by, role, approved_at, comments, restrictions, services text[]; revoked_at/by/reason |
| `date_override` | worker_id, rule_id, field, original_value, reason, by, at |
| `secure_link` | branch_id, worker_id, purpose (`APPLICATION`/`DOCUMENT_REQUEST`; `SIGNATURE` reserved for Phase 3), token_hash (sha-256, unique), requested_document_type_ids, expires_at, status (`ACTIVE`/`LOCKED`/`EXPIRED`/`CANCELLED`/`COMPLETED`), failed_attempts, sent_by, sent_at, last_opened_at |
| `link_attempt` | link_id, ip_hash, success, at (for rate limiting and lockout evidence) |
| `worker_transfer` | worker_id, from_branch, to_branch, reason, by, at, reviewed_by, reviewed_at |
| `notification` | branch_id, user_id null=all branch staff, type, payload (no sensitive data), read_at |
| `audit_log` | id bigserial, at, actor_user_id, actor_type (`STAFF`/`APPLICANT_LINK`/`WORKER_JOB`), branch_id, ip, action, table_name, row_id, old jsonb, new jsonb — append-only |

| `agreement` | branch_id, worker_id, kind (`APPLICATION_DECLARATION`/`POLICY_ACK`/`OFFER_LETTER`/`CONTRACT`), method `signature_method`, agreed_document_id (immutable copy of what was agreed), statement_version, statement_sha256, agreed_full_name, link_id, agreed_at, ip, user_agent, signing_preference, document_id (paper scan), recorded_by — insert-only |

Phase 2 placeholders (do not create yet): supervision, appraisal, spot_check, confidential_note, leaver, template. Phase 3: signature_request.

## 9. Audit trail

- Generic trigger `app.audit_row()` on every table except `audit_log`: insert/update/delete → one row with old/new JSON, `actor_user_id = auth.uid()`, `branch_id`, `ip = current_setting('app.client_ip', true)`. Soft delete appears as update with `deleted_at` set → log action `SOFT_DELETE`.
- Redact in audit JSON: `token_hash`, and never store file contents.
- App-level events written explicitly: `LOGIN`, `LOGIN_FAILED`, `MFA_ENROLLED`, `LINK_SENT`, `LINK_OPENED`, `LINK_DOB_FAILED`, `LINK_LOCKED`, `DOCUMENT_VIEWED`, `DOCUMENT_DOWNLOADED`, `FILE_INFECTED`, `GATE_APPROVED`, `OVERRIDE`.
- Protection: `audit_log` owned by a dedicated role; `revoke update, delete, truncate` from everyone else; insert only via the security-definer trigger/function. A test proves `authenticated` cannot modify it.
- Phase 1 UI: "History" tab on the worker profile (filtered by row/worker). Admin log search is Phase 2.

## 10. Secure links (Phase 1, all items MUST)

Flow: staff create link (name, DOB, phone, email required) → email sent → applicant opens `/l/[token]` → DOB check → short session → form/uploads → submit → Pending review.

- Token: 32 random bytes (`crypto.randomBytes`), base64url; store only `sha256(token)`; one link = one person + one purpose.
- Expiry: default 3 days (`LINK_EXPIRY_DAYS = 3`); sender may choose another; resend issues a NEW token and invalidates the old; cancel supported.
- DOB gate: compare in constant time; `MAX_DOB_ATTEMPTS = 5` → status `LOCKED`, notify branch staff, audit. Rate limit DOB attempts and uploads per link and per IP (Postgres-backed counter; no Redis).
- After DOB success: set `__Host-` prefixed, `Secure`, `HttpOnly`, `SameSite=Strict` cookie bound to link id, 30-minute sliding lifetime, never beyond link expiry.
- Scope: link session can only read/write its own draft `application_form` and upload documents for its own worker. Never render any official-record data back.
- Link pages: mobile-first; no third-party scripts; `Referrer-Policy: no-referrer`; `X-Robots-Tag: noindex`; strict CSP; show Primeway applicant privacy notice before first input; equal-opps section optional and clearly separate.
- Uploads: allowed `application/pdf`, `image/jpeg`, `image/png` (HEIC TBC); max 10 MB per file; verify magic bytes server-side; random storage path chosen by server; upload into `quarantine` bucket via server-issued signed upload URL scoped to that exact path; row created with `PENDING_SCAN`.
- Worker scans with clamd → clean: move to `documents` bucket, `PENDING_REVIEW`; infected: delete object, `INFECTED`, notify staff, audit. Staff UI MUST NOT offer view/download for anything not scanned clean.
- Staff view/download: server generates signed URL (≤ 60 s), audit `DOCUMENT_VIEWED`/`DOCUMENT_DOWNLOADED`, `Content-Disposition` set, never inline-render untrusted HTML/SVG.
- Email body: first name + link only; no other personal data.
- Staff entry path: same Zod schemas, same review rules, `source = STAFF`, `entered_by` recorded.

## 10a. Agreements and signatures

### Phase 1 (build now)
- **Online agreement:** end of application form and policy acknowledgements. Must meet UK law (clear affirmative action):
  - Checkbox "I have read and agree" MUST render unchecked (`defaultChecked` never true; no saved-draft restoring a tick). Dropdown `signing_preference` (`AGREE_ONLINE` | `PAPER_AT_OFFICE`; labels TBC) MUST have no default (placeholder "Choose…"). Server-side Zod rejects submit unless `agreed === true` and a preference is chosen — never trust the client.
  - Full document/wording rendered on the page before the checkbox (not only a link).
  - Store in `agreement`: **who** = agreed_full_name (typed), worker_id, link_id (DOB-verified session) or staff user; **what** = `agreed_document_id` (immutable stored copy of the exact text/PDF shown, in `documents` bucket) + statement_version + statement_sha256; **when** = agreed_at (`timestamptz`, server clock, display Europe/London); **where** = ip (from trusted proxy header, validated) + user_agent.
  - `agreement` rows are insert-only: no update/delete grant for `authenticated`; changed wording = new version = new agreement row.
  - Not the UK GDPR lawful basis for processing (basis = contract/legal obligation); do not wire it to data deletion.
- **Paper signature:** offer letter and contract signed on paper; staff upload the scan as a `document` (types `OFFER_LETTER_SIGNED`, `CONTRACT_SIGNED`) → quarantine → scan → Pending review → approve. Insert `agreement` row with method `PAPER_SCANNED`, `document_id`, `recorded_by`.
- **Method everywhere:** every signature-dependent record references an `agreement` row; UI shows the method badge (Online agreement / Paper, scanned). Enum `signature_method` = `ONLINE_AGREEMENT` | `PAPER_SCANNED` | `E_SIGNATURE` (last value reserved for Phase 3; not reachable in Phase 1 code).
- Paper route is permanent (keep it after Phase 3).
- Stage 10 completes automatically when an approved `CONTRACT_SIGNED` document exists.
- Signed copies immutable: no update grant on their `document` rows' file fields; storage path never overwritten; corrections = new upload, old kept; soft delete ADMIN only.
- Log: `AGREEMENT_RECORDED`, plus normal upload/review events.

### Phase 3 (do not build): built-in e-signatures
Behind `lib/esign` (fallback TBC: low-cost paid provider with UK/EU processing + DPA). `secure_link` purpose `SIGNATURE`; DOB gate; scroll-to-end + consent; drawn/typed/agree; server re-checks `source_sha256`, stamps with pdf-lib, appends a signature record page, stores `signed_sha256`; immutable output; creates `agreement` with method `E_SIGNATURE`.

## 11. Pending review queue

Per branch. Items: submitted applications and `PENDING_REVIEW` documents. Actions: approve (requires dates where the document type needs them, validated by date rules) or reject (reason required; optional "send new link"). Approval updates profile, stage evidence and therefore computed status.

## 12. Worker service (Phase 1 jobs)

| Job | Schedule |
| --- | --- |
| `scan-upload` | on enqueue after each upload |
| `expire-links` | every 15 min |
| `backup-files` | weekly (Sun 02:00 Europe/London); copy both buckets to Spaces; keep last 4 copies (TBC) |

pg-boss in the same Postgres; singleton scheduling. The worker holds the service-role key and privileged DB URL; the web app holds neither. ClamAV needs ~1.5–2 GB RAM; size the worker instance accordingly and keep `freshclam` updates running.

## 13. Auth and session security

- Invite-only accounts (Admin / Branch Manager invites). No public sign-up (disable in Supabase).
- Password: min 12 chars, breached-password check if available.
- TOTP MFA enrolment forced on first login; every protected route requires `aal2`.
- Idle timeout 30 min (`IDLE_TIMEOUT_MINUTES`), absolute session 12 h (TBC).
- Security headers on all routes: CSP (nonce-based), HSTS, `X-Frame-Options: DENY` / `frame-ancestors 'none'`, `Referrer-Policy: strict-origin-when-cross-origin` (no-referrer on link pages), `Permissions-Policy` minimal.
- Server actions: validate input with Zod; check origin (Next.js default) — do not disable.
- Secrets only in DO encrypted env vars; `.env*` git-ignored; Dependabot + `npm audit` in CI.
- Sentry: scrub PII, no session replay.

## 14. UI and design

- **Screens and flows:** the prototype in `docs/specs/design/prototype/index.html` (97 screens, aligned with spec v3). Its `README.md` lists which screens are Phase 1, 2 or 3; screens for later phases show a banner. Live copy: https://claude.ai/artifact/3x3RqZTgwYuk9sJT2EtmrY. **If a screen and this spec disagree, the spec wins.**
- **Design system:** `docs/specs/design/design-spec.md`, `design-system.css` and `icons.js` describe components, spacing and type. Where their colours differ from the prototype, use the prototype's.
- Override: accent is **blue** `#2462C8` (hover `#1C4FA4`, tint `#E7EFFC`), neutrals blue-grey; the teal in `design-system.css` is outdated. Status colours: green approved/valid, amber in progress/due, red blocked, grey leaver, blue-teal `#0A6A8C` on `#DDEFF7` for shadowing.
- Sample values pre-filled in prototype forms are illustration only. The agreement tick box and signing-preference dropdown must start empty in the app.
- Staff UI desktop-first (min 1024 px). Link pages mobile-first.
- Translate the prototype CSS into Tailwind theme tokens + shadcn components; do not ship the prototype's SPA mechanics.

## 15. Phase 1 milestones (each = one or more 2–3 hour sessions; end each with a short "done / next" note)

| # | Milestone | Done when |
| --- | --- | --- |
| M1 | Repo + Next.js + Tailwind/shadcn tokens + lint + Vitest + Playwright + CI + Sentry | CI green on an empty app; ESLint rule blocking privileged Prisma import exists |
| M2 | Schema + SQL migrations: tables, enums, RLS, helpers, soft delete, audit trigger; synthetic seed | RLS test suite (section 7) passes; audit immutability test passes |
| M3 | Auth: invite, login, forced TOTP, idle timeout, branch switcher, `withUser`, `can()` | E2E: user without MFA cannot reach any page; HR in branch A cannot open branch B worker by URL |
| M4 | Worker profile + document types (Admin) + staff upload via quarantine (scan stubbed) | Profile CRUD with audit rows; sensitive tables gated by policy |
| M5 | Stages, hard deps, stage states, date rules + override | 100% unit coverage of `lib/domain`; override creates badge + audit row |
| M6 | Induction, training, shadowing, competency records | Stage 11–14 evidence linked; competency rule enforced |
| M7 | Gates + "Why blocked?" + computed compliance status (TS + SQL agree) | Gate 2 impossible via API when any requirement missing (test) |
| M8 | Worker service: pg-boss, ClamAV scan, link expiry, weekly backup | EICAR test file is rejected and never viewable; backup restores a sample file |
| M9 | Secure links + application form + document request + email + Pending review | E2E on mobile viewport: full applicant journey; 5 wrong DOBs lock link; expired/cancelled token refused |
| M10 | Online agreement + paper signature recording (section 10a) | Checkbox renders unticked and dropdown unselected (test); server rejects submit without both; agreement row stores who/what (document copy + hash)/when/where and cannot be updated; approved scanned contract completes stage 10; signed copy cannot be replaced |
| M11 | Basic dashboard, search, history tab, Admin transfer | Transfer moves all rows and sets review flag; history tab shows audit entries |
| M12 | Hardening: security review of every route/action, header check, dependency audit, restore test, DPIA/privacy notice/retention confirmed | Checklist signed off by Zak; pilot branch go-live |

## 16. Open items (TBC — do not decide silently)

1. Gate 1 requirements: stages 1–12 complete?
2. DBS Adult First allowed for Gate 1?
3. Separation of duties: can the same user complete a check and approve the gate?
4. Stage target times (references 14 d, DBS 28 d?).
5. Accept HEIC uploads?
6. Email provider + DPA.
7. Retention periods (incl. unsuccessful applicants).
8. CQC steps for transfers.
9. Phase 2 data migration scope.
10. File backup retention (last 4 weekly copies?) and whether daily is needed.
11. Absolute session length (12 h?).
12. Online agreement: tick box enough for declarations and policy acknowledgements (employment adviser)? Dropdown options? E-signature acceptability for contracts before Phase 3.
