---
title: "Lifting Diary — Architecture as the Input"
description: "Why architecture knowledge is the key ingredient in AI-assisted development — and how the /docs folder pattern kept Claude Code consistent across 22 prompt sessions."
date: 2026-04-20
tags:
  - claude-code
  - software-architecture
  - developer-tools
  - case-study
status: draft
---

# Architecture as the Input

> The quality of Claude's output is bounded by the quality of the constraints you give it. The constraints come from your architecture knowledge.

Claude Code is fast at generating code. What it is not good at — across multiple sessions, across a growing codebase — is staying consistent without explicit guidance. Each session is a fresh context. Without something anchoring its choices, Claude will make decisions that are individually reasonable but collectively incoherent: one page fetches data in a Server Component, another uses `useEffect`; one action validates with Zod, another doesn't bother.

The solution used throughout this project was the `/docs` folder: a set of Markdown files written *before* the features they governed, each one a plain-English constraint that `CLAUDE.md` told Claude to read before writing any code.

---

## The /docs Folder Pattern

Six constraint documents were written across the project. Each one was written before the code it governed — not as documentation of what was already built, but as a specification of what was allowed:

| Document | Core rule | What it prevented |
|---|---|---|
| `ui.md` | Only shadcn/ui components — no custom UI primitives | Component sprawl; inconsistent styles across pages |
| `data-fetching.md` | Server Components only; Drizzle ORM only; queries scoped to authenticated user | Route Handlers, `useEffect` fetching, raw SQL, data leaks between users |
| `data-mutations.md` | Server Actions with Zod validation; no `redirect()` inside actions | Unvalidated input, client-side `fetch()`, server-side redirect bugs |
| `auth.md` | Clerk everywhere — consistent `auth()` calls in actions and components | Missing auth checks; inconsistent sign-in patterns |
| `routing.md` | All authenticated routes under `/dashboard`; protection via middleware | Unprotected routes; inconsistent URL structure |
| `server-components.md` | `params` must be awaited (Next.js 15 requirement) | Runtime errors from unawaited promise params |

The order matters. `ui.md` was written before the first page was built. `data-fetching.md` was written before the first query. `data-mutations.md` was written before the first form. The constraint had to precede the code for it to shape the code.

---

## What Happens Without the Contract

Two concrete examples from this project where the contract was either absent or violated:

### The redirect() mistake

During session 18, the create-workout Server Action was implemented with a `redirect()` call inside it — a natural choice if you're not aware of the constraint. The redirect worked, but it caused a `NEXT_REDIRECT` error to surface as an unhandled exception in the client.

The correction was: "the redirect should be done client side, not within the server action." That worked. Then, critically, the constraint was *written into the doc*:

> *"Add into `docs/data-mutations.md` a rule that says the `redirect()` function should not be used within server actions. Redirects should instead be done client side after the call to a server action resolves."*

Without that third step — updating the doc — the same mistake would be available for the next session.

### The Next.js 15 params requirement

Session 20 introduced a dynamic route (`/dashboard/workout/[workoutId]`). Next.js 15 made `params` a Promise — they must be awaited before use. Claude didn't know this without being told.

The solution was `server-components.md`:

> *"Create a new `docs/server-components.md` file outlining that accessing params MUST be awaited because this is a Next.js 15 project where params HAVE to be awaited as it's a promise."*

One document. All future dynamic routes in the project then got this right automatically, because Claude read the doc before generating the page.

---

## When Claude Still Needs a Developer

The timezone bug in session 15 is the clearest example of where architectural knowledge was required — not to write code, but to identify the problem.

The symptom: clicking a date in the calendar consistently selected the *previous* day. Clicking the 25th showed the 24th in the URL. The data loaded correctly when the date was typed manually, which ruled out the data layer.

The cause: `Date` objects created in the browser include the local timezone offset. When serialised and parsed on the server, the offset was being lost, shifting the date back by the UTC difference. This is a well-known JavaScript timezone trap — and identifying it required knowing that the issue was at the client/server boundary, not in the calendar component itself.

Once the root cause was named, Claude implemented the fix correctly. But the diagnosis — knowing *where* to look and *why* the boundary mattered — came from the developer.

![The date navigator after the timezone fix — the selected date in the UI and the date in the URL now match](img/8_Show_the%20Date_Corectly.PNG)

---

## The Productivity Model

| Claude handles | Developer must own |
|---|---|
| Boilerplate: layouts, forms, routes, schema files | Architectural decisions: what pattern to use and why |
| Repetitive patterns: adding a new page following established conventions | Constraint documents: defining what is and isn't allowed |
| Debugging: implementing a fix once the cause is identified | Root cause identification: especially at system boundaries |
| Refactoring: updating multiple files to follow a new rule | Knowing when something is wrong: catching mistakes before they become patterns |
| Documentation: generating handover docs from a well-structured codebase | Reviewing generated output: Claude does not flag its own errors |

The final session — generating the full client handover document from a review of the entire codebase — was possible precisely because the architecture was clean and the standards had held. A codebase with drift, inconsistent patterns, and undocumented decisions produces a document that reflects the drift. Garbage in, garbage out.

---

Architecture knowledge is not something Claude can replace — it is the input that makes AI-assisted development reliable. The Prompt Log shows how these principles played out across all twelve build phases, with the exact prompts used at each stage and the reasoning behind each decision.

[Prompt Log →](prompt-log.md)
