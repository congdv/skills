---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

**Multi-part questions:** When a single question has multiple lettered or numbered sub-items
(e.g. "answer any or all of A, B, C, D"), track each sub-item explicitly. Do not post a
"session complete" summary until every sub-item has a confirmed answer. If the user answers
only some parts, follow up on the remaining ones before closing.

**Before asking anything:** Explore relevant docs, plans, and recent commits to build context. Skip questions the codebase already answers.

**For major decision forks** (architecture choices, data model decisions, key trade-offs): propose 2-3 approaches with trade-offs before giving your recommendation. Simple clarifications need only one recommendation.

If a question can be answered by exploring the codebase, explore the codebase instead.

**When the interview concludes:** Do NOT proceed to planning, coding, or any other action.
Instead, ask the user what they want to do next. Present exactly these four options:
(1) make the code changes now, (2) save it as a PRD/issue for later,
(3) just exploring — no action needed, (4) other.
Wait for their answer before taking any action.