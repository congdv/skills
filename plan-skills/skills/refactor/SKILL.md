---
name: refactor
description: Interview the user to surface refactoring opportunities after completing a plan. Reads the plan file and recent git diff to ask specific, grounded questions, then produces a prioritized list of refactor suggestions with effort/impact ratings. Use when the user has just finished implementing a plan and wants to find cleanup work, or when user says "refactor", "cleanup after plan", or "what can we improve".
---

# Refactor Interview

## Process

### Step 1 — Load context

Run these in parallel:

```bash
# Most recently completed plan (only to-do/ plans get implemented; backlog/ is not-yet-ready)
ls docs/plans/to-do/*.md | sort | tail -5

# Recent changes
git diff main...HEAD --stat
git diff main...HEAD
```

Read the plan file the user just finished. Skim the diff to understand what was added/changed.

**If the diff touches any UI files** (`.tsx` or files in `apps/web/`):
- Read `.claude/UI.md`
- For every file touched in the diff, check its line count (`wc -l`). Flag any that exceed the thresholds from UI.md Rule 1: >300 lines in a component, >500 lines in a page. Add flagged files to the Step 3 list automatically — do not wait for the user to surface them in the interview.

If the diff touches `ui/src/` files, also read `docs/tech/ui-tokens.md`. Pass the full token class reference to every UI-related agent — never let agents invent class names.

### Step 2 — Interview (6 rounds)

**Early-exit:** If Step 1 already surfaces clear, specific opportunities (e.g., obvious duplication visible in the diff), present them immediately and ask: "I already spotted a few things from reading the code — want me to list them now, or run through the full interview first?" If the user says list them, skip straight to Step 3.

Ask **2–3 focused questions per round**. Wait for the user's full answer before moving to the next round. Do not dump all questions at once.

**Round 1 — Duplication**
- Did you write similar logic more than once? Where?
- Are there any helper functions that nearly duplicate each other?
- Is any validation or transformation repeated across routes or modules?

**Round 2 — Abstraction leaks**
- Does any layer know too much about another (e.g., route handler doing DB logic, service layer building HTTP responses)?
- Are there any `as unknown as X` casts, overly wide types, or places you had to fight the type system?
- Did anything end up in the wrong file because there was no good home for it?

**Round 3 — Naming & clarity**
- Any function or variable names you're not happy with?
- Are there any boolean flags or magic values that should be named constants or enums?
- Would a new reader understand what each module does from its name alone?

**Round 4 — Security & Types**
- Did every new route handler validate its request body through a Zod schema before touching business logic?
- Are there any `any` types, missing return type annotations, or places where you widened a type to make it compile?
- For owner-only routes: does every one check `c.var.role === 'owner'` before executing? For domain table queries: does every one go through `withTenant`?
- Are there bare `catch (e)` blocks that swallow errors silently, or error paths that return inconsistent shapes?
- **Forms (UI only):** Does every new form use the project's required library (TanStack Form / `@tanstack/react-form`)? Do fields with required inputs show inline field-level errors (not just a single bottom-of-form message)?

**Round 5 — Performance**
- Any places where you load more data than you use (over-fetching)?
- Any N+1 query patterns — a query inside a loop, or separate fetches that could be batched?
- Any work that runs on every request but could be cached or computed once?
- Extra Neon round-trips per request (e.g. multiple `withTenant` calls that could share one `db.batch()`)?
- **UI only:** unnecessary re-renders (unstable props, inline objects/functions), large client bundle additions, `"use client"` pushed higher than needed, or first-paint data fetched in `useEffect` instead of a Server Component?

**Round 6 — Cost**
- Does this add a Twilio, Resend, Neon, or KV call per request, per booking, or per cron tick?
- Does it change SMS volume? Does it respect the Pro quota (300/month included, $0.02 overage) and the 1000/month hard cap? Is the free tier still email-only?
- Does any cron or list query scale with total rows across all orgs instead of per-org or per-day windows?

### Step 3 — Produce prioritized list

After all 6 rounds, output a ranked table:

```
## Refactor Opportunities

| # | Area | What to change | Effort | Impact |
|---|------|---------------|--------|--------|
| 1 | ...  | ...           | S/M/L  | H/M/L  |
| 2 | ...  | ...           | ...    | ...    |
```

- **Effort**: S = < 30 min, M = half day, L = multi-day
- **Impact**: H = correctness/security risk or major DX win, M = noticeable improvement, L = nice-to-have
- Sort by Impact DESC, then Effort ASC
- Only include items the user surfaced or confirmed — do not invent suggestions

## Rules

- Ask follow-ups if the user's answer is vague ("which file?", "can you show the pattern?")
- Skip a round if the user says nothing applies — don't pad
- Never suggest rewrites of things that are working correctly and clearly written
- Keep suggestions scoped to what was just built in the plan, not the whole codebase
- **UI agents must receive the token reference.** Before dispatching any agent that touches `ui/src/`, read `docs/tech/ui-tokens.md` and include the token-to-class mapping in the prompt. Missing this causes a full redo pass (wrong class names throughout).
