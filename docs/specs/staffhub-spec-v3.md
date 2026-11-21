# Primeway StaffHub – Developer Specification v3

Primeway Care Staff Compliance System · 20 Nov 2026 · Zak Osman

This version replaces v2 (9 Nov 2026). It cuts the system down so that one developer, working with Claude Code, can build it in three phases. Security, the database schema and row-level security are complete from Phase 1.

## 1. Purpose and scope

Primeway needs a web system that keeps every carer ready for a CQC inspection. No one works until they are fully checked, and every check holds evidence of who did it, when, and the document.

- **Built for:** Primeway Care only. It is not a multi-company product. It is separate from CareHub, Primeway's client system.
- **Platform:** a web app. Staff screens are built for desktop. The applicant link pages are built for phones first, because carers open them on a mobile browser. There is no native app.
- **Covers:** recruitment and onboarding (16 stages), two approval gates, compliance status, secure applicant links, the audit trail and soft delete in Phase 1. Ongoing compliance, reports and the CQC worker file in Phase 2. AI features in Phase 3.
- **Excludes:** all third-party integrations.
- **Regulation:** supports CQC Regulation 19 (fit and proper persons employed, Schedule 3 checks) and Regulation 18 (staffing: supervision and appraisal). Handles special category data under UK GDPR.

## 2. Changes from v2

| Area | v2 | v3 |
| --- | --- | --- |
| Builder | External developers | **Zak, with Claude Code** |
| Roles | 5: Admin, Branch Admin, HR, Recruitment, Read-only | **3: Admin, Branch Manager, HR.** Recruitment is done by HR at Primeway |
| Platform | Responsive on desktop, tablet and phone | **Desktop for staff; phone-first for applicant links** |
| Stage order | Strict: no stage before the stages it depends on | **Hard dependencies only where one stage needs another's data.** Everything else in any order, shown amber or red until done. Gates stay strict |
| Phases | Phase 1 core, Phase 2 ongoing, Phase 3 AI | **Phase 1** foundation and recruitment (with all security), **Phase 2** after beta hiring, **Phase 3** after completion |
| Audit trail | Phase 1 | **Phase 1** (database triggers). Log search screen in Phase 2 |
| Bin | Phase 1, full screen | **Soft delete in the schema from Phase 1.** Bin screen, restore and 30-day removal in Phase 2 |
| Smart date reading (AI) | Phase 1 | **Phase 3.** Staff type the dates |
| DBS certificate scan | Stored until Gate 2, then deleted | **Phase 3.** Until then only the certificate details are recorded, no scan |
| SMS fallback, daily summary email | Phase 2 | **Phase 3** |
| AI assistant | Phase 3 | **Phase 3** (after the system is complete) |
| Signatures | E-signatures in Phase 2, with the 3 templates | **Phase 1:** online agreement (tick box and dropdown) for application declarations and policy acknowledgements; offer letter and contract signed on paper, scanned and uploaded by staff. **E-signatures move to Phase 3.** The paper route stays permanently |
| CQC worker file | Phase 1 | **Phase 2,** together with its one-click PDF |
| Template editor | HR and above create templates | **Phase 3.** Phase 2 templates are fixed in code |
| Branch transfers | Workflow with CQC steps | **Admin-only action,** logged, with a review flag for the receiving branch |
| Background worker | Phase 1, many jobs | **Phase 1, small:** virus scanning, link expiry, file backup. More jobs in Phase 2 |
| Compliance status | Recalculated by a daily job | **Calculated when read,** from stages and expiry dates, so it is never out of date |
| Settings without code | Link expiry, lockout limit, reminder periods | **Constants in code** in Phase 1. Only courses and document types are stored in the database |
| Data access | Next.js queries Supabase as the user | **Prisma, run as the user inside a transaction,** so row-level security still applies (section 14) |

## 3. Delivery phases

### Phase 1 – Foundation and recruitment (beta)

Goal: Primeway can recruit and onboard new carers in the system, in one pilot branch, with real data and full security.

- Project setup, design system, error monitoring and automated tests.
- Database schema, row-level security on every table, soft delete and audit triggers.
- Staff login with mandatory two-factor, idle timeout and user invites.
- Branches, the three roles and the branch switcher.
- Worker profile, document types and staff document uploads.
- The 16 stages with hard dependencies, stage colours and date rules with overrides.
- Simple induction, training, shadowing and competency records as stage evidence.
- Gate 1 and Gate 2 with the "Why blocked?" panel, and calculated compliance status.
- Application links and document request links with full link security, the Pending review queue and email.
- Background worker: virus scanning, link expiry and the weekly file backup.
- Online agreement for applicant declarations and policy acknowledgements, and scanned paper signatures for the offer letter and contract (section 11).
- Worker history tab (from the audit trail), a basic branch dashboard and Admin-only transfers.
- Before real data: a data protection impact assessment (DPIA) and the applicant privacy notice.

### Phase 2 – Ongoing compliance (after beta hiring)

- Renewals and expiry reminders, with automatic document request links.
- Training matrix view.
- Supervisions, appraisals, spot checks and medication observations.
- Confidential HR notes and leavers.
- CQC worker file with its one-click PDF, and other PDFs generated from data.
- Audit log search screen and the overrides report.
- Full dashboard tiles and CSV exports.
- Bin screen, restore and automatic removal after 30 days.
- The three fixed templates (offer letter, contract, reference request), filled in from the worker's data and printed for paper signature (e-signature from Phase 3).
- Migration of existing staff from spreadsheets and Drive (scope to confirm).

### Phase 3 – After completion

- AI assistant: MCP connector, in-app assistant and scheduled AI tasks.
- Smart date reading from uploaded documents.
- DBS certificate scan with automatic deletion after Gate 2.
- SMS fallback and the daily manager summary email.
- Built-in e-signatures (section 11), alongside the permanent paper route.
- Template editor, and settings screens for link expiry, lockout limit and reminder periods.

Each phase is tested on a test site with synthetic data before it reaches real users.

## 4. Users, roles and permissions

There are three roles. Each user is assigned to one or more branches and only ever sees data for the branch they are currently working in. Roles and permissions are stored as data and checked through one permission module, so a role such as Recruitment or Read-only can be added later without rewriting checks.

| Permission | Admin | Branch Manager | HR |
| --- | --- | --- | --- |
| Branches | All, switches between them | Own branch | One or more assigned |
| Create branches | Yes | No | No |
| Onboarding stages (create, edit, upload evidence, enter applications) | Yes | Yes | Yes |
| Staff after onboarding (renewals, supervisions, records) | Yes | Yes | Yes |
| Approve Gate 1 and Gate 2 | Yes | Yes | Yes |
| Override date rules (reason required) | Yes | Yes | Yes |
| Send links; approve or reject uploads | Yes | Yes | Yes |
| Health declaration, DBS details, confidential notes | Yes | Yes | Yes |
| Generate PDFs and exports (Phase 2) | Yes | Yes | Yes |
| Manage users | All branches | Own branch | No |
| Transfer a worker between branches | Yes | No | No |
| Change courses, document types and settings | Yes | No | No |
| View a worker's history | Yes | Yes | Yes |
| Search the full audit log (Phase 2) | Yes | No | No |
| Restore from the bin (Phase 2) | Anything | Items they could delete | Items they could delete |
| Permanently delete or empty the bin (Phase 2) | Yes | No | No |
| AI features (Phase 3) | To be decided in Phase 3 | | |

**Privacy note:** all three roles see special category data (health and DBS). This is acceptable because Primeway's recruiters are its HR staff. If Primeway later hires recruiters who are not HR, a Recruitment role without access to health, DBS and confidential data should be added before they get accounts. Sensitive data is kept in its own tables from Phase 1 so that this needs only new policies, not a schema change.

## 5. Branch model

Each branch works as a separate system for its users, but there is one codebase and one database. Every branch record carries a `branch_id`. Separation is enforced twice: in the server code, where every query is scoped to the user's current branch and checked against their role, and in the database through row-level security (section 14).

- **No combined dashboard.** Admin picks a branch from the branch switcher and sees only that branch. HR users with several branches use the same switcher.
- **Shared settings.** Only Admin adds or changes training courses and document types, either shared by all branches or for one branch.
- **Users** are created per branch. Branch Managers manage users in their own branch; Admin manages all. Only Admin creates branches.
- **Transfers.** Only Admin can move a worker to another branch. The record moves with its full history, the transfer is logged with a reason, and the worker shows a "Transfer review" flag in the receiving branch until a Branch Manager or HR user there confirms they have reviewed compliance. Any extra CQC steps are added later as a checklist.

## 6. Staff journey, gates and compliance status

### The 16 stages

1. Application received
2. Interview completed
3. Conditional offer
4. References requested
5. References approved
6. Identity verified
7. Right to Work checked
8. DBS submitted
9. DBS certificate received and reviewed
10. Contract completed (signed or agreed)
11. Corporate induction
12. Mandatory training
13. Shadowing completed
14. Competency assessed
15. Final compliance approval (Gate 2)
16. First independent shift (date entered manually)

Each stage records who completed it, the date and its evidence.

### Stage order: hard dependencies only

Stages can be worked on in any order, so office staff can check identity, Right to Work and training while waiting for references or DBS. A stage is locked only when it needs data from another stage:

| Stage | Locked until |
| --- | --- |
| 2 Interview completed | 1 Application received |
| 3 Conditional offer | 2 Interview completed |
| 5 References approved | 4 References requested |
| 9 DBS certificate received and reviewed | 8 DBS submitted |
| 10 Contract completed | 3 Conditional offer |
| 13 Shadowing completed | Gate 1 approved |
| 14 Competency assessed | 12 Mandatory training (the related course) |
| 15 Gate 2 | Every Gate 2 requirement (below) |
| 16 First independent shift | Gate 2 approved |

Date rules (section 8) still apply whenever both dates exist, whatever order the stages were done in.

### Stage colours

| Colour | Stage state | Meaning |
| --- | --- | --- |
| Grey | Not started | Nothing recorded yet |
| Amber | In progress or waiting | Started, or waiting on someone else (for example a referee or DBS) |
| Red | Overdue or failed | Past its target time, or rejected (for example a reference rejected or a DBS outcome that needs review) |
| Green | Complete | Done, with evidence |
| Locked | Locked | A hard dependency is not complete |

Target times per stage are constants in code to start with (for example references 14 days, DBS 28 days; to confirm). Red is kept for things that need action, so the onboarding list does not turn red simply because work is under way.

### The two gates

**Gate 1 – Approved for shadowing.** The pre-employment checks are done, so the person can go on shadow shifts alongside an experienced carer. They never work alone. Proposed requirements (to confirm): stages 1–12 complete, which means application, interview, offer, references approved, identity, Right to Work, DBS reviewed, contract, induction and mandatory training.

**Gate 2 – Approved for independent work.** The final sign-off that the person can visit clients on their own. Requirements: Gate 1 approved, shadowing complete with a "complete" decision, a "competent" outcome for each service they will provide, every mandatory document approved (none waiting in Pending review), and nothing expired.

Both gates are approved by HR, Branch Manager or Admin. The button stays locked until every requirement is met, and a **"Why blocked?"** panel lists what is missing and who needs to act. Each approval records the approver, date, comments, restrictions, branch, role and the services the worker is competent to provide. Gate checks run on the server and cannot be bypassed.

### Compliance status

Each worker has one compliance status, calculated whenever it is read from their stages, gates and expiry dates. It is never stored, so it can never be out of date.

| Status | Colour | Meaning |
| --- | --- | --- |
| In progress | Amber | Onboarding under way |
| Approved for shadowing | Blue-teal (#0A6A8C on #DDEFF7) | Gate 1 passed; shadow shifts only |
| Approved for independent work | Green | Gate 2 passed and nothing expired |
| Blocked | Red | An expired document (for example a visa or Right to Work), a failed check, or a missing renewal |
| Leaver | Grey | Left the organisation; record kept read-only |

Employment status (active, inactive, suspended, leaver) is stored separately from compliance status.

## 7. Staff profile and documents

The staff profile is the single official record for each worker.

- **Personal details:** full name, address, phone, email, date of birth, NI number, employee number, emergency contact.
- **Employment details:** job title, branch, employment type, hours, line manager, start date, first independent shift date, employment status.
- **Visa and Right to Work:** visa type, sponsor details, expiry date and follow-up check dates.
- **DBS record:** certificate number, issue date, level, outcome, Update Service status, who checked it and when. No scan of the certificate is stored until Phase 3.
- **Recruitment records:** application, CV, interview form and scoring, offer, contract, references, employment history with gap explanations, ID documents, health declaration, confidentiality agreement and policy acknowledgements.
- **Equal opportunities:** optional, stored in its own table, never shown on the profile or used in decisions; only used for anonymous reporting.

**Document types** are added and changed by Admin only. Each type defines whether it needs an issue date, an expiry date and a reminder period.

**Files:** documents received from applicants and workers are stored as uploaded, in private storage, and opened only through short-lived signed links. Every view or download of a sensitive document is logged. Data entered in the system is stored as data; PDFs are generated from it in Phase 2. **Signed documents** (scanned paper copies now, e-signed files from Phase 3) are the exception: the signed file is stored permanently and never regenerated or changed, because it must stay exactly what the person signed.

## 8. Date rules and override

The system rejects any date that breaks the required order. The rules are written once as shared validation code (Zod) used by both the forms and the server, so they cannot be bypassed.

| Rule | Must be true |
| --- | --- |
| Interview | On or after the application date |
| Offer | On or after the interview date |
| Reference received | On or after reference requested |
| DBS certificate (new application) | On or after the DBS application date |
| DBS via Update Service | Certificate may be older; the Update Service check date must be on or after the application date |
| Induction completed | After the offer |
| Shadowing completed | After induction and mandatory training |
| Competency completed | After the related training |
| First independent shift | After Gate 2 and after DBS, Right to Work, references, induction, training, shadowing and competency |
| Any document | Expiry date after issue date |
| Reviews | Next review date after the last assessment |

**Override:** HR, Branch Manager or Admin can override a rule with a mandatory reason. The item shows an "Overridden" flag on the profile (and later in the CQC worker file), and every override is logged with who, when, the rule, the original value and the reason. Correcting a mistyped date is a normal edit, also logged. Gate requirements cannot be overridden.

## 9. Induction, training, shadowing and competency

In Phase 1 these are simple records that supply the evidence for stages 11–14 and the Gate 2 checks.

| Record | Fields |
| --- | --- |
| Training | Course (from the Admin list), completion date, expiry date, certificate upload, provider, method (online, classroom, practical) |
| Induction | Modules, trainer, completion date, knowledge test score |
| Shadowing | Shift dates, supervisor, hours, feedback, decision (complete or more shadowing needed) |
| Competency assessment | Type (medication, moving and handling, IPC, personal care, BLS, documentation, equipment- or client-specific), assessor, date, outcome (competent, partly competent, not competent), comments, actions, reassessment date, evidence |

**Default course list** (Admin can change it): safeguarding, medication, moving and handling, infection prevention and control, basic life support and first aid, health and safety, fire safety, MCA and DoLS, dementia, autism and learning disability, equality and diversity, GDPR, food hygiene, and the Care Certificate standards.

**Competency rule:** a worker cannot be approved for a service without a "competent" outcome for it.

Phase 2 adds the training matrix view (every worker against every course, with overdue and expiring items highlighted) and renewal reminders.

## 10. Applicant links and the Pending review queue (Phase 1)

Applicants and workers never log in. Staff send them a secure, expiring link, and anything submitted waits for staff approval before it reaches the official record.

**Link types:**

- **Application link:** opens the online application form and the first document uploads (ID, Right to Work, proof of address, certificates). The form saves automatically, so the applicant can continue later through the same link.
- **Document request link:** asks for specific documents. In Phase 1 staff send it with **Request document**; in Phase 2 it is also sent automatically when a document reaches its reminder period.

**Staff entry:** HR, Branch Manager or Admin can also fill in an application directly (from a paper form, a phone call or a walk-in) and upload documents. The record shows it was entered by staff and by whom, and the same date rules and review steps apply.

**Creating a link** requires the person's name, date of birth, phone number and email. In Phase 1 links are sent by email only.

**Security (all required in Phase 1):**

- The person enters their date of birth before the link opens. After 5 wrong attempts the link locks and staff are notified.
- Each link is a long random token (at least 32 bytes), stored only as a hash, tied to one person and one request.
- Links expire after 3 days by default. The sender can set a different expiry, and can resend or cancel a link.
- After the date of birth is checked, the browser gets a short-lived, secure, http-only session cookie tied to that one link.
- The link only allows form completion and uploads. It shows nothing from the official record, only the applicant's own unsubmitted draft.
- Rate limiting on the date of birth check and on uploads, by link and by IP address.
- Uploads: PDF, JPEG and PNG only (HEIC to confirm), checked by the file's contents and not just its name, with a size limit (10 MB per file to start).
- Uploads go to a private quarantine area. The background worker virus-scans them (ClamAV). Clean files move to private document storage; infected files are deleted, logged and reported to staff. Staff cannot open a file until it has passed the scan.
- Link pages load no third-party scripts or analytics, send no referrer, and are not indexed by search engines.
- Link emails contain no personal data beyond the person's first name.
- The page shows Primeway's applicant privacy notice before any data is entered.

**Pending review:** each upload and each submitted application appears in the branch's **Pending review** queue. HR, Branch Manager or Admin approves or rejects it, with a reason. Approval updates the profile, expiry dates and compliance status. Rejection can send a new link.

## 11. Agreements and signatures

### Phase 1: online agreement and paper signatures

- **Online agreement:** at the end of the application form, and for policy acknowledgements, the applicant ticks **"I have read and agree"** and chooses from a dropdown how they will sign documents that need a signature: **Agree online** or **Sign on paper at the office** (options to confirm). It must meet UK law (an electronic agreement is valid when the person takes a clear, positive action):
  - The box is **never pre-ticked** and the dropdown has **no pre-selected answer**. The form cannot be submitted until the person ticks the box and chooses an option themselves.
  - The full document or wording is shown before the box, not only a link to it.
  - The system saves a permanent record of **who** (full name as typed, the worker record, and the date-of-birth-checked link session), **what** (a stored copy of the exact document or wording they agreed to, with its version and a fingerprint so it can be shown unchanged later), **when** (date and time, UK time and UTC) and **where** (IP address).
  - The record cannot be edited or deleted. If the wording changes later, people agree to the new version as a new record.
  - Privacy: this agreement is not used as the legal basis for processing staff data under UK GDPR (that basis is the employment contract and legal obligations), so withdrawing it does not delete their record.
- **Paper signatures:** in Phase 1 the offer letter and the contract are signed on paper. Office staff scan the signed copy and upload it to the worker's profile, where it is reviewed like any other upload.
- **The paper route stays permanently,** for anyone who cannot complete things online, even after e-signatures arrive.
- **Signature method on every record:** anything that depends on a signature or agreement records how it was completed: online agreement, paper (scanned) or, from Phase 3, e-signature. The profile shows the method, and so will the CQC worker file in Phase 2.
- **Stage 10** (Contract completed) completes when the scanned signed contract is uploaded and approved.
- **Signed copies are kept permanently.** A stored signed copy cannot be replaced; a correction is a new upload and the old one is kept. Only Admin can move a signed copy to the bin.
- Agreements, uploads and approvals of signed documents are all logged.

### Phase 3: built-in e-signatures

Built into the system: no external provider, no per-signature fee, and contracts never leave Primeway's own storage. A low-cost paid provider with UK or EU processing and a data processing agreement is the fallback if the built-in version proves too costly (decided in Phase 3).

- Staff press **Send for signature** on a PDF (uploaded, or generated from a template). This sends a signing link with the same security as other links (section 10).
- The person reads the whole document, confirms they agree to sign electronically, then signs (draws or types their name) or ticks **"I have read and agree"**.
- The system stamps the signature, name, date and time and IP address onto the PDF, adds a final page recording these details, stores a fingerprint (SHA-256 hash) of the final file and saves it permanently.

## 12. Notifications

- **Phase 1 – email only:** links to applicants, and staff alerts for links locked, uploads pending review, infected files and approvals needed. Staff also see these in the notification bell.
- **Phase 2:** expiry reminders at configurable periods (default 60 and 30 days), outstanding references, DBS and Right to Work, and one-click chase.
- **Phase 3:** SMS for people without email, and the daily manager summary.
- The email provider must process data in the UK or EU and sign a data processing agreement (chosen at build; to confirm).

## 13. Dashboard, reports and audit trail

**Phase 1 dashboard:** a filterable worker list for the current branch (default: active workers), onboarding progress by stage, workers blocked, uploads pending review and links locked. One search bar finds workers in the current branch.

**Phase 2:** the full dashboard tiles (expiring training and competencies, supervisions due, visa expiries, overrides, leavers), CSV export of any list, the overrides report and the CQC worker file with its one-click PDF.

**Audit trail (Phase 1):**

- Logged automatically by database triggers: every create, edit (old and new value), deletion and restore, approval, rejection, override and status change. The application also logs logins, links sent, opened and locked, and every view or download of a sensitive document.
- Each entry records the user, branch, date and time, and IP address.
- The log is append-only and enforced in the database: no role, including Admin, can change or delete entries.
- Phase 1 shows each worker's history on their profile. Phase 2 adds the log search screen for Admin.

## 14. Tech stack and architecture

| Part | Choice | Responsibility |
| --- | --- | --- |
| Web app and server | Next.js (App Router, TypeScript) on DigitalOcean App Platform, London | All screens, forms, server actions, validation, gates, links |
| Database access | Prisma | All queries from the web app, run as the signed-in user |
| Background worker | Node.js worker on DigitalOcean App Platform, London | Virus scanning, link expiry, weekly file backup (Phase 1); reminders and other jobs later |
| Job queue | pg-boss (Postgres-backed) | Retries, no lost jobs, one instance per scheduled job |
| Database | Supabase Postgres, London (eu-west-2), Pro plan | All data, row-level security, audit log |
| Auth | Supabase Auth with @supabase/ssr | Email and password, mandatory two-factor (TOTP), session cookies |
| File storage | Supabase Storage, private buckets | Quarantine and documents; short-lived signed URLs |
| File backup | DigitalOcean Spaces, London | Weekly copy of all stored files |
| Email | Provider with UK/EU processing (to confirm) | Links and staff alerts |
| UI | Tailwind CSS, shadcn/ui, design system from the prototype (blue accent #2462C8) | Screens follow the design prototype (97 screens, aligned with v3, phases marked). Where a screen and this spec disagree, the spec wins |
| Forms and rules | React Hook Form and Zod | Shared validation and date rules |
| PDFs | pdf-lib | Generated PDFs (Phase 2); signature stamping (Phase 3) |
| Tables | TanStack Table | |
| Tests | Vitest (rules, permissions, RLS) and Playwright (end-to-end) | |
| Monitoring | Sentry, EU region, with personal data scrubbed | |

**How the database is reached (decision for Phase 1):** Prisma normally connects as a privileged database user, which skips row-level security. To keep RLS as a second layer, the web app never queries the database directly. Every query goes through one helper that opens a transaction, switches to the `authenticated` database role, and sets the signed-in user's verified identity and IP address for that transaction only. RLS policies and audit triggers then see who is acting. Only the background worker and database migrations use the privileged connection.

**Architecture rules:**

- Business logic (stage dependencies, date rules, gate requirements, status calculation and permissions) lives in plain TypeScript modules, not in pages or components, so it is testable and shared.
- Every server action and route checks the user's session, role and current branch before doing anything.
- The browser never queries the database or storage directly. Database tables are not exposed through Supabase's public Data API.
- The Supabase service-role key and the privileged database connection exist only in the worker and in migrations, never in the web app or the browser.
- Every rule and permission has automated tests, including tests that prove one branch cannot read another branch's data.

## 15. Security, hosting and backups (all Phase 1)

All production data and backups stay in London: Supabase eu-west-2 and DigitalOcean LON. Primeway owns every account, the domain, the code repository and the data.

**Branch separation:** server-side scoping and role checks on every query, plus row-level security on every table that holds branch data. Policies check the user's branch assignments and role. Deleted rows are hidden by policy.

**Access and data protection:**

- Two-factor login is mandatory for every staff account before any data can be seen.
- Idle session timeout (30 minutes to start) and an absolute session limit.
- Strong password rules, and invite-only accounts.
- Encryption in transit (TLS) and at rest.
- Private storage with short-lived signed URLs; every sensitive download logged.
- Security headers on every page (content security policy, HSTS, no framing).
- Secrets only in DigitalOcean's encrypted settings, never in the code repository.
- Dependency updates and security alerts switched on for the repository.
- Development and testing use synthetic data only.
- Before the beta uses real data: a DPIA, the applicant privacy notice, and retention periods agreed (including for unsuccessful applicants).

**Backups:**

| What | How | Frequency |
| --- | --- | --- |
| Database | Supabase managed backups (Pro: daily, kept 7 days) | Daily |
| Uploaded files | Worker copies the storage buckets to a private DigitalOcean Spaces bucket in London | Weekly (daily if a week's loss is not acceptable) |

## 16. Soft delete and the bin

Nothing is deleted straight away.

- **Phase 1:** every table that can be deleted from has `deleted_at` and `deleted_by` columns. Deleted items are hidden from every screen, search and export by row-level security and query helpers. Deletion is logged.
- **Phase 2:** the bin screen per branch, restore (by anyone who could delete the item; Admin can restore anything), permanent removal by Admin, and automatic removal after 30 days by the worker.
- The audit trail is never deleted and never goes to the bin. Permanently deleted data can remain in backups until they expire.

## 17. AI assistant (Phase 3)

The MCP connector for business Claude, ChatGPT and Gemini plans, the in-app assistant and scheduled AI tasks, as described in section 16 of v2, are moved to Phase 3. The design will be reviewed before building, including which data the AI can see. Health, DBS and confidential data should be hidden from the AI by default.

## 18. Out of scope

- Integrations: rota, Nourish, Monday.com, Google Workspace, BrightPay, Citation, and a public API.
- Self-service: an employee portal or "change my details" workflow.
- Third-party e-signature providers (unless chosen as the Phase 3 fallback in section 11), native apps, auto-cropping of photos and an organisation-wide dashboard.
- HR casework modules (disciplinary, grievance, PIP); confidential notes cover these in Phase 2.

## 19. Open items to confirm

- Gate 1 requirements: stages 1–12 complete?
- Allow Gate 1 with a DBS Adult First check while the full certificate is pending?
- Can the same person enter a check and approve the gate for that worker, or must approval be by someone else?
- Target times per stage (for example references 14 days, DBS 28 days).
- Accept HEIC photos from iPhones on the applicant link?
- Email provider and its data processing terms.
- Retention periods, including for unsuccessful applicants.
- Exact CQC steps for branch transfers.
- Data migration scope for existing staff (Phase 2).
- File backup retention (proposed: last 4 weekly copies).
- Online agreement: confirm with Primeway's employment adviser that a tick box is enough for application declarations and policy acknowledgements, and confirm the dropdown options.
- E-signatures (Phase 3): confirm that a simple electronic signature is acceptable for contracts.
