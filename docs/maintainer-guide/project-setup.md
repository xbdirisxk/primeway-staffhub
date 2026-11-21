# Project setup guide

First-time setup of the StaffHub repository, branch protection and Claude Code. Follow the steps in order.

## 0. Before you start
Install, if you don't have them:
- **Node.js** current LTS (`node -v` should show 22 or newer) and npm
- **Git**, and optionally the **GitHub CLI** (`gh`)
- **Docker Desktop** — needed later to run Supabase locally
- **Python 3.10+** — the Claude Code security plugin needs it
- **Claude Code** (desktop app or terminal)

## Where things live
- **One folder for the whole project:** the clone of your GitHub repo, for example `~/Projects/staffhub` (Windows: `C:\Users\<you>\Projects\staffhub`). Everything — the Next.js app, `docs/`, `.claude/` — lives inside it.
- **Claude Code always runs from that folder.** Every session, every prompt and every agent starts there, because that's where it finds `CLAUDE.md`, the agents and the docs. You never create a new folder per session or per prompt.
- Don't put the repo inside a cloud-synced folder (OneDrive, iCloud Drive, Dropbox). Syncing `node_modules` and `.git` causes slow, broken installs.

## 1. Clone the repo and start a setup branch
In a terminal (macOS Terminal, or Git Bash / PowerShell on Windows):
```bash
mkdir -p ~/Projects && cd ~/Projects      # your projects folder (create it once)
git clone https://github.com/<you>/<repo>.git staffhub
cd staffhub                               # from now on, everything happens in here
git log --oneline -1                      # main needs at least one commit (a README is enough)
git switch -c chore/project-setup
```
If the repo is completely empty, add a README on GitHub first (Add file → Create new file), then run `git pull` so `main` exists.

## 2. Create the Next.js app
```bash
npx create-next-app@latest . --typescript --eslint --tailwind --app --src-dir --import-alias "@/*" --use-npm
```
- If it refuses because the folder isn't empty, create it in a temporary folder (`npx create-next-app@latest ../staffhub-tmp …`) and move everything except `.git` into the repo.
- **Don't use the `with-supabase` example template.** It reads Supabase from the browser, which the spec forbids. Supabase auth is added in milestone M3, the spec's way.
- Check `npm run dev` opens the starter page, then stop it.

## 3. Install the git hooks (husky)
```bash
npm install --save-dev husky
npx husky init
```
This adds a `prepare` script to `package.json` and a `.husky/` folder. The starter kit's hooks replace the default one in the next step.

## 4. Copy in the starter kit
Unzip the starter kit and copy its contents into the repo root, replacing files when asked:
```
CLAUDE.md
.claude/settings.json
.claude/claude-security-guidance.md
.claude/agents/architect.md, db-rls-reviewer.md, security-reviewer.md
.husky/pre-commit, .husky/pre-push
.github/pull_request_template.md
docs/ (whole folder, including docs/specs/)
```
Then on macOS or Linux:
```bash
chmod +x .husky/pre-commit .husky/pre-push
```

## 5. Check `.gitignore` for secrets
`create-next-app` already ignores `.env*`. Add these lines so the example file (names only, no values) can be committed:
```
!.env.example
supabase/.temp/
```
Never put real keys in `.env.example`.

## 6. Add the Supabase CLI (local only for now)
```bash
npm install --save-dev supabase
npx supabase init
```
This creates the `supabase/` folder. **Don't** create or link a cloud project yet. That happens in M2/M3, after the data model is designed. When you do:
- Create the project in **London (eu-west-2)**, with separate projects for staging and production.
- Turn off the Data API for app tables, and keep the service-role key out of the web app.

## 7. First commit through a pull request
```bash
git add .
git commit -m "chore: project setup, Claude Code config, docs structure"
git push -u origin chore/project-setup
gh pr create --fill          # or open the PR on github.com
```
Merge the PR on GitHub, then:
```bash
git switch main && git pull
git commit --allow-empty -m "test"   # should be BLOCKED by the pre-commit hook
```

## 8. Protect `main` on GitHub
**Settings → Rules → Rulesets → New ruleset → New branch ruleset**
- **Name:** `protect-main` · **Enforcement:** Active · **Target branches:** include the default branch
- **Bypass list:** empty, so even you go through pull requests
- **Rules to turn on:**
  - Restrict deletions
  - Block force pushes
  - Require a pull request before merging, with **0 required approvals**. You work alone, and GitHub doesn't let you approve your own PR.
  - Require status checks to pass: add this after milestone M1 creates the CI workflow.

**Plan note:** rulesets on a **private** repository need GitHub Pro, Team or Enterprise. On the free plan they only work on public repos. Your options for a private repo on the free plan:
- **Upgrade to GitHub Pro (recommended).**
- **Rely on the local protection only.** That means the husky hooks plus Claude Code's deny rules, which are weaker because they can be bypassed on purpose.

**Also turn on:** Settings → Code security → Dependabot alerts and security updates, plus secret scanning if your plan offers it.

## 9. Set up Claude Code in the repo
Open Claude Code **in the repo folder**:
- **Terminal:** `cd ~/Projects/staffhub` then run `claude`.
- **Claude desktop app:** start a Claude Code session and choose the `staffhub` folder as its working folder.

Every later session starts the same way, in the same folder. The project settings already enable the security plugin. If it doesn't load, install it:
```
/plugin install security-guidance@claude-plugins-official
```
Check that everything is loaded by asking: *"Which subagents and plugins are available in this project?"* You should see `architect`, `db-rls-reviewer`, `security-reviewer` and `security-guidance`.

## 10. The first four sessions (prompts)
Start each one in a **new session** and in **plan mode**: press `Shift+Tab` until the status bar shows plan mode.

### Session 1 — Orientation (no changes)
```
Read CLAUDE.md, docs/README.md, docs/specs/staffhub-spec-v3-ai.md, docs/specs/design/prototype/README.md, docs/features/README.md and docs/decisions/. Don't change anything yet.
1. Summarise the project in about 10 lines so I can check you've understood it.
2. List anything in the spec, CLAUDE.md or the docs that contradicts itself or looks risky.
3. List the TBC items that block Phase 1, in the order we'll hit them.
4. Confirm the three agents and the security-guidance plugin are available.
```

### Session 2 — Architecture skeleton (design only)
```
Use the architect agent. Goal: the whole-system skeleton for Phase 1, before any feature code.
Cover: folder structure and module boundaries; the request lifecycle (browser → server action → session/MFA check → can() → withUser() → RLS → audit trigger); how the web app and the background worker are deployed on DigitalOcean and which secrets each one holds; local, staging and production environments; migration tooling (where the SQL for RLS, grants, helper functions and triggers lives alongside Prisma); and the shared foundations listed in docs/features/README.md.
Interview me first. Then propose docs/architecture/overview.md, docs/architecture/environments.md and one ADR per decision. Don't write any application code.
```
After you approve the plan:
```
Write the approved architecture docs and ADRs, update docs/decisions/README.md and add a docs/progress.md entry. Work on branch docs/architecture.
```

### Session 3 — Data model and RLS design (design only)
```
Use the architect agent, then the db-rls-reviewer agent. Goal: design — not build — the Phase 1 data model.
Produce docs/architecture/data-model.md (Mermaid ER diagram; every Phase 1 table from spec section 8 with columns, types, relationships, delete behaviour, indexes, enums and soft delete) and docs/architecture/rls-matrix.md (table × role × select/insert/update, helper functions, sensitive tables, insert-only tables).
Ask me about every TBC item that affects the schema. Have db-rls-reviewer check the design against its checklist and resolve its findings before showing me the final version. No migrations yet.
```

### Session 4 — Milestone M1 (project tooling)
```
Milestone M1 from docs/specs/staffhub-spec-v3-ai.md section 15, on branch feat/m1-setup.
Set up: TypeScript strict; ESLint including a no-restricted-imports rule that blocks src/lib/db/privileged outside worker/ and scripts/; Prettier; Vitest; Playwright; shadcn/ui with design tokens taken from the CSS variables in docs/specs/design/prototype/index.html (blue accent); Sentry (EU region, PII scrubbing) config with no DSN committed; a GitHub Actions workflow running lint, typecheck and tests on every pull request; npm scripts; .env.example with variable names only.
Fill in the Commands section of CLAUDE.md and "How to run" in docs/testing/README.md. Create docs/features/project-setup.md from the template. Run security-reviewer before we open the PR.
```
After M1 is merged, add the CI check under **Require status checks to pass** in the GitHub ruleset.

## 11. Every milestone after that (M2–M12)
```
Milestone M<n>: <name> (spec section 15). Read the spec sections it relies on and docs/features/<feature>.md.
Plan mode: interview me, then give a plan covering files, tables, RLS policies, tests, docs and the features affected (docs/features/README.md).
After I approve: work on branch feat/m<n>-<name> in small steps. Run db-rls-reviewer after any database change and security-reviewer before the PR. Update the feature doc (including "Impact if changed"), the dependency map and docs/progress.md.
```
End every session with: *"Update docs/progress.md with what's done, what's next and open questions."*
