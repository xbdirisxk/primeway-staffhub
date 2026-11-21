# Primeway StaffHub — design prototype

Clickable HTML prototype of the staff compliance system (97 screens). It is the **visual and flow reference** for the Next.js build. It is **not** production code, and it is **not** the source of truth for behaviour.

**Aligned with Developer Specification v3** (21 Nov 2026). Where a screen and the spec disagree, **the spec wins** (`docs/specs/staffhub-spec-v3-ai.md`).

## Files
- `index.html` — the whole prototype: styles, icons, sample data, every screen and the router, in one file. Open it in a browser.
- `avatars/*.jpg` — staff and worker photos (not included; see below).

## Phases
Every screen belongs to a delivery phase. Screens for later phases show a banner at the top and a **P2** / **P3** tag in the **Flows** panel. Anything not tagged is Phase 1.

| Phase | Screens |
| --- | --- |
| **1 — build now** | Access (sign in, 2FA, reset, invite, timeout) · dashboard and worker lists · search · notifications · account · new applicant by link (link, DOB, 4-step application, review queue) · manual entry (all 16 stages and both gates) · worker profile tabs: overview, onboarding, documents, training, history, edit · onboarding records (DBS details, induction, shadowing, competency, override) · gates · record a signed document · send link, document request, locked link · transfer and transfer review · users, roles, branches, training courses, document types |
| **2 — after beta hiring** | ongoing compliance tabs · confidential notes · renewals · training matrix · supervisions · HR note · leaver · reports · overrides report · CQC worker file · audit log search · generate contract from template · templates settings |
| **3 — after completion** | daily summary email · e-signature (sign, signed) · reminders and links settings · security and data settings |

In the code, `PHASE` (in the runtime section near the bottom) holds this map.

## Roles
Three fixed roles: **Admin**, **Branch Manager**, **HR**. Use **View as** (bottom of the screen) to switch, and **Flows** to step through every screen. `PW.ROLES` and `PW.PERMS` near the top of the main script define them.

| Permission | Admin | Branch Manager | HR |
|---|---|---|---|
| Branches | All, switches | Own | Assigned |
| Onboarding stages 1–16 | Yes | Yes | Yes |
| Staff after onboarding | Yes | Yes | Yes |
| Approve Gate 1 and Gate 2 | Yes | Yes | Yes |
| Override date rules (reason required) | Yes | Yes | Yes |
| Approve uploads from links | Yes | Yes | Yes |
| Confidential HR notes, health declaration | Yes | Yes | Yes |
| Generate PDFs and exports (Phase 2) | Yes | Yes | Yes |
| Manage users | All branches | Own branch | No |
| Transfer workers between branches | Yes | No | No |
| System settings (courses, document types) | Yes | No | No |
| Search the full audit log (Phase 2) | Yes | No | No |

## What changed to match spec v3
- **Stage order:** stages can be done in any order. Only stages that need another stage's data are locked: 2←1, 3←2, 5←4, 9←8, 10←3, 13←Gate 1, 14←related training, 15←every Gate 2 item, 16←Gate 2. Each stage shows a state pill: grey **not started**, amber **in progress**, red **overdue or failed**, green **complete**, **locked**. The manual-entry rail explains why a stage is locked.
- **Signatures (Phase 1):** the offer letter and contract are signed on paper, then scanned and uploaded by staff. There is a new screen, *Record a signed document*. Every signed or agreed item shows its method: **Paper, scanned** or **Online agreement**. E-signature screens are tagged Phase 3; the paper route stays permanently.
- **Online agreement:** the application is now 4 steps. Step 3 is health and optional equal opportunities. Step 4 shows the full declaration wording, a tick box that is **never pre-ticked**, and a signing-preference dropdown with **no default**. A note explains that name, wording version, time and IP are saved.
- **Privacy notice** appears on the date-of-birth page, before any data is entered.
- **Equal opportunities** is optional and explained as stored separately and never seen by decision-makers.
- **No AI date reading:** staff type the dates from the document. The lavender "suggested" fields and "read automatically" labels are removed from Phase 1 screens.
- **No DBS certificate scan:** only the certificate details are recorded. All "copy deleted at Gate 2" wording is removed.
- **Email only:** SMS options are removed from Phase 1 screens. If a person has no email, staff use manual entry and collect documents in person.
- **No daily summary email** in Phase 1; the dashboard button now opens Pending review.
- **Transfers are Admin-only** (`transfer` permission).
- **Uploads:** JPEG, PNG or PDF, up to 10 MB, virus scanned before anyone can open them.
- **Sessions:** idle sign-out is 30 minutes. Two-factor uses an authenticator app only, and an Admin resets lost devices (no email codes).
- The **assessor confirmation** on competency forms is a tick box tied to the signed-in user, not a drawn signature.

## Still to confirm (from the spec)
Gate 1 prerequisites, DBS Adult First, dropdown wording for the signing preference, retention periods, CQC transfer steps. Screens show these with a **To confirm** tag where relevant.

## Avatars
Photos load from `avatars/<code>.jpg`. Without them the avatars show as broken images, but layout and everything else work. Files expected:

af, ah, ap, aw, bl, cd, cw, di, do, ek, en, fa, fn, gd, gh, hs, hw-purple, hw-teal, iy, jm, jt, lc, lf, me, nb, oa, pj, pm, ps, rm, rq, sa, tb, tb-purple, vp, wh, yo

All names and photos are sample data. Do not put real staff data into the prototype.

## Notes for the real build
- The prototype hides things on screen only. In the app, enforce every permission in the API and again in Postgres (RLS), and deny by default when a role is unknown.
- The roles enum has exactly three values: `ADMIN`, `BRANCH_MANAGER`, `HR`. No `Recruitment` or `Read-only` values anywhere — not in the database, RLS policies or sessions.
- Sample values in forms (pre-filled fields, selected chips) are for illustration. In the real app, the agreement tick box and signing-preference dropdown must start empty.
- Translate the CSS into Tailwind tokens and shadcn/ui components; do not ship the prototype's single-file router.
