---
name: feature-workflow
description: >-
  The standard build loop for a single feature, bugfix, or change that needs a
  plan: plan, ponytail cut gate, TDD, real-world E2E, verify, review, merge.
  Also carries the standing constraints that superpowers and ponytail leave
  unresolved between them — ponytail's UI scope limit, the ponytail re-arm rule,
  the conflict rulings, and test integrity. Use when the user types
  "/feature-workflow", or when starting implementation work where what to build
  is already clear. Use "project-workflow" instead when the work is foggy enough
  that you cannot yet name the questions, and it will route back here per slice.
---

# Feature Workflow

Steps 5–12. Numbering continues `project-workflow`'s 1–4, which produces the
spec and slices this loop consumes; entered directly, step 5 is the start.

Superpowers walks the route. Ponytail keeps it from bloating. The constraints
below are where those two disagree, resolved once so no task re-litigates them.

**Read the constraints before step 5. They gate every step after it.**

---

## Constraints

### Ponytail scope: backend open, UI closed

`ponytail` applies fully to backend, server, build, and tooling code — cut
abstractions, dependencies, and files freely there.

It does not get to cut user-facing surface. In UI code, treat these as
explicitly requested, never speculative (ladder rung 1 does not apply):

- loading, empty, and error states
- keyboard navigation, focus order, Escape/Enter handling
- undo and optimistic updates where the surrounding UI already has them
- transitions and states the design system already defines

Lazy in the UI means *fewer elements and no new components* — reuse a primitive,
reuse a token. Never fewer states or a plainer control.

Trade UI away for performance only with a measurement showing the current
version is too slow. "It'd be simpler" is not that measurement.

Ponytail 5.3.3 already protects the *design system* ("a house component beats
a native widget") and never cuts accessibility. What it does not cover is rung 1,
which cannot tell a speculative abstraction from a loading state. That is the
gap this closes.

### Ponytail re-arm

The `SessionStart` hook injects ponytail's ladder **once per session**. Nothing
re-injects it: `ponytail-mode-tracker.js` re-emits the ruleset only under
`if (isQoder)`, `/ponytail` emits a one-line banner, and `commands/ponytail.toml`
is a one-paragraph paraphrase carrying the ladder without its guardrails —
no "never cut" list (validation, security, accessibility), no "a house component
beats a native widget", no "never lazy about the change itself".

- **Re-arm by invoking the skill** (`ponytail:ponytail`). That is the only thing
  that reloads the full ruleset mid-session. `/clear` and `/compact` re-fire
  `SessionStart`, but re-arm after them anyway: re-arming costs one call, and a
  missed reload costs the rest of the session.
- **Re-arm at step 6, and after every compaction.**
- **Never infer active constraint from `[PONYTAIL]` or `.ponytail-active`.** Both
  read a flag file, not the context. They say `full` in hour three of a session
  where the ladder stopped being read in hour one. The mode did not turn off;
  the rules aged out.

### Resolved conflicts

Fixed rulings. Do not re-decide these per task:

| Conflict | Ruling |
|---|---|
| ponytail's "one runnable check, no frameworks" vs `superpowers:test-driven-development` | **TDD wins.** |
| ponytail rung 1 vs UI surface | **UI wins.** See ponytail scope above. |
| An implementer's "left it per YAGNI" vs a reviewer's finding | **The reviewer wins.** `subagent-driven-development/task-reviewer-prompt.md` is explicit: a stated rationale never downgrades a finding's severity. Ponytail licenses building less, not defending after the fact. |
| A `mattpocock-skills` skill vs the skill a step names | **The named skill wins.** That plugin (1.3.1) is installed for `wayfinder`, `to-spec`, and `to-tickets` only — all three explicit-invoke. But 11 of its skills auto-invoke, and 9 overlap steps here: `codebase-design`, `domain-modeling`, `research`, `prototype`, `grilling` (step 5), `tdd`, `diagnosing-bugs` (step 7), `code-review` (step 10), `pr` (step 11). Use what the step names, not what fires. |

### Test integrity

A test that has never failed proves nothing. Every level, not just unit tests —
E2E is where green-by-construction hides best.

- **Watch it fail first.** Before calling any test done, break the code it covers
  on purpose — return empty, flip the branch, drop the side effect — and confirm
  it goes red. Still green = decoration; delete or rewrite it.
- **Capture fixtures, don't invent them.** Record one real payload (real API
  response, real session, real upload) and commit it. Invented fixtures encode
  assumptions about the data; captured ones encode the data.
- **Mock only at the outer boundary** — third-party call, clock, network.
  Anything stubbed between the entry point and the assertion means the stub is
  under test.
- **Hand-derive expected values.** An expectation computed by the code under test
  passes no matter what that code does.

`superpowers:test-driven-development` has this as the Mutation Check
(`writing-good-tests.md`), but scopes it to unit test files. It applies to E2E too.

### Browser automation

`claude-in-chrome` against a live system is a **probe, not a suite** — one-shot,
non-deterministic, real auth and real third-party state. That is its value, and
why anything it catches must become a committed, re-runnable spec. A finding
that lives only in a transcript protects nothing tomorrow.

**Stop at the login wall — never authenticate on the user's behalf.** When a
flow needs credentials, halt and ask the user to log in themselves in that
browser, then continue when they say it is done.

- Do not type credentials from anywhere: the conversation, a `.env`, an
  environment variable, a config file, a password manager, or a previous
  transcript. Finding a working credential is not permission to use it.
- The same applies to MFA codes, SSO redirects, and CAPTCHAs. Hand those back.
- `claude-in-chrome` drives the user's real Chrome, so a manual login costs them
  a few seconds and leaves the session authenticated for every step after.
- After login, don't screenshot or read back pages showing tokens, session
  cookies, or account secrets. A transcript outlives the session.

Testing the login flow *itself* is the one case that needs credentials in the
browser. Use a throwaway test account whose credentials the user supplies for
that purpose — never a real one — and still ask before the first attempt.

Where a project's own instructions name a different driver for its UI, that wins.

---

## Gate

| Situation | Start at |
|---|---|
| Bug or small fix | step 6 re-arm (no plan to cut, re-arm only), then step 7 (`systematic-debugging` first) |
| Spec or plan already decided | step 5 |
| Can't yet name the questions | stop — use `project-workflow` |

---

## 5. `superpowers:writing-plans` — one plan per slice

Turns "what" into "in what order, touching which files".

Plan the phase marked NEXT in `roadmap.md`, from the stories it covers in
`spec.md`. A bug or a request with no spec plans from the request itself.

Skip `superpowers:brainstorming` if a map or spec already decided it. Keep it for
implementation-level choices nothing has reached yet.

## 6. Re-arm ponytail on the plan — the cut gate

**Before any code.** Invoke the skill (`ponytail:ponytail`). Not `/ponytail` — see
**Ponytail re-arm** above for why the command is not equivalent.

Ponytail is already on from the session start; this is not where it activates,
it is where you deliberately re-read it. The plan is where speculative
*implementation* first appears. Cutting a plan step costs nothing; cutting
shipped code costs a rewrite. Brainstorming stays exploratory on purpose —
ponytail is explicit that laziness never shortens comprehension.

Read the plan against the ladder. Kill speculative abstractions, new files, new
deps, config for values that never change. Bounded by **Ponytail scope** above.

## 7. `superpowers:executing-plans` + `superpowers:test-driven-development`

Ponytail stays on as a mode — its minimalism and TDD's "simplest code to pass"
agree, and where they don't, see **Resolved conflicts**.

Bug instead of a plan → `superpowers:systematic-debugging` first.

## 8. Real-world E2E — prove it against the actual system

Read `e2e-testing` before running a live pass — it covers isolation, setup,
and teardown for shared environments. If the project has no `e2e-testing`
skill, use the project's own runner against a real process (driver table below).

Unit, lint, and API tests prove the pieces. This proves the *use case*.

**Where the cases come from:** the numbered user stories in `spec.md`. One
story = one scenario, and each of its "Accepts when" and "Rejects when"
conditions is one assertion in that scenario. The test matrix is already written; don't re-invent it at
test time. Entered without a spec, enumerate the user-visible outcomes first.

**The shape — nothing stubbed between entry and assertion:**

```
captured fixture → the real endpoint/handler → real processing → hand-derived expected output
```

Fixtures, mocking boundary, and the mutation check are in **Test integrity** above.

**Edge cases — enumerate, don't imagine:** empty / zero / one / many · malformed
and wrong-type input · unauthorized and expired auth · slow, failed, and partial
network · duplicate and concurrent submission · unicode, very long strings,
injection-ish payloads · offline or interrupted mid-flow.

**Pick the driver by target:**

| Target | Driver |
|---|---|
| Web app in a real browser session | `claude-in-chrome` |
| Desktop / Electron app | Playwright over CDP |
| HTTP service, CLI, worker | the project's own runner against a real process |

*Check the project's instruction file first* — some projects forbid one of these
outright, and that overrides this table.

**Record the pass** in `PHASE-<N>-TEST-RESULTS.md`: date, driver, pass/fail per
story and per assertion, and anything left behind. Step 12 links it from
`roadmap.md`.

`superpowers` 7.0.0 has zero E2E or browser coverage; all 15 skills assume a fast
unit runner. This step is not duplicating an existing skill.

## 9. `superpowers:verification-before-completion`

Evidence before assertions. Run the test command from CLAUDE.md "Project facts"
and every other check, read the output, then claim done.

## 10. Review: complexity, then correctness, then security

`/ponytail-review` → `/code-review` → `/security-review`. That order — deleting
code first means less to review, and security audits the shape that actually
ships rather than one two passes are about to change.

The three scopes overlap only on simplification, so skipping one leaves a real
gap. When `/code-review` repeats a simplification finding `/ponytail-review`
already raised, act on it once:

| Pass | Covers | Explicitly excludes |
|---|---|---|
| `/ponytail-review` | over-engineering, dead flexibility | correctness, security, performance |
| `/code-review` | correctness bugs, reuse/simplification/efficiency | security |
| `/security-review` | injection, authn/authz, secrets, crypto, input validation, TOCTOU, deserialization, XSS, supply chain | DoS, rate limiting, resource exhaustion |

**Gate the security pass, don't always-run it.** Three review passes on a CSS
tweak is what ponytail exists to prevent. Run it when the diff touches any of:

- auth, sessions, tokens, or permission checks
- input parsing, deserialization, or file/path handling
- shell or SQL construction, or anything building a command string
- crypto, secrets, or credential storage
- a network-facing handler, or a new dependency

Extend that list with whatever the current project treats as a trust boundary —
its `CLAUDE.md` / `AGENTS.md` usually names them. In an app that talks to remote
hosts, that includes anything crossing the wire, running over SSH, or spawning a
user-supplied binary.

If in doubt, run it — ponytail is explicit that security measures are never
simplified away.

**All three report; none applies.** The built-in `/simplify` covers the same
complexity hunt as `/ponytail-review` but applies its fixes in the same pass,
landing changes in the tree before you have read them. Report-then-decide is the
point. Same reason `/code-review` runs without `--fix`.

Pair with `superpowers:receiving-code-review` when acting on findings — it holds
the YAGNI-defense procedure (grep for real usage before implementing a
reviewer's suggestion).

## 11. `superpowers:finishing-a-development-branch`

Merge, rebase, or abandon — decided once.

## 12. Roadmap

If the merged work completes the phase marked NEXT, apply the roadmap rule in CLAUDE.md. Otherwise skip.

---

## Failure modes

- **Ponytail on UI with no scope rule.** Rung 1 reads a loading state as
  speculative and deletes it.
- **Trusting the badge.** `[PONYTAIL]` reads the flag file, not the context.
  Re-arm; never infer.
- **An E2E test nobody watched fail.** Green-by-construction hides best at this level.
- **Reaching for `/simplify` at step 10.** It applies what it finds. Read first.
