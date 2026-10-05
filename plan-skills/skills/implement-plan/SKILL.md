---
name: implement-plan
description: Implement a numbered plan from docs/plans/ for the keshi booking SaaS project. Use when the user says "implement plan N", "start plan N", "work on plan N", or "next plan". Handles worktree creation, subagent-driven implementation, two-stage review (spec + quality), and merge back to master.
---

# Implement Plan
Before implement the plan, you need to review other docs to see any gap or different, if you find something, please use /grill-me to interview me for clarify what you should do.

Executes a single numbered plan from `docs/plans/` end-to-end: worktree → implement → review → merge.

## Workflow

### 1. Load the plan
Plans live in `docs/plans/to-do/<NN>-<slug>.md`, `docs/plans/backlog/<NN>-<slug>.md`, or `docs/plans/done/<NN>-<slug>.md`. Find the file by number (glob all three folders, e.g. `docs/plans/{to-do,backlog,done}/<NN>-*.md`) — don't assume which folder it's in.

- **Found in `backlog/`?** — backlog plans aren't prioritized yet. Tell the user this plan is still in backlog and ask whether to (a) promote it to `to-do/` first (`git mv` into `docs/plans/to-do/`) and continue, or (b) pick a different plan. Do NOT implement straight out of backlog without confirming.
- **Found in `done/`, or file contains `**Status: ✅ Completed`?** — tell the user this plan is already done and ask which plan to run instead. Do NOT re-implement it.
- **Type: HITL** — stop after implementation and tell the user to manually verify before running the next plan. Do NOT auto-proceed.
- **Type: AFK** — fully automated, proceed through to merge.
- **Blocked by** — confirm all listed prerequisite plans have `**Status: ✅ Completed` in their file before starting. Prerequisites may live in any of the three folders — check all when locating them.

### 1a. In-progress check (another session may already be on this plan)

Before picking plan `<NN>` (whether user-specified or auto-selected via "next"), run:

```bash
git worktree list
git branch -a --list "feat/<NN>-*"
```

If a worktree or branch matching `feat/<NN>-*` already exists, another session is already implementing this plan. Do NOT start a second one. Tell the user and pick the next eligible plan instead (or ask which plan to run if `<NN>` was explicit).

When auto-selecting "next", skip any plan number with a matching branch/worktree and continue down the list until an unclaimed one is found.

### 1b. Business-model impact check

Before setting up the worktree, check `docs/business/` (`product-vision.md`, `business-model.md`, `decisions.md`, etc.) for relevance to this plan. If the plan changes pricing, tiers, seats, roles, or positioning, update the relevant docs/business file(s) and explain the impact to the user in chat before proceeding to implementation.

### 2. Set up isolated workspace
Use the `EnterWorktree` tool with name `feat/<NN>-<slug>` (e.g. `feat/02-better-auth-poc`).

After `EnterWorktree` completes, run `rm -rf .claire/` from the repo root to remove the stale directory left by a known typo bug in the tool.

### 3. Implement with subagent-driven development

**Cross-check plan details against reality.** Before implementing, verify port numbers, file paths, and config keys the plan mentions against the actual project files. Hardcoded values in plans (e.g. a port like `8788`) may be stale — always confirm against `wrangler.toml`, `vite.config.ts`, etc. before using them.

**Surface ambiguous values before building.** If the plan uses hedging language ("or", "e.g.", "approximately", "business hours range") for a concrete value that will affect the UI or API contract, treat it as an open question. Ask the user to decide before dispatching any subagent. Do not let subagents pick the value silently.

Invoke `superpowers:subagent-driven-development`. Provide subagents with:
- Full plan text (copy verbatim — don't make subagents read the file)
- Tech stack context (see below)
- Architecture rules (read `docs/tech/architecture.md` and pass the relevant sections — subagents must follow the module conventions, route factory pattern, and file structure defined there)
- **UI token rules** (for any UI work — read `docs/tech/ui-tokens.md` and pass the token class reference; subagents must use named token classes, never raw values, inline styles, or `var(--...)` in JSX)
- **Design source** (for any UI work — if the plan references `docs/design/`, read the relevant source file(s) (e.g. `docs/design/hi-fi/design_handoff_keshi/source/dashboard-calendar.jsx`) and pass the component JSX verbatim so subagents can match the exact layout, spacing, and interaction patterns. Never let subagents invent the UI from the plan text alone.)
- Security constraints (summarise from `.claude/SECURITY.md`)
- Any Cloudflare resource IDs / env values needed

**Architecture docs are read-only source of truth.** Subagents must adapt code to match the architecture — never edit `docs/tech/architecture.md` or any other doc file to match the implementation.

### 4. Two-stage review (automatic, don't skip)

**Stage A — Spec compliance (blocking)**

1. Extract the full `## Acceptance criteria` section from the plan file verbatim.
2. For each `- [ ]` item, do one of the following to verify it is actually satisfied:
   - Read the relevant source file(s) and confirm the code is present and correct, OR
   - Run a command (test, curl, grep) that produces observable evidence it works.
3. Produce an explicit checklist — one line per criterion — using ✅ or ❌:
   ```
   ✅ `auth-schema.ts` generated and committed
   ❌ `withTenant` RLS smoke-test: table not found in migration
   …
   ```
4. **If any item is ❌**, stop the review, fix the gap (return to Step 3 with a targeted subagent), then re-run this checklist from scratch. Do NOT proceed to Stage B until every item is ✅.

**Stage B — Code quality (non-blocking on Minor)**

- Fix Critical and Important issues before merge.
- Note Minor issues only — do not block on them.

### 5. Verify tests pass
`npm run test` from repo root must exit 0 before merge.

### 6. Fix docs drift (mandatory)

Compare what was just implemented against the relevant `docs/tech/` files (architecture, data model, auth, integrations, etc.).

For each doc that diverges from the implementation:
1. List the specific discrepancy (one line each).
2. Update the doc file in-place to match what was built — no new files, no placeholders.
3. Commit all doc changes: `docs: update <filename> to reflect plan <NN> implementation`.

**What counts as drift:**
- A new table, column, or relationship not reflected in `docs/tech/data-model.md`
- A new or changed API endpoint not reflected in `docs/tech/architecture.md`
- Auth or tenancy behaviour that diverges from `docs/tech/auth-and-tenancy.md`
- A new integration not documented in `docs/tech/integrations.md`
- Any "TBD" or placeholder that the implementation now resolves

If no drift is found, state "No docs drift detected" and move on.

### 7. Merge and clean up
Invoke `superpowers:finishing-a-development-branch` → choose "Merge back to master locally".

### 8. Stamp the plan as done and move it
Add `**Status: ✅ Completed — <YYYY-MM-DD>**` on the line immediately after the plan title (before `**Type:**`) in the plan file. Then `git mv` it into `docs/plans/done/` (from whichever folder it was in). Commit both changes together: `docs: mark plan <NN> complete`.

### 9. Run refactor interview
Invoke the `refactor` skill. This surfaces cleanup opportunities from the just-merged work before moving on.

### 10. Report completion
State which plan was completed and what the next plan is (with any prerequisites to check).

---

## Tech stack (provide to every subagent)

| Layer | Choice |
|---|---|
| API | Hono 4.x on Cloudflare Workers (`workerd` — no Node.js APIs) |
| UI | React 19 + TanStack Router + Vite + Tailwind CSS v4 |
| Forms | TanStack Form (`@tanstack/react-form`) — do NOT use RHF or Formik |
| Database | Neon PostgreSQL via `@neondatabase/serverless` HTTP transport only |
| ORM | Drizzle ORM (`drizzle-orm/neon-http`) |
| Auth | Better Auth + organization plugin |
| Date/timezone | Luxon |
| SMS | Twilio REST API |
| Email | Resend |
| Cache / Rate limit | Cloudflare KV (`KESHI_CACHE`) |
| Storage | Cloudflare R2 (`STORAGE_BUCKET` → `keshi-storage-dev` in dev) |
| Worker name | `keshi-api` |

## UI styling rules (enforce for every UI subagent)

These are hard constraints from `CLAUDE.md` and `docs/tech/ui-tokens.md`:

- **No inline styles.** Never use `style={{...}}` in JSX.
- **No raw values.** Never write `#1B1B1B`, `oklch(...)`, `var(--something)`, or `bg-[#...]` in JSX. Use named token classes only.
- **Use the token class names** from `docs/tech/ui-tokens.md`. Key mappings:
  - Surfaces: `bg-background`, `bg-surface-wash`, `bg-surface-cream`, `bg-surface-mute`
  - Ink: `text-ink-primary`, `text-ink-secondary`, `text-ink-tertiary`, `text-ink-inverse`
  - Accent: `text-accent-umber`, `bg-accent-umber`, `bg-accent-soft`
  - Semantic: `text-danger`, `text-positive`, `text-warning`
  - Borders: `border-hairline`
  - Radii: `rounded-lg` (6px inputs), `rounded-card` (10px panels), `rounded-sheet` (16px modals), `rounded-pill` (buttons)
  - Shadows: `shadow-1` (cards), `shadow-popover` (modals/dropdowns)
  - Fonts: `font-sans`, `font-serif`, `font-mono`
- **Typography rules:** `font-serif` on headings only; `font-sans` for all in-product text (inputs, labels, nav, body); `font-mono` for eyebrow labels only.

## Key security constraints (summarise for subagents)

- All domain table queries must use `withTenant(db, orgId, query)` — never `db.select()` directly (uses `db.batch()` internally; `db.transaction()` is not supported on neon-http)
- Every domain table needs RLS (`ENABLE ROW LEVEL SECURITY`, `FORCE ROW LEVEL SECURITY`, `tenant_isolation` policy)
- All routes under `/api/` must be on the authenticated router (auth middleware applied)
- Public routes only under `/public/:slug/`
- Owner-only resources (services, staff, settings, billing) must check `c.var.role === 'owner'`
- All request bodies validated through Zod schemas before business logic
- Secrets (`GOOGLE_CLIENT_SECRET`, `BETTER_AUTH_SECRET`, API keys, etc.) go in `.dev.vars` only — NEVER in `wrangler.toml [vars]`. Even if the plan says "declare in wrangler.toml", only commit non-sensitive config (e.g. `ENVIRONMENT`) there. `.dev.vars` is gitignored and overrides wrangler.toml at runtime.
- No hardcoded secrets — all from `c.env.*`
- OAuth `callbackURL` in a split API/UI app must be absolute — use `${window.location.origin}/path` (resolves to the UI origin in the browser). A relative path like `/dashboard` resolves against the API base URL, causing a 404.
- Owner-initiated SMS must append `"\n\nReply STOP to opt out"` server-side

## Appointment status enum
`confirmed | done | no_show | cancelled` — no `pending` state. All new bookings are immediately `confirmed`.

## HITL vs AFK
The plan file itself declares its type on the `**Type:**` line — always read it from there. Never assume a plan's type from memory or a hardcoded list.
