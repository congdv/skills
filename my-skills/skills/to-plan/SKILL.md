---
name: to-plan
description: Use when the user wants to convert a spec, PRD, feature description, or requirement into a numbered implementation plan file saved on disk. Asks whether to save under docs/plans/to-do/ or docs/plans/backlog/ (or accepts an explicit target directory); auto-detects existing plan numbering and file format across both folders; checks the drafted plan against other open plans for conflicts before writing, re-grilling the user if one is found.
---

# To Plan

Convert a spec or feature description into a numbered implementation plan file, matching the naming pattern and section structure of plans that already exist in the target directory.

## Process

### 1. Determine the target directory

- If the user passed a path as an argument, use it directly and skip the folder question below.
- Otherwise, the base directory is `docs/plans/` in the project root, which holds two subfolders:
  - `docs/plans/to-do/` — plans ready to be implemented soon
  - `docs/plans/backlog/` — captured ideas, not yet prioritized
- **Ask the user which folder to save the new plan in** (`to-do` or `backlog`) before doing anything else. Do this first — the answer determines which folder's plans get scanned for template/structure below.
- Confirm the chosen folder exists: `ls docs/plans/<folder>`

### 2. Scan existing plans

Plan numbers are **globally unique across both folders** — a number is only used once, whether the plan lives in `to-do/` or `backlog/`. To compute the next number, scan both:

```bash
ls -1 docs/plans/to-do docs/plans/backlog
```

Extract from the combined set of filenames:

| What to detect | How |
|---|---|
| Naming pattern | e.g. `{NN}-{slug}.md`, `plan-{NN}.md` |
| Highest integer | strip letters (e.g. `03a` → `3`), find max across **both folders** |
| Zero-padding width | count digits in existing numbers (`01` → 2) |

Then read the **most recent plan file** (highest number) **within the chosen target folder** to extract its exact section headings and structure. If the target folder has no plans yet (e.g. a fresh `backlog/`), fall back to the most recent plan in the other folder instead — the two folders share the same template.

While scanning, also skim the **What to build** / **Goal** section (or first paragraph) of every plan in `to-do/` and `backlog/` and note, in a line or two each, what area/feature it touches. This is the "open plans" inventory used for conflict-checking in Step 5 — build it now while the files are already open, rather than re-reading them later.

`docs/plans/_deprecated/` is an archive of superseded plans and is never scanned for numbering, templates, or conflicts — it holds closed/superseded work, not open plans.

### 3. Gather source material

Work from whatever is in the conversation — PRD, user story, feature description, issue body. If the user passed a file path or issue number as an argument, read or fetch it first.

### 4. Draft the plan content

Mirror the section structure of the most recent plan. Two common formats:

**Standard format** (used in most plans in this project):
```markdown
# {NN} — {Title}

**Type:** AFK | HITL

## What to build

{End-to-end description of behavior — not layer-by-layer implementation.}

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- {plan-slug} or None — can start immediately
```

**Extended format** (used when tasks need detailed implementation notes):
```markdown
# Plan {NN} — {Title}

## Goal

{One-paragraph goal statement.}

## Tasks

### 1. Task Name

{Detailed implementation notes, code snippets if encoding a decision.}

### 2. Task Name

{Details.}

## Acceptance criteria

- [ ] Criterion 1

## Blocked by

- {plan-slug} or None — can start immediately
```

Choose whichever format matches the majority of existing plans in the directory. When in doubt, use the standard format — it's simpler.

### 5. Check for conflicts with open plans

Compare the drafted plan against the open-plans inventory gathered in Step 2 (everything in `to-do/` and `backlog/`, excluding `_deprecated/`). Look specifically for:

- **Overlapping scope**: two plans changing the same feature/area/table/route in ways that aren't obviously compatible (e.g. this plan and an existing one both redesign the same settings page, or both touch the same schema field with different shapes).
- **Contradictory decisions**: this plan assumes something an open plan explicitly does differently (e.g. this plan bills per-seat monthly, an open plan already assumes annual-only billing).
- **Broken or missing sequencing**: this plan should be `Blocked by` an open plan (or vice versa) but isn't, or an existing plan's `Blocked by` chain would become inconsistent once this plan exists.
- **Duplicate work**: an open plan already covers what this plan is proposing.

If nothing conflicts, say so briefly and move on to Step 6.

If something conflicts, **stop before quizzing the user on scope** — invoke `/grill-me`, framing the specific conflict as the topic: name the two plans, state exactly what's contradictory or overlapping, and drive to one of:
- narrowing this plan's scope to remove the overlap,
- adding/correcting a `Blocked by` relationship,
- or revising the conflicting open plan (only with the user's explicit sign-off — never silently edit or move another plan to `_deprecated/`).

Do not proceed to Step 6 until the conflict is resolved and the user has confirmed the resolution.

### 6. Quiz the user

Present the draft plan and ask:

- Does the scope feel right — too broad, too narrow?
- Should this be split into multiple plans?
- Are blocked-by dependencies correct?
- Is the type (AFK / HITL) right?

Iterate until the user approves. **Do not write the file until approved.**

### 7. Compute the file name

- **Next number** = highest existing integer across both `to-do/` and `backlog/`, + 1
- **Zero-padding** = match existing width (e.g. `28` exists → next is `29`, not `029`)
- **Slug** = kebab-case of the plan title, lowercase
- **Full path**: `docs/plans/{folder}/{NN}-{slug}.md`, where `{folder}` is whichever the user picked in Step 1

Examples (numbers are illustrative, not simultaneous — each is "current max + 1" at the time it's created):
```
docs/plans/to-do/106-stripe-webhook-handler.md
docs/plans/backlog/107-manager-invite-flow.md
```

### 8. Write the file

Write the approved plan to the computed path. Confirm the path to the user.

### 9. Update affected architecture docs

If the plan describes changes to the tech stack, data model, or system structure, check
`docs/tech/` for files that need updating in the same session:

- `docs/tech/architecture.md` — monorepo structure, tech stack table, request lifecycle
- `docs/tech/data-model.md` — table definitions, column renames
- `docs/tech/auth-and-tenancy.md` — auth flow changes

Update any file whose content would be wrong or misleading after the plan is implemented.
Do this before ending the session — don't leave docs describing a stack that no longer matches
the plan.

## Numbering Rules

- Parse all filenames in **both `to-do/` and `backlog/`** for a leading integer (ignore sub-letters like `03a` — those are lateral additions, not sequential slots)
- Sort numerically, take the max integer across both folders combined
- New sequential plan = max + 1, regardless of which folder it's saved to
- Never reuse or skip numbers — this holds across the whole `docs/plans/` tree, not per-folder
- Sub-letters (`03a`) are only for plans that fork from an existing slot; new plans from `/to-plan` always use plain integers
- A plan in one folder may be `Blocked by` a plan number that lives in the other folder — that's expected, since numbering is shared

## What makes a good plan

- **Behavior, not layers**: describe end-to-end user-visible behavior, not "add a column" or "write a query"
- **Acceptance criteria are verifiable**: each one can be checked by running the app or a test
- **Blocked by is specific**: reference the plan slug or number, not vague dependencies
- **Type is honest**: HITL means a human must make a decision or review before the plan can complete; AFK means an agent can implement and merge without interruption
