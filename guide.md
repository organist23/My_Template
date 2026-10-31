# Guide: using the workflow template in a real project

This guide covers setup, then the exact prompts for each stage. It is for you,
not for Claude, so do not copy it into projects.

## Template layout

```
template/
├── CLAUDE.md                               ← rules + "Project facts" to fill in
├── roadmap.md                              ← phase template, step 3 overwrites it
├── guide.md                                ← this file, stays in the template
├── .claude/
│   └── skills/
│       ├── project-workflow/SKILL.md       ← /project-workflow (steps 1–4)
│       └── feature-workflow/SKILL.md       ← /feature-workflow (steps 5–12)
└── examples/
    └── e2e-testing-clio.md                 ← model for your project's e2e skill
```

Claude Code loads skills only from `.claude/skills/<name>/SKILL.md`. The e2e
example sits outside that folder on purpose. It is specific to the Firm B Clio
project and must not load in other projects.

---

## Part 1 — One-time setup

### Step 1. Install the plugins (once per machine)

The workflow calls plugins it does not include. Without them, steps fail silently.

| Plugin | Source | Used in |
|---|---|---|
| `superpowers` | Anthropic official marketplace | steps 5, 7, 9, 11 |
| `ponytail` | community, `dietrichgebert/ponytail` | steps 6, 10 |
| `mattpocock-skills` | Anthropic official marketplace | optional: `wayfinder`, `to-spec`, `to-tickets` |

In Claude Code, run these from the template folder. When asked for a scope, pick
**project**. That writes `.claude/settings.json`, so the plugin list is saved
with the template and every copied project declares the same plugins.

Both marketplaces are already declared in the template's `.claude/settings.json`.
Accept the folder trust prompt, then install:

```
/plugin install superpowers@claude-plugins-official
/plugin install mattpocock-skills@claude-plugins-official
/plugin install ponytail@ponytail
```

Restart Claude Code. Type `/` and confirm `/ponytail-review` appears.

Other tools, installed by you:

| Tool | Used in | How |
|---|---|---|
| Claude in Chrome extension | step 8, web apps | Chrome Web Store |
| Playwright | step 8, desktop / Electron apps only | per project, `npm i -D playwright` |
| git | steps 11–12 | merge hash goes into roadmap.md |

### Step 2. Copy the template into a project

PowerShell, run from the new project's root:

```powershell
$t = "C:\Users\Admin\Desktop\sample_md_files"
Copy-Item "$t\CLAUDE.md", "$t\roadmap.md" .
Copy-Item "$t\.claude" . -Recurse -Force
```

Once the template is in a git repository, `git clone <repo> my-project` works too.
Delete `guide.md` and `examples/` from the clone afterwards.

Open Claude Code in the project and accept the trust prompt. Type `/` and confirm
`project-workflow` and `feature-workflow` appear.

### Step 3. Fill in "Project facts" in CLAUDE.md

Section 3 of CLAUDE.md has placeholders that `feature-workflow` reads: trust
boundaries for step 10, the UI test driver for step 8, and the test command.
Fill them in, or let Claude do it:

```
Read CLAUDE.md and .claude/skills/feature-workflow/SKILL.md. Inspect this codebase,
then fill in the "## 3. Project facts" placeholders in CLAUDE.md. Change nothing else.
```

### Step 4. Write a project-specific e2e-testing skill

Step 8 reads an `e2e-testing` skill. The template ships only the Clio example.
Use it as the model, not the content.

```
Read C:\Users\Admin\Desktop\sample_md_files\examples\e2e-testing-clio.md as a model only. Write
.claude/skills/e2e-testing/SKILL.md for THIS project with the same sections:
isolation from live, setup, prove isolation before creating data, during the pass,
teardown, common mistakes. Keep its safety patterns: a pinned test target that
refuses to run when unset or pointing at production, reads-only against
production, teardown that deletes everything the pass created. Ask me about our
environments, shared accounts, and test data before writing.
```

If the project has no live system yet, skip this step and tell Claude in step 8:
"no live environment yet, use the project's own runner".

---

## Part 2 — Daily use

### Stage A. New project or vague idea → project-workflow (steps 1–4)

```
/project-workflow I want to <goal in your own words>. Save the spec to spec.md.
```

The skill does not name a file for the spec. Ask for `spec.md` so later sessions
can read it.

What happens, and what you do:

| Step | Claude does | You do |
|---|---|---|
| 1. Understand | asks questions in groups | answer, then confirm the one-sentence goal |
| 2. Spec | writes numbered user stories (US-1, US-2…) with "Accepts when" and "Rejects when" | check every story is testable, answer open questions |
| 3. Slice | fills roadmap.md, one phase NEXT, rest LATER | check the order and that Phase 0 is small |
| 4. Route | hands Phase 0 to feature-workflow | confirm |

Already have a spec? Paste or point to it. Claude checks it against step 2's rules
before slicing:

```
/project-workflow Here is my spec: spec.md. Check it, then slice it into roadmap.md.
```

### Stage B. Build one phase → feature-workflow (steps 5–12)

Start each phase in a fresh session:

```
/feature-workflow Build the phase marked NEXT in roadmap.md. The spec is spec.md.
```

| Step | What happens | Your prompt if it stalls |
|---|---|---|
| 5. Plan | writes the implementation plan | `Write the plan for the NEXT phase.` |
| 6. Cut gate | re-arms ponytail, cuts speculative plan steps | `Invoke ponytail:ponytail and cut the plan.` |
| 7. Code + TDD | tests first, then code | `Execute the plan with TDD.` |
| 8. E2E | one scenario per story, one assertion per Accepts/Rejects condition | `Run step 8 using the e2e-testing skill.` |
| 9. Verify | runs every command, reads output | `Verify before claiming done.` |
| 10. Review | complexity, then correctness, then security if gated | `/ponytail-review`, then `/code-review`, then `/security-review` |
| 11. Merge | merge, rebase, or abandon | `Finish the branch.` |
| 12. Roadmap | marks phase DONE, promotes next phase to NEXT | `Apply the roadmap rule in CLAUDE.md.` |

Then start a new session and repeat Stage B for the next NEXT phase.

### Stage C. Bug or small fix

```
/feature-workflow Bug: <what happens, what should happen, how to reproduce>.
```

Claude re-arms ponytail (step 6), debugs first, then fixes with TDD (step 7) and
continues through review.

### Stage D. Long session or after /compact

```
Invoke ponytail:ponytail to re-arm the ruleset.
```

Do this after every `/compact` or `/clear`, and whenever the session has run for
hours. The `[PONYTAIL]` badge does not prove the rules are still in context.

---

## Part 3 — Rules to remember

- **One phase at a time.** Never start a LATER phase before NEXT is DONE.
- **Do not log in for Claude.** When a browser test hits a login, Claude stops. Log
  in yourself in that Chrome window, then say "done".
- **Reviews report, you decide.** Do not use `/simplify` or `/code-review --fix` at
  step 10.
- **E2E results** go in `PHASE-<N>-TEST-RESULTS.md`, and roadmap.md links them.
- **Template updates:** fix gaps here in `sample_md_files` first, then re-copy into
  projects. Edits made inside one project do not flow back.

---

## Quick reference

| Situation | Prompt |
|---|---|
| New project | `/project-workflow I want to <goal>. Save the spec to spec.md.` |
| Have a spec | `/project-workflow Here is my spec: spec.md. Check it, then slice it into roadmap.md.` |
| Build next phase | `/feature-workflow Build the phase marked NEXT in roadmap.md. The spec is spec.md.` |
| Bug | `/feature-workflow Bug: <symptom, expected, repro>.` |
| After /compact | `Invoke ponytail:ponytail to re-arm the ruleset.` |
| Where am I? | `Read roadmap.md and tell me which phase is NEXT and which step we are on.` |
