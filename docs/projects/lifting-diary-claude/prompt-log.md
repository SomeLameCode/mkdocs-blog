---
title: "Lifting Diary — Prompt Log"
description: "Annotated walkthrough of all 12 build phases — the prompts that drove each phase and why each approach was chosen."
date: 2026-04-20
tags:
  - claude-code
  - developer-tools
  - workflow
  - prompt-engineering
status: draft
---

# Prompt Log

An annotated walkthrough of the build from first prompt to handover document. Prompts are curated — the full set is in the source repository. The purpose here is to show the *pattern* behind each phase, not an exhaustive transcript.

---

## The Build in 12 Phases

| Phase | Mode | Representative prompt | Why this approach |
|---|---|---|---|
| **01 Setup** | Default | `/terminal-setup` · `/theme` · `/config` · `/model` · `/init` | Environment configured before any code. `/init` generated the initial `CLAUDE.md` from the codebase — the session contract that governs every subsequent prompt |
| **02 Auth** | Default → Edit | *"Is it possible to launch sign in and sign up with Clerk via a modal? Do not make any updates to the code."* | Default mode first — explore feasibility before committing. The explicit "do not update" instruction is the guard that prevents premature implementation |
| **03 Schema** | Plan (×3 rounds) | *"Plan a table schema to log workouts... make sure this is normalized. The schema must be created using Drizzle ORM for a Postgres DB hosted on Neon."* | Schema is the foundation. Three rounds of plan mode: initial design, then two targeted refinement prompts removing columns and renaming fields. No code was written until all three iterations were agreed |
| **04 Seed data** | Default | *"List all available tables within the liftingdiarycourse db on Neon"* → *"Generate example data for user id [X]. Do not insert yet."* → *"This looks great. Now insert."* | MCP (Neon) enabled Claude to query the database directly. The generate-review-insert sequence mirrors the plan-review-implement pattern: always agree before acting |
| **05 UI scaffold** | Edit | *"Create a /dashboard page with a datepicker set to the current date... ONLY generate the UI for this page. DO NOT generate any data fetching or server side code just yet."* | UI separated from data deliberately. Reviewing the interface before wiring up data fetching prevents two concerns from being entangled at first draft |
| **06 Standards → data** | Edit | *"Create a new `docs/data-fetching.md` file... ALL data fetching MUST be done via Server Components... then implement the data fetching for the dashboard page"* | Standards written first, then implementation. Claude reads the doc (per `CLAUDE.md`) before writing the query — so the first implementation already follows the rule |
| **07 Bug fix** | Default → Edit | *"Outline a plan on how to fix the issue... @src/app/dashboard/page.tsx @src/app/dashboard/calendar-client.tsx — whenever I select a date, the previous date is selected. Could this be a timezone issue?"* | `@file` references focus the context window. Default mode for diagnosis (with the developer's timezone hypothesis already included), then edit mode once the plan was reviewed |
| **08 Branch merge** | Plan → Edit | *"Give me a plan on how to merge the dashboard-page branch into main, resolve any merge issues, and create a new branch called create-workout-page. Do not implement anything just yet."* → *"Ok great. Now implement this plan."* | Two-step: plan, review, implement. Branch operations touch git history — the cost of a mistake is higher than most code changes, so plan mode is always worth it here |
| **09 More standards** | Edit | *"Create a new `auth.md` documentation file... this app uses Clerk for authentication."* · *"Create a `data-mutations.md` file... all data mutations MUST be done via Server Actions... ALL server actions MUST validate via Zod."* | Standards expanded as new concerns arose. `auth.md` and `data-mutations.md` were written before any auth-guarded actions or mutation code — the constraint preceded the implementation |
| **10 Features + correction** | Edit | *"Create a new page at /dashboard/workout/new with a form to create a new workout."* → *"The redirect should be done client side, not within the server action."* → *"Add into `docs/data-mutations.md` a rule that says `redirect()` should not be used within server actions."* | Three-step correction loop: implement → catch drift → update the doc. The third step is critical — without updating `data-mutations.md`, the same mistake is available to every future session |
| **11 Routing** | Edit | *"Create a new `routing.md` documentation file... all routes should be accessed via /dashboard... route protection should be done via Next.js middleware."* | Same standards-first pattern, applied to routing. Document written before the middleware was implemented |
| **12 Handover doc** | Default | *"Review the entire project — architecture, structure, dependencies, workflows, and design decisions. Generate a professional client-facing Design & Handover Document in Markdown format."* | The final output: a complete handover document covering setup, architecture, known limitations, and future improvements — entirely generated by Claude. This was only reliable because the architecture had stayed consistent throughout |

---

## What the Log Shows

A few patterns are visible across all 12 phases:

**The standard precedes the feature.** In phases 06, 09, and 11, the constraint document was written in the same prompt (or the one immediately before) as the feature it governed. Claude read it, then implemented. The implementation followed the rule from the first line.

**Corrections feed back into docs.** Phase 10 is the clearest example: a mistake was caught, corrected, and then *written into the constraint doc*. Without that third step, the correction only applies to the current session.

**Plan mode absorbs the high-risk operations.** Schema design (phase 03), bug investigation (phase 07), and branch merges (phase 08) all used plan mode first. These are the cases where a wrong first move is expensive to undo.

**The handover document validates the whole approach.** Phase 12 generated a 600-line architecture, setup, and limitations document from a cold review of the codebase. It was accurate and complete. Claude was given no summary and no prior context — it derived the architecture, the patterns, the standards, and the known limitations entirely from reading the code. That is only possible because the code was coherent enough to be read that way. Consistent patterns produce a readable codebase; a readable codebase produces a reliable handover. A messy codebase produces a messy document.

---

The twelve phases show a consistent pattern: plan before implementing, write standards before writing features, and feed corrections back into the constraint documents. These disciplines are not specific to Lifting Diary — they generalise to any AI-assisted development where consistency across sessions matters.

[← Back to Lifting Diary](../lifting-diary-claude.md)
