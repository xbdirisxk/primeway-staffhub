# Design Spec — HR System (based on Primeway Care prototype)

Source: https://claude.ai/artifact/QW1PGUWaKPa6rHPo8pdTus
Rule: the HR system must look and behave **exactly** like this prototype. Only content, labels, data and screens change. Exact CSS is in `design-system.css`, icons in `icons.js` — reuse verbatim, do not restyle.

---

## 1. Visual character
- Calm, clinical, light-only product UI (no dark mode, by design — built for client sign-off).
- White surfaces on a white ground, separated by **hairline borders with real contrast**, almost no shadows.
- Near-black ink headings, **deep teal** as the single accent. Soft tints for status only.
- Dense but airy: 13px body, 36px controls, 46px table rows.
- British English, sentence case everywhere ("Users and roles", not "Users And Roles").

## 2. Colour tokens (`:root`)
| Token | Hex | Use |
|---|---|---|
| `--bg` | #FFFFFF | Page background |
| `--side` | #F6F8F7 | Sidebar, table headers, row hover, read-only fields |
| `--surface` | #FFFFFF | Cards |
| `--surface-2` | #EEF3F1 | Segmented control track, icon-button hover, file icon bg |
| `--surface-3` | #E2EAE7 | Count chips, meter track, nav hover, neutral pills |
| `--line` | #CFDBD6 | Card / input / table borders |
| `--line-soft` | #E0E9E5 | Inner dividers, row separators |
| `--ink` | #14191E | Primary text |
| `--ink-2` | #3B454B | Secondary text, labels |
| `--ink-3` | #5F6B70 | Muted text, meta, table headers |
| `--accent` | #0E7C6B | Primary buttons, active states, focus, chart bars |
| `--accent-2` | #0A6557 | Primary hover |
| `--sky` | #9FD9CC | Chart current bar, completed step bar |
| `--sky-soft` | #E1F2EE | Selected option / active menu tint |
| `--sky-line` | #B9E0D6 | Selected option border, info callout border |
| `--link` / `--link-2` | #0E7C6B / #0A6557 | Links |

**Status pairs (fg / bg)**
| Status | FG | BG | Meaning |
|---|---|---|---|
| ok | #1B7F3B | #E3F3E6 | Active, complete, verified |
| warn | #9A4A06 | #FFEFD9 | Due soon, held, partial |
| bad | #B1222A | #FCE7E7 | Overdue, missing, error, destructive |
| info | #1F6FB2 | #E7F0FA | In progress, due |
| note | #5A2BC2 | #EFE8FF | Suggested / AI-prefilled |
| teal | #0B6B63 | #DCF2EF | New / initial stage |
| gray | #3B454B | #E2EAE7 | Draft, neutral |

**Avatar tones** (initials on tint): blue #E3F1FB/#0B5E9C · teal #DCF2EF/#0B6B63 · amber #FFEFD9/#9A4A06 · red #FCE7E7/#B1222A · purple #EFE8FF/#5A2BC2 · gray #E8EBF0/#3D4256 · green #E1F4EA/#0E7A55. Real photo avatars used where available (circular, object-fit cover).

Overlay scrim: `rgba(18,20,51,.45)`. Focus ring: `0 0 0 3px rgba(14,124,107,.18)` + border `--accent`.

## 3. Typography
- Font: **Inter** 400/500/600/700 (Google Fonts), fallback system UI. `-webkit-font-smoothing: antialiased`.
- Body 13px / 1.5. Tabular numbers (`.tnum`) for all figures, dates, counts.
- H1 19px/1.25 600, letter-spacing −0.01em, `text-wrap: balance`.
- H2 15px/1.3 600 · H3 13px 600 · Dialog H2 17px · Auth card H1 20px.
- Nav 12.5px · labels 12px 500 · meta/hints 12px · group labels 11.5px 500 +0.02em.
- KPI tile value 22px 600 (small tile 18px).
- Weights are restrained: 400 body, 500 emphasis, 600 headings only.

## 4. Spacing, radius, elevation
- Radius: `--r` 12px (cards), `--r-s` 9px (buttons, inputs), `--r-xs` 7px. Pills 999px. Dialogs 16px, auth card 20px, popovers 14px.
- Page padding 20px 24px 40px; section gap 18px; grid gaps 16–20px.
- Card header 12px 16px; card padded body 20px 22px; list rows 7px 16px, min-height 46px.
- Shadows only on floating layers: popover `0 18px 50px rgba(20,25,30,.16)`, dialog `0 20px 60px rgba(18,20,51,.25)`, toast `0 10px 30px rgba(20,25,30,.14)`, auth card `0 12px 40px rgba(20,25,30,.06)`. Active nav item gets a 1px ring + tiny shadow.

## 5. App frame / layout
- **Sidebar** 232px, `--side` bg, right border `--line-soft`, sticky full height.
  - Brand: 28px rounded-8 logo + product name (13px 600).
  - Group labels ("Care", "Admin") → for HR e.g. "People", "Admin".
  - Nav item 36px, icon 17px, 12.5px text, optional right-aligned count chip.
  - **Active**: white bg, 1px ring, 500 weight, teal icon, **3px teal bar on the left edge**, count chip turns teal-tinted.
  - Footer: signed-in user (avatar + name + role), links to Account.
- **Top bar** min 54px, sticky, bottom border: sidebar toggle icon button → breadcrumbs (chevron separators; last crumb can be a dropdown menu) → spacer → page actions → notification bell with red dot + popover.
- Sidebar collapses (toggle) and becomes an off-canvas drawer < 900px; mini logo shows in top bar when hidden.
- **Content**: full width; variants `.narrow` 1120px, `.form` 1040px.
- **Page head**: H1 + muted sub-line (date · key alerts), actions right-aligned (filters as outline buttons with chevron, primary CTA).
- Grids: `grid2` / `grid3` / `grid4` auto-fit (min 260/220/230px). `row2` flex with wrap for side-by-side cards.
- Settings use a **216px sub-nav** + pane (`.split`).
- Long forms use a **284px section rail** (`.fcol`) with search, part headers, numbered section links with status dots (ok/bad/warn/changed) + sticky bottom save bar showing "Saved" tick.

## 6. Components (class → behaviour)
**Buttons** `.btn` 36px, 9px radius, 12.5px 500, 16px icon, press = translateY(1px).
`-p` primary teal · `-o` outline white · `-t` text/ghost · `-d` danger outline · `-r` danger solid · `-x` disabled. Sizes `-sm` 30px, `-lg` 44px, `-w` full width. Icon button `.ib` 32px square.

**Pills** `.pill p-ok|warn|bad|info|note|teal|gray|navy` — 11.5px 500, tinted bg. Used for every status.

**Avatars** `.av` 30px circle; `.s` 24, `.l` 40 (radius 12), `.xl` 52 (radius 16); `.stack` overlaps −8px with white ring.

**Cards** `.card` white, 1px `--line`, 12px radius. `.hd` header (title + right meta/link), `.ft` footer. `.tile` KPI card: label → big value + pill → muted description; whole tile clickable.

**Lists** `.row` (avatar + title/sub + right meta/pill, hover `--side`), `.kv` key/value rows (160px key column), `.ev` timeline events (time · text · right meta) under `.day` headers, `.diff` old (red strikethrough) → new (green) change view.

**Tabs** `.tabs/.tab` underline tabs with count chips, 2px teal underline when active. **Segmented** `.seg` pill-track toggle.

**Callouts** `.callout ok|warn|bad|note|info` tinted box, bold lead, optional inline button.

**Tables** header row `--side`, 40px, 12px 500 muted; body rows 46px, 12.5px, hover `--side`; highlighted row `#FFFBF2`; group header rows `tr.g`; name cell = avatar + bold name + small muted ID. Horizontal scroll wrapper `.tw`.

**Forms**
- Label `.lbl` 12px 500 (red `*` required, muted "(optional)").
- Field `.f` 38px wrapper around input/select with leading/trailing icons/text; focus = teal border + ring; `.ro` read-only grey; `.sg` **suggested/prefilled** (lavender border #C4B5FD, bg #FCFAFF).
- Textarea `.ta`; hint `.hint`.
- Choice chips `.opt` (selected = sky tint), dashed `.opt.add`.
- Toggle `.sw` 36×22 teal when on. Checkbox list `.ck`. Radio cards `.rc` / `.method` (selected = teal border + ring).
- Conditional sub-panel `.when` (grey box, animated reveal).
- Question layout `.q`: 2-column grid (200px label column + answer), stacks < 760px.
- Collapsible sections `.sec` (`<details>`): numbered square badge (teal when open), title, status pill, rotating chevron, footer with prev/next.
- Steppers: `.steps` (bar-style progress, 6 cols) and `.stp` (numbered circles joined by lines; done/current = teal).
- Search `.search` 40px, max 760px. Upload drop zone `.drop` dashed. File row `.filebox`, file-type tile `.ftile` (folded-corner doc icon), document slots `.slot` (dashed when missing).

**Directory cards** `.ccard` (A–Z grid, 340px min): avatar + name/ID + status pill, 4 key/value lines, footer meta; A–Z jump bar `.az` and `.letter` dividers.

**Charts** CSS bar chart `.chart`: 26px rounded bars, track in `--surface-3`, fill teal, current month in `--sky`, dark tooltip on hover. `.legend`, `.meter` progress bars (amber when below target).

**Calendar** `.cal` 7-col month grid, today = teal circle, event chips `.cev` coloured by state (routine/initial/in progress/done/overdue/next).

**Document preview** `.pgwrap/.pg` A4 page mock with grey line placeholders, optional red "DRAFT" watermark.

**Overlays**
- Notification popover (380px): title + "Mark all as read", items with icon tile + unread teal dot, "Show all" button.
- Dialog `.dlg` 440px: icon tile (teal or red), title, text, right-aligned buttons; busy state.
- Side sheet `.sheet` 440px from right.
- Toast top-right: white card with teal tick that draws itself.

**Auth pages** (sign in, 2FA, forgot/reset, invite): grey page, top bar with wordmark + language pill, centred 440px card (`.wide` 520, `.split` 860 with side panel e.g. QR code), 44px fields, 6-box code input, password rule list, 3-step indicator.

## 7. Iconography
Lucide-style outline icons, 24 viewBox, stroke 1.8, round caps. Set: home, users, cal, file, doc, check, checks, checks2, user, userplus, log, settings, search, bell, panel, down, right, left, x, plus, mail, download, upload, print, send, eye, info, warn, lock, shield, trash, rotate, zoomin, zoomout. Sizes: nav 17, buttons 16, fields 18, dialog 24.

## 8. Motion
- Hover transitions 0.15s on bg/border.
- Page enter: fade + 6px rise (0.22s); forward/back navigation slides ±28px; View Transitions API where supported.
- Popover: fade + slight scale from top-right, cubic-bezier(.2,.8,.2,1).
- Section expand: fade + 4px drop. Toast slides in from right. Tick draws via stroke-dashoffset.
- All motion off under `prefers-reduced-motion`.

## 9. Prototype mechanics (to replicate)
- Single-file SPA, hash routing (`#home`, `#clients`…), every screen rendered from shared helpers (`shell`, `sidebar`, `topbar`, `btn`, `pill`, `av`, `card`, `tile`, `kv`, `field`).
- Clickable throughout; destructive / committing buttons open a confirm dialog, others fire a toast.
- **Flows navigator**: floating pill at bottom-centre ("Flows · 12 / 48") opens a panel listing every screen grouped by flow, with Previous / Restart / Next; ← → arrow keys step through screens.
- Realistic UK sample data (names, regions, dates, IDs like `PC-0299`), photo avatars, today's-date context in headers.
- Responsive: drawer sidebar < 900px, single-column grids < 600px, safe-area insets respected.

## 10. Screen patterns in the source (map these to HR)
| Source flow | Screens | Likely HR equivalent |
|---|---|---|
| Access | Sign in, 2FA (app / email), Forgot, Reset, Invite, Invite 2FA | Same |
| Home | Dashboard: greeting + 4 KPI tiles, monthly bar chart, "due" list, completeness meters, recent activity | HR dashboard (headcount, leave, onboarding, compliance) |
| Directory | Clients A–Z cards, New client, Prefill | Employees directory, Add employee |
| Record | Overview, Assessment, Documents, PDFs, History tabs | Employee profile (personal, job, documents, history…) |
| Multi-part form | 6-part registration and review forms with section rail and save bar | Onboarding / appraisal forms |
| Scheduling | Reviews calendar + list, set review date | Leave / appraisal / probation calendar |
| Documents | Org-wide documents, upload, verify, generate PDF, send PDF | Contracts, right-to-work, policies |
| Work queue | Actions, Notifications | Approvals, tasks |
| Admin | Users, User detail, Roles & permissions, Settings (organisation, documents, wording, security), Audit log, Account | Same |

**Awaiting:** HR system functional specs → then build every screen in this exact system.
