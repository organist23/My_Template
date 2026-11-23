---
name: project-workflow
description: >-
  The discovery and planning loop for work that is still foggy — you cannot yet
  name the questions, write a plan, or say which files to touch. Produces a spec
  with numbered user stories, slices it into phases for roadmap.md, and routes
  each slice to feature-workflow. Use when the user types "/project-workflow",
  when starting a new project, or when a request is too vague for feature-workflow
  to consume. Also use when an approved requirement is added, changed, removed,
  or conflicts with another. Once the spec and slices exist, hand off to
  feature-workflow per slice.
---

# Project Workflow

Steps 1–4. These produce the spec and phased roadmap that `feature-workflow`
(steps 5–12) consumes. If you already have a clear spec and know which files to
touch, skip to `feature-workflow` directly.

**This workflow answers: "What are we building and in what order?"**
**Feature workflow answers: "How do we build this specific piece?"**

**On entry:** The user may provide anything from a vague idea to a full spec.
Do not wait for a complete spec. Read what they gave you, then grill it (step
1). Whatever they already decided is a settled branch of the design tree. A
full spec is checked against step 2's rules, and each gap becomes a question in
a grilling round.

---

## Gate

| Situation | Action |
|---|---|
| Goal is foggy, can't name the questions yet | Start at step 1 |
| Goal is clear but no spec or user stories | Start at step 1 with the goal as a settled branch. Grill only the remaining branches |
| Spec exists but work isn't sliced into phases | Check the spec against step 2's rules. Gaps go to step 1 as a grilling round, then step 3 |
| Phases exist in roadmap.md, ready to build | Skip — go to `feature-workflow` step 5 |
| Spec approved, and a requirement is added, changed, removed, or conflicts, or a redesign, schema, API, or business-rule change | **Amend** below |

---

## 1. Understand — clarify what the user actually wants

Grill the user until you reach a shared understanding. This is the only step
that interviews. Step 2 writes from what was decided here.

**The design tree.** Every decision branches into the decisions that hang off
it. The roots:
- What problem the user is solving, not what feature they want
- Who uses it and what they see
- What already exists (code, APIs, services, accounts)
- What "done" looks like and how they would verify it

Step 2 needs these branches settled too, so grill them:
- Each story's "Rejects when": bad input, duplicates, auth failure, timeout
- Safeguard policies for consequential actions (soft or hard delete, retention,
  who confirms), from `feature-workflow` **Safeguards by risk**
- What is out of scope
- For a product with a UI: product type, audience, style, light or dark, density,
  screen widths, and any printed output with the official form it must match.
  Detect the stack from the repo yourself

**Rounds.** The frontier is every decision whose prerequisites are settled.
Ask the whole frontier in one round, then wait for answers. A question that
depends on another question still open in this round belongs to a later round.
Each round's answers move the frontier. Recompute it and ask the next round.

**Format.** Number each question and give your recommended answer. Word each
question so "yes" accepts the recommendation.

```
❓ **Q1** - **<question title>**: <question body, choices if any>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body>

➡️ <your recommended answer>
```

**Facts are yours, decisions are the user's.** When a question needs a fact
from the environment (files, schema, tools, existing data), dispatch a subagent
for anything beyond a quick read. Never ask the user for something you can find. Only
questions downstream of a running lookup wait for it. Put every decision to the
user and wait. Never answer one yourself.

**Also:**
- Challenge assumptions that sound like solutions disguised as requirements
- Do not widen scope. If the user says "just X", that is the scope
- A question talking cannot settle (how a screen should look or feel) needs
  something to react to. Generate a preview with the `ui-ux-pro-max` skill
  (`--design-system --preview`), or a throwaway prototype, and put it to the
  user in a grilling round. Do not re-ask it in words. Palette, style, and fonts
  are the user's decisions. A search that returns nothing falls back to the
  skill's defaults, and you say so

**Done when** the frontier is empty: every branch visited, nothing silently
assumed. Do not start step 2 until the user confirms the shared understanding.

**Output:** A one-sentence goal statement the user confirms.

Example:
> **Goal:** When a matter changes stage in Clio, automatically create the
> appropriate task assigned to the responsible attorney, within 30 seconds.

---

## 2. Spec — write numbered user stories with acceptance criteria

Turn the confirmed goal into a spec saved as `spec.md` at the project root.
Do not interview here. Write from the decisions grilled in step 1. A gap found
while writing goes back to step 1 as a grilling round.
Each user story is numbered because
`feature-workflow` step 8 uses these numbers as the E2E test matrix.

### Format

```markdown
# Spec: <Project Name>

## Goal
<The one-sentence goal from step 1.>

## Assumptions
- <What's already true — existing accounts, APIs, infrastructure>
- <What the user confirmed in step 1>

## Design references
- Design system: `design-system/<slug>/MASTER.md` (page overrides in `pages/`)
- Printed outputs: <form name> must match <reference file>
- Screen widths: <phone, desktop>

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

## Critical workflows
- CW-1: <name>: US-1 → US-3 → US-5

## Out of Scope
- <Things explicitly not built, and why>
```

### Rules

- **Every story is testable.** Each story becomes one E2E scenario in
  `feature-workflow` step 8, and each "Accepts when" and "Rejects when"
  condition becomes one assertion in it. If you can't test it, rewrite it.
- **Number every story. Numbers are permanent.** `feature-workflow`, test
  results, and `roadmap.md` reference them. Never renumber or reuse one. A
  changed story gets `Changed <date>: <reason>` under its heading. A removed
  story gets `Withdrawn <date>: <reason>` and keeps its number.
- **Safeguards are criteria.** A story with a consequential action (delete,
  deactivate, approve, logout, editing a business record) lists the safeguards
  from `feature-workflow` **Safeguards by risk** as "Accepts when" and "Rejects
  when" conditions. A policy the user has not decided (soft or hard delete,
  retention) goes back to step 1 as a grilling question.
- **Name the critical workflows.** Flows that cross several stories and break
  the business when they fail. Each becomes one system E2E spec in
  `feature-workflow` step 8. Keep the list short.
- **Include "Rejects when."** The happy path is half the spec. What happens on
  bad input, duplicate events, auth failure, timeout?
- **List what's out of scope.** This feeds the "Deliberately not built" line in
  `roadmap.md` phases.
- **No open questions before step 3.** A pasted spec's open questions go to
  step 1 as a grilling round. Don't slice foggy work — you'll re-slice it later.
- **UI stories point at the design.** Persist the approved design system with
  `ui-ux-pro-max` (`--design-system --persist --output-dir <project root>`).
  Each UI story gets "Accepts when: matches MASTER.md (or its page override) at
  each screen width in Design references". A printed output gets "Accepts when:
  the print matches <reference file>". Regenerating MASTER.md needs `--force`,
  which only the user authorizes, through Amend.

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
  has no test scenario and no "done when" condition. The one exception is an
  Amend phase that supersedes a technical decision. Its tests are the stories
  that decision touches.
- **Phase 0 is the smallest thing that proves the architecture works.** Not a
  skeleton, not scaffolding — a real user-visible outcome, however small.
- **One phase is `NEXT`. Everything else is `LATER`.** Never work on two phases
  at once.
- **Each phase states what it deliberately does NOT build** and which later
  phase owns it. This is what ponytail (in `feature-workflow`) enforces during
  the build — if it's not in this phase's stories, it gets cut.
- **DONE phases are history.** Never renumber or edit them. LATER phases may be
  renumbered, reordered, or inserted.
- **A phase too big for one build** splits into Phase Na and Nb before it
  starts. Found mid-build: stop and propose the split.
- **Bug fixes get no phase.** They run through the `feature-workflow` Gate.
- **Each critical workflow has an owner.** The phase that ships its last story
  writes its system spec, and that phase's "Done when" names the CW.

### Output — populate `roadmap.md`

`roadmap.md` is the single template for phase entries: NEXT, LATER, reopened,
and the DONE fields step 12 adds. Replace its placeholders with real phases.
Keep its `## Part` headings. The end of a Part is a full-E2E milestone in
`feature-workflow` step 8.

---

## 4. Route — hand the first phase to feature-workflow

Once `roadmap.md` is populated:

1. Confirm the user agrees with the phases and their order
2. Fill CLAUDE.md "Project facts" from the spec and the repo: trust boundaries
   (auth, roles, and every permission rule the spec names), UI test driver,
   Test command, Full E2E command. A tool that does not exist yet reads
   `none yet, added in Phase <N>`, and that phase installs it as its first task
3. Identify the phase marked `NEXT`
4. Hand it to `feature-workflow` starting at step 5

**The handoff:**

> The phase marked NEXT is **Phase 0 — Webhook listener**.
> It covers US-1 and US-2. Entering `feature-workflow` at step 5.

From here, `feature-workflow` takes over:
- Step 5 turns this phase into an implementation plan
- Steps 6–11 build, test, review, and merge it
- Step 12 updates `roadmap.md` (marks DONE, promotes NEXT)
- Then this cycle repeats for the next phase

---

## Amend — when an approved requirement changes

Use when `spec.md` is approved and a requirement is added, changed, removed, or
conflicts with another, before, during, or after the build, including when every
phase is DONE. Revisit only what the change touches. Completed work and phase history stay.

**1. Classify, then act.** One request can match several rows (removing one
story can change another). Apply each row that matches.

| Change | `spec.md` | `roadmap.md` | Build |
|---|---|---|---|
| Clarification, story text unchanged | no edit | no edit | continue |
| A story in the NEXT phase changes | edit the story, add a `Changed` line | edit the NEXT phase only if its outcomes or "Done when" change | revise the plan (step 5), cut the revision (step 6) |
| New requirement | new US number | add it to a LATER phase or a new phase | current phase unchanged unless the user says otherwise |
| Conflicts with an approved story | no edit until the user picks | no edit | stop, show both stories, put the choice to the user as a grilling round (step 1 format), then classify again |
| Changes a story of a DONE phase | edit the story, add a `Changed` line | new LATER phase with the line `Reopens Phase <N>: <US-n changed, or what changes>`. The DONE entry stays untouched | built as its own phase |
| Requirement removed or replaced | `Withdrawn` line, number kept, removed from `## Critical workflows` | remove it from LATER phases. Already shipped: a new phase `Reopens Phase <N>: US-<n> withdrawn`, done when the behavior is gone and the stories it touched still pass | the phase that removes the behavior deletes the tests of that behavior only. A test filed under the withdrawn story that covers behavior still shipped moves to the story that owns it. Its stored data, including records still in progress, stays unless the user decides otherwise |
| Undo a past decision | edit the story, `Changed` line naming the date of the behavior it restores. A technical decision with no story: no edit | a new phase. A technical decision or a recorded user decision gets the line `Supersedes Phase <N> convention: <old> → <new>, <reason>` (or `decision`) | a forward change, built as a phase. To `git revert` instead, see **Reverting a phase** after step 4 |
| UI or flow redesign | conditions unchanged: no edit. Conditions changed: as for a changed story | a new phase, `Reopens Phase <N>: <what changes>` | its E2E specs are updated without weakening any assertion |
| Schema, API, or business-rule change | as above for each affected story. No story text changes: no edit | a new phase, even when no story text changes. Exception: every affected story is in the NEXT phase and none is in a DONE phase, then revise NEXT as for a changed NEXT story and put any `Supersedes` line in it | the Test scope must include Regression selection (`feature-workflow` step 8) |
| Schema change with existing data | as above | as above | the plan includes the migration, how existing values map to the new columns (the user approves the mapping), a backup, a way to reverse it that touches only rows the migration changed, and every reader of the old columns (code, APIs, reports, exports, and installed clients still running older builds). Exception: adding an optional field that no existing value moves into needs only the migration, its default, and the readers. Backup, reverse, and test copy apply once existing values move or change. The dry run reports each case of the mapping with real examples, so the user approves the mapping on the data, not on a count. The migration and its reverse run first on a test copy the user provides (an export or backup restored into a throwaway deployment). The schema changes in two deploys: first add the new fields as optional and migrate, then remove the old field in a later phase, because the deployed schema must accept every existing row. The plan states the production order (deploy, dry run, run) for the user to carry out. If a push deploys automatically, the user decides when the change is merged. Never run it against production data yourself. Any loss of existing data needs the user's decision |

**2. Impact.** Read `spec.md`, `roadmap.md`, the code and callers in the
affected files, their tests, the latest `docs/test-results/`, and the schema.
Read the NEXT and every Paused branch too (`git diff <default>...<branch>`), since
unmerged code can bypass the change. List every reader and every writer of the
changed fields or rules, on the default branch and on those branches.
List the affected stories, the phases that shipped them (the "Shipped as" lines
in `roadmap.md`), the modules, tables, APIs, existing data, and their E2E specs
(feature and critical workflow). Show the user this list, the classification,
and a plan summary as a change note. The approved change note becomes the
header of the step 5 plan, which does not restate it. `Changed` and `Reopens`
lines stay one line each.

**3. Approval.** The change note needs the user's OK before any doc or code
change. A clarification that edits nothing is the only exception. Decisions the
change note raises (data mapping, retention, which story wins) are asked as a
grilling round in step 1 format, and approval waits until that frontier is empty.

**4. Return** to `feature-workflow` step 5 when the affected phase is NEXT. Otherwise
it waits in the roadmap. One phase at a time still holds: a reopen phase waits as LATER unless the user moves it to
NEXT. To build it ahead of an unfinished NEXT phase, the unfinished phase goes
back to LATER with the line `Paused <date>: <reason>`, keeps its branch, and
becomes NEXT again when the new phase is DONE. On resume, rebase its branch on
the default branch. Resolve conflicts in source files and regenerate codegen
(for example `npx convex codegen`), never hand-merge it. Then rerun
`feature-workflow` step 5 against what merged meanwhile. Its Test scope adds the
feature and system specs of those phases. If no phase is NEXT, the new phase becomes NEXT. When one Amend creates
several phases, the user sets their order and only the first becomes NEXT. New
phases go under a new `## Part` heading when they don't serve an unfinished
Part's goal, and always when every phase is DONE. Finishing that Part triggers a
full E2E run. New phases take the next unused phase number.

**Reverting a phase.** Only when the user asks, and after the change note is approved (Amend step 3):

`<default>` below is the repo's default branch (`main` or `master`).

1. `git diff --name-only <merge> <default>` (or the branch that holds the merge) lists only `spec.md`, `roadmap.md`, and
   results files (`docs/test-results/`, or older root `PHASE-*-TEST-RESULTS.md`).
   Anything else means later work sits on it: build a forward change instead.
   Then check the NEXT and every Paused branch (`git diff <default>...<branch>`).
   A hit is code on the branch that uses a name or table from the phase's
   "Shipped as", or a file the merge added that the branch changes. Generated
   files never count. A shared file such as the schema counts only when the
   branch changes the same table or function the merge changed. Also list LATER
   phases whose stories depend on the reverted ones. Any hit: the user
   re-slices them first.
2. If the phase changed the schema and its migration ran anywhere, run the
   reverse migration there first. The old schema rejects migrated rows, so the
   revert cannot deploy until then.
3. On a new branch from `<default>`, `git revert -m 1 <merge>`. For a squash or
   rebase merge, revert the phase's commits.
4. The revert deletes files the merge added. Restore the results file, and every
   E2E spec (feature or system) that also covers a story still shipped, then
   commit them:
   `git checkout <merge> -- docs/test-results/PHASE-<N>-TEST-RESULTS.md <specs>`.
   A restored system spec whose workflow includes a reverted story drops that
   hand-off, and `## Critical workflows` in `spec.md` changes the same way. If
   fewer than two stories remain, delete that system spec and its CW line.
5. Run it as its own phase through steps 8–12, a new roadmap entry with the next
   unused phase number and the line
   `Reverts Phase <N> (<revert hash>)` written after the revert commit. Its Test
   scope covers the remaining stories the reverted phase touched. Done when the
   reverted behavior is gone and those stories pass. The revert phase is NEXT.
   An unfinished NEXT phase is paused as in step 4, so step 12 marks the revert
   DONE.
6. The reverted stories get `Withdrawn` lines. The reverted DONE entry stays.
   Data the reverted code wrote is the user's decision.

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

Amend  ←── requirement change at any step ──→  back to step 5
```

---

## Failure modes

- **Skipping step 1.** Building a spec from assumptions produces a spec you
  rewrite after the first phase ships. A few grilling rounds cost less.
- **Answering your own decisions.** A grilling round the user did not answer
  produced your opinion, not their spec.
- **Interviewing in step 2.** Gaps found while writing go back to step 1 as a
  round, so the tree stays complete.
- **User stories without "rejects when."** The happy path works. Everything
  else is discovered in production.
- **Slicing by layer instead of by story.** Three phases of invisible
  infrastructure before anything is verifiable.
- **Phase 0 is too big.** If it takes more than a few days, slice smaller.
  Phase 0's job is to prove the architecture, not ship the product.
- **Open questions carried into slicing.** Foggy stories produce foggy phases
  that get re-sliced mid-build.
- **Restarting the cycle for a change.** Amend touches only the affected
  stories and phases.
