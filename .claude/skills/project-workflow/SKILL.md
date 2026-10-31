---
name: project-workflow
description: >-
  The discovery and planning loop for work that is still foggy — you cannot yet
  name the questions, write a plan, or say which files to touch. Produces a spec
  with numbered user stories, slices it into phases for roadmap.md, and routes
  each slice to feature-workflow. Use when the user types "/project-workflow",
  when starting a new project, or when a request is too vague for feature-workflow
  to consume. Once the spec and slices exist, hand off to feature-workflow per slice.
---

# Project Workflow

Steps 1–4. These produce the spec and phased roadmap that `feature-workflow`
(steps 5–12) consumes. If you already have a clear spec and know which files to
touch, skip to `feature-workflow` directly.

**This workflow answers: "What are we building and in what order?"**
**Feature workflow answers: "How do we build this specific piece?"**

**On entry:** The user may provide anything from a vague idea to a detailed
requirements list. Do not wait for a complete spec. Start by reading whatever
they gave you, identify what is unclear or missing, and ask questions to fill
the gaps. The user does not need to know the "right" format — that is your job.
Meet them where they are:

| User provides | You do |
|---|---|
| A vague idea ("I want to automate tasks") | Ask what, who, where, when, how they'd verify it |
| A paragraph describing the problem | Confirm your understanding, ask about edge cases and scope |
| A bullet list of requirements | Validate each one is testable, ask about what's missing |
| A full spec | Check it against step 2's rules (numbered stories, "Rejects when", no open questions), fix gaps, then step 3 |

---

## Gate

| Situation | Action |
|---|---|
| Goal is foggy, can't name the questions yet | Start at step 1 |
| Goal is clear but no spec or user stories | Start at step 2 |
| Spec exists but work isn't sliced into phases | Check the spec against step 2's rules, then step 3 |
| Phases exist in roadmap.md, ready to build | Skip — go to `feature-workflow` step 5 |

---

## 1. Understand — clarify what the user actually wants

Before writing anything, extract clarity. Ask questions until you can state the
goal in one sentence that the user agrees with.

**Do:**
- Ask what problem the user is solving, not what feature they want
- Ask who uses it and what they see
- Ask what already exists (existing code, APIs, services, accounts)
- Ask what "done" looks like — how would they verify it works?
- Challenge assumptions that sound like solutions disguised as requirements

**Don't:**
- Guess and build — a wrong guess costs more than a question
- Ask all questions at once — group by topic, wait for answers
- Widen scope — if the user says "just X", that's the scope

**Output:** A one-sentence goal statement the user confirms.

Example:
> **Goal:** When a matter changes stage in Clio, automatically create the
> appropriate task assigned to the responsible attorney, within 30 seconds.

---

## 2. Spec — write numbered user stories with acceptance criteria

Turn the confirmed goal into a spec. Each user story is numbered because
`feature-workflow` step 8 uses these numbers as the E2E test matrix.

### Format

```markdown
# Spec: <Project Name>

## Goal
<The one-sentence goal from step 1.>

## Assumptions
- <What's already true — existing accounts, APIs, infrastructure>
- <What the user confirmed in step 1>

## User Stories

### US-1: <Action the user or system takes>
**As** <who>, **I want** <what>, **so that** <why>.

**Accepts when:**
- [ ] <Observable, testable condition>
- [ ] <Observable, testable condition>

**Rejects when:**
- [ ] <What should NOT happen>

### US-2: <Next action>
...

## Out of Scope
- <Things explicitly not built, and why>

## Open Questions
- <Anything unresolved — block on these before step 3>
```

### Rules

- **Every story is testable.** Each story becomes one E2E scenario in
  `feature-workflow` step 8, and each "Accepts when" and "Rejects when"
  condition becomes one assertion in it. If you can't test it, rewrite it.
- **Number every story.** `feature-workflow` references them by number.
- **Include "Rejects when."** The happy path is half the spec. What happens on
  bad input, duplicate events, auth failure, timeout?
- **List what's out of scope.** This feeds the "Deliberately not built" line in
  `roadmap.md` phases.
- **Resolve open questions before step 3.** Don't slice foggy work — you'll
  re-slice it later.

---

## 3. Slice — break the spec into ordered phases

Take the numbered user stories and group them into phases for `roadmap.md`.
Each phase must be independently shippable and verifiable.

### How to slice

**Slice by dependency, not by component.** The question is *"what must exist
before this can work?"* — not *"what layer is this?"*

Bad slicing (by layer):
```
Phase 0: Database schema        ← can't verify, nothing uses it
Phase 1: API endpoints          ← can't verify, no consumer
Phase 2: UI                     ← finally verifiable
```

Good slicing (by user story):
```
Phase 0: US-1, US-2 (webhook → task creation)  ← verifiable end-to-end
Phase 1: US-3, US-4 (task templates)            ← builds on Phase 0
Phase 2: US-5 (dashboard)                       ← builds on Phase 1
```

### Rules

- **Each phase maps to at least one user story.** A phase with no user story
  has no test scenario and no "done when" condition.
- **Phase 0 is the smallest thing that proves the architecture works.** Not a
  skeleton, not scaffolding — a real user-visible outcome, however small.
- **One phase is `NEXT`. Everything else is `LATER`.** Never work on two phases
  at once.
- **Each phase states what it deliberately does NOT build** and which later
  phase owns it. This is what ponytail (in `feature-workflow`) enforces during
  the build — if it's not in this phase's stories, it gets cut.

### Output — populate `roadmap.md`

Fill in the template with real content:

```markdown
# Roadmap

## Part one — Core task automation

### Phase 0 — Webhook listener NEXT

Proves the core loop: Clio webhook → Convex → task created in Clio.
Covers US-1, US-2.

- A stage change on a matter creates one task assigned to the responsible attorney
- Duplicate stage changes do not create duplicate tasks

**Done when:** a stage change on a test matter creates exactly one correctly
assigned task within 30 seconds.

Deliberately not built, and why: task templates (Phase 1 owns it),
dashboard (Phase 2 owns it).

### Phase 1 — Task templates LATER

Covers US-3, US-4.

- Attorneys can define task templates per stage
- Templates support due-date offsets relative to the stage change date

**Done when:** a stage change creates a task using the matching template
with the correct due date.

Deliberately not built, and why: dashboard (Phase 2 owns it).

### Phase 2 — Dashboard LATER

Covers US-5.

- Web UI showing recent runs, failures, and task counts per matter
```

---

## 4. Route — hand the first phase to feature-workflow

Once `roadmap.md` is populated:

1. Confirm the user agrees with the phases and their order
2. Identify the phase marked `NEXT`
3. Hand it to `feature-workflow` starting at step 5

**The handoff:**

> The phase marked NEXT is **Phase 0 — Webhook listener**.
> It covers US-1 and US-2. Entering `feature-workflow` at step 5.

From here, `feature-workflow` takes over:
- Step 5 turns this phase into an implementation plan
- Steps 6–11 build, test, review, and merge it
- Step 12 updates `roadmap.md` (marks DONE, promotes NEXT)
- Then this cycle repeats for the next phase

---

## The full picture

```
project-workflow                    feature-workflow
━━━━━━━━━━━━━━━━                    ━━━━━━━━━━━━━━━━
1. Understand                       5.  Plan
2. Spec (user stories)              6.  Cut (ponytail gate)
3. Slice (populate roadmap)  ──→    7.  Code + TDD
4. Route NEXT phase          ──→    8.  E2E test
                                    9.  Verify
                                    10. Review (2–3 passes)
                             ←──    11. Merge
                                    12. Update roadmap ──→ next NEXT phase
                             ──→        (loop back to step 5)
```

---

## Failure modes

- **Skipping step 1.** Building a spec from assumptions produces a spec you
  rewrite after the first phase ships. One round of questions costs less.
- **User stories without "rejects when."** The happy path works. Everything
  else is discovered in production.
- **Slicing by layer instead of by story.** Three phases of invisible
  infrastructure before anything is verifiable.
- **Phase 0 is too big.** If it takes more than a few days, slice smaller.
  Phase 0's job is to prove the architecture, not ship the product.
- **Open questions carried into slicing.** Foggy stories produce foggy phases
  that get re-sliced mid-build.
