# Staff Compliance prototype — build notes

Artifact: https://claude.ai/artifact/3BtLP73Gg8D9kfLkDrLYmE (version 2, 10 Nov 2026)
Built from: Primeway_Care_Staff_Compliance_System_Developer_Specification_v2.md + design/design-spec.md

## Decisions agreed with Mohamed
9 Nov 2026
- Scope: Phase 1 + Phase 2, plus applicant-side secure link screens.
- Signed in as Admin with branch switcher; "View as" previews Branch Admin, HR, Recruitment, Read-only.
- Open items built as the spec proposes and marked "To confirm" on screen: Gate 1 = stages 1–12; Recruitment cannot see health declaration; Read-only can export PDFs; link locks after 5 wrong DOB attempts; AI provider; retention periods; CQC transfer steps.

10 Nov 2026
- Colour: brand accent changed from teal to blue (#2462C8, hover #1C4FA4, tint #E7EFFC), neutrals shifted to blue-grey. Spec RAG colours kept: green = approved/valid, amber = in progress/due, red = blocked, grey = leaver. "Approved for shadowing" (spec: teal) now a blue-teal (#0A6A8C on #DDEFF7) so it doesn't read as green.
- Workers list shows Active workers only by default. Other groups are reached only through the "Showing …" status dropdown: Blocked, In onboarding, Shadowing only, Inactive, Suspended, Leavers. Recruitment's dropdown shows onboarding groups only.
- Add applicant offers two routes: send a secure link, or fill in manually. Manual entry is a staff form for every stage (1–16 plus Gate 1 and Gate 2) with a stage rail, date-rule checks per stage and a save bar.

## Gaps found in the spec check and added (10 Nov)
- Daily manager summary email preview (section 10).
- Receiving-branch transfer review in East London (section 4).
- Change employment status (active / inactive / suspended), separate from compliance status (section 5).
- Signed-out-after-idle screen (section 14).
- Recruitment sees only onboarding uploads in Pending review (section 3).

94 screens in 12 flows. Branches: North London (default), East London, Hertfordshire. All sample data is synthetic.
