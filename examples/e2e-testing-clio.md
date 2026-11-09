---
name: e2e-testing
description: Use when a change has to be proven against Firm B's real Clio account — a live end-to-end pass, a manual check list, "test this for real", "verify live", or any work that creates test matters, tasks or webhook deliveries in Clio. Covers isolating the pass from the live deployment and from the separate automation server.
---

# End-to-end testing against real Clio

> Used during `feature-workflow` step 8 (Real-world E2E).
>
> **Example only.** Written for the Firm B Clio/Convex project. For a new project,
> write `.claude/skills/e2e-testing/SKILL.md` using this file as the model (see guide.md).

**Never run a live pass on a shared deployment.** Every pass gets three things of
its own: a throwaway Convex project, its own Clio webhook subscriptions, and its
own matter prefix. Firm B's Clio account is shared with the live deployment, a
separate automation server, and possibly another session's test pass running at
the same time.

## Routing — it is all in the matter description

Set every test matter's description to:

    <YOUR_PREFIX> ZZTASKFLOWTEST <what this matter is for>

Example: `ZZPHASE10TEST ZZTASKFLOWTEST stage change chain`

| Consumer | Its gate | Sees the example? |
|---|---|---|
| Live deployment `impressive-platypus-984` | accepts a description **or `display_number`** **starting with** `ZZTASKFLOWTEST` | **no** — it starts with your prefix. Check `display_number` too, see Creating test matters |
| The separate automation server | **skips** a description **containing** `ZZTASKFLOWTEST` anywhere | **no** — it contains it |
| Another session's test deployment | their own prefix, `startsWith` | **no** |
| **Your** deployment | `TEST_MATTER_PREFIX=<YOUR_PREFIX>` | **yes** |

Three rules follow, and each one has bitten a past pass:

- **Your prefix goes first.** That single fact is what excludes live and includes
  you. Put `ZZTASKFLOWTEST` first and live fires on your matter too.
- **`ZZTASKFLOWTEST` must appear somewhere.** Drop it and the separate automation
  server writes 20+ of its own tasks onto your matter.
- **Your prefix must be unique to this pass** (`ZZ<PHASE><WHAT>TEST`), never
  `ZZTASKFLOWTEST` itself. Another test deployment may be armed right now.

`isTestMatter` (`convex/runs.js`) matches `startsWith` against `[display_number,
description]`. Never loosen it to `includes`: Clio bakes the client's name into
`display_number`, so every matter of the "ZZTASKFLOWTEST Client" contact contains
the prefix mid-string. Unset prefix = no gate. Armed = fail-closed.

## Setup

```bash
# 1. A new Convex PROJECT — an account gets one dev deployment per project
npx convex dev --once --configure new --team mar-ang --project taskflow-<pass> --dev-deployment cloud

# Pin the new deployment. Empty or live = unset, so every command below refuses to run.
TEST_DEPLOYMENT=$(grep '^CONVEX_DEPLOYMENT=' .env.local | cut -d= -f2 | cut -d' ' -f1)
case "$TEST_DEPLOYMENT" in *impressive-platypus-984*) unset TEST_DEPLOYMENT ;; esac
echo "test deployment: ${TEST_DEPLOYMENT:?STOP - step 1 failed or still points at live}"

# 2. Copy env from live, then override the prefix.
npx convex env list --deployment dev:impressive-platypus-984
npx convex env set --deployment "${TEST_DEPLOYMENT:?}" -- NAME "$value"   # the -- matters: JWT_PRIVATE_KEY starts with -----
npx convex env set --deployment "${TEST_DEPLOYMENT:?}" TEST_MATTER_PREFIX ZZ<PASS>TEST

# 3. Data, minus three tables (clioWebhooks, runs, clioMatterStages)
npx convex export --deployment dev:impressive-platypus-984 --path snapshot.zip
zip -d snapshot.zip 'clioWebhooks/*' 'runs/*' 'clioMatterStages/*'
# Import runs only if the listing succeeds AND none of the three tables remain.
if unzip -l snapshot.zip > snapshot.txt && ! grep -qE ' (clioWebhooks|runs|clioMatterStages)/' snapshot.txt; then
  npx convex import --deployment "${TEST_DEPLOYMENT:?}" --replace-all snapshot.zip
else
  echo "STOP: could not list snapshot, or excluded tables still in it. Nothing imported."
fi

# 4. Your own Clio subscriptions — no OAuth needed
npx convex run --deployment "${TEST_DEPLOYMENT:?}" webhooks:register '{"firmId":"k17513zs2kh44p6evn1rfqckd58c7brz"}'
```

**Every command that writes names `--deployment "${TEST_DEPLOYMENT:?}"`.** If
`TEST_DEPLOYMENT` is unset, the shell refuses to run the command at all. Only
`env list` and `export` name live, and both only read.

**Excluding `clioWebhooks` from the import is not optional.** `beginRegister`
deletes `begun.oldHookId` after it registers, so importing live's hook rows makes
your registration destroy the live deployment's Clio subscriptions. Exclude `runs`
and `clioMatterStages` too, for a clean slate.

Step 4 needs no OAuth flow: the copied `clioConnections` row decrypts with the
copied `CLIO_TOKEN_KEY`, and the Clio app's redirect URI is only consulted during
OAuth. The new subscriptions point at your `CONVEX_SITE_URL`.

## Prove the isolation before creating anything

1. **Every `clioWebhooks` row has `sealed` set.** Accepted ≠ real — `sealed` means
   Clio completed the handshake and will actually deliver.
2. **The gate fires:**
   `npx convex run --deployment "${TEST_DEPLOYMENT:?}" runs:start '{"firmId":"k17513zs2kh44p6evn1rfqckd58c7brz","matterId":1,"matterNames":["Definitely Not A Test Matter"]}'`
   → logs `runs: skipped, not a test matter`.
3. **You are testing the code you think you are.** Call a function to confirm it
   exists; `npx convex function-spec` has reported a stale function set here.

## Creating test matters

The app has no matter-write path and must not grow one. Write a throwaway
`convex/spike.js` exporting an `internalAction` that reads the token via
`internal.clio.forFirm` and calls `clioFetch`, `npx convex dev --once`, then
`npx convex run --deployment "${TEST_DEPLOYMENT:?}" spike:createTestMatter '{...}'`.
**Keep the file until teardown is done.** Teardown uses it to delete your
subscriptions and test matters.

A matter needs a client: reuse contact `2417596538` ("ZZTASKFLOWTEST Client").
A task needs an assignee — Clio rejects one without.

**Check the first matter's `display_number` before any stage change.** Live's
gate also matches `display_number`, and Clio builds it from the client name. If it
starts with `ZZTASKFLOWTEST`, live will act on every test matter whatever your
prefix. Stop and report it. Do not change the client or the number format yourself.

## During the pass

- **Hand-derive every expected date and write it down before the first fire.** A
  date computed by the code under test proves nothing about the code under test.
- **Task presence in Clio is not a signal.** The `runs` table and the log lines are.
- **Spike every new Clio write payload live.** A green suite proves nothing about
  what Clio accepts.
- **If tasks you did not write appear on your matter, stop and report it.** The
  Phase 9 pass got 21–35 strays per matter with a correctly formatted description
  and the cause was never found. Working around it hides the signal.

## Teardown — this is a real client account

- `npx convex run --deployment "${TEST_DEPLOYMENT:?}" backfill:audit '{"firmId":"k17513zs2kh44p6evn1rfqckd58c7brz"}'`
  then `backfill:purge`. It refuses unless `TEST_MATTER_PREFIX` is set, and
  re-reads each task's name and matter before deleting it.
- **Delete your Clio subscriptions** (add a `DELETE /webhooks/{id}` action to
  `convex/spike.js`), or they deliver forever from a deployment nobody watches.
- **Delete your test matters** (add a `DELETE /matters/{id}` action to the same
  spike). Re-read each matter first and delete it only if its description starts
  with your prefix. `backfill:purge` removes tasks, not matters.
- Delete `convex/spike.js`, re-push, then delete the Convex project.
- Record the pass in `docs/test-results/PHASE-<N>-TEST-RESULTS.md`, including anything left behind.

## Common mistakes

| Symptom | Cause |
|---|---|
| Live deployment writes tasks on your test matter | description starts with `ZZTASKFLOWTEST` instead of your prefix |
| 20–35 unexplained tasks per matter | `ZZTASKFLOWTEST` missing from the description. If it is present, the cause is unknown (Phase 9): stop and report, do not work around it |
| Setup prints `STOP` or `parameter null or not set` | step 1 failed or `.env.local` points at live. Fix that before anything else |
| Live's Clio subscription dies mid-pass | `clioWebhooks` was included in the import |
| Gate looks armed but nothing is gated | the gate is on a branch this deployment never deployed |
| A matter you meant to exclude fires anyway | prefix collides with another live pass |

One shared risk with no workaround: both deployments use the same Clio refresh
token and Clio rotates it on use. If one refreshes on a 401, the other may need a
manual Reconnect.
