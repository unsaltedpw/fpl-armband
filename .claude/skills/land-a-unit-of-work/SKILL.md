---
name: land-a-unit-of-work
description: Land a finished unit of work — merge the FPL-Armband PR, merge its vault twin, and retire both worktrees. Use when a branch is ready to merge, when several PRs need landing in sequence, or when cleaning up after a merge. Covers the gh footguns that have already put retracted claims into permanent history.
---

# Landing a unit of work

⚠️ **This is NOT a review gate**, with one deliberate exception noted at §4. `merge-gate` and
`review-gate` were retired outright in `c83c5bb3` (2026-08-20) as an internal review ritual that did not belong in a published product
repo. **CI is the full-suite witness.** Do not re-introduce a twelve-condition checklist, a
review-record requirement, or a local full-suite run before merge. This skill is the *mechanical*
landing sequence and the footguns in it — nothing more.

⚠️ **§4 is the one retired condition deliberately carried** — `merge-gate` row 12,
*"the paired branch for this work, where one exists, merges in the same sitting"* — because
`~/.claude/CLAUDE.md` mandates it independently of any review ritual. **Do not delete §4 as a
rebuilt condition.**

**The unit of work is a worktree PAIR: an fpl branch and a vault branch of the same name.** Landing
one and not the other is the failure this exists to prevent — the evidence for what just shipped
stays on a branch nobody reads.

## 1. Read CI before you merge. Actually read it.

```bash
gh pr checks <n>
gh pr view <n> --json mergeable,mergeStateStatus -q '.mergeable+" "+.mergeStateStatus'
```

⚠️ **Read `go test`'s own result, and read ALL of it.** A wrapper can exit 0 while the suite under
it failed — and **filtering the output by exclusion is not a pass signal either.**

⚠️ **Dated 2026-08-31, recorded in an FPL-Armband commit body (the hash cited here before was not a real object).** A build agent piped
`go test ./...` through **`grep -v '^ok' | head -40`**. The exclusion filter stripped the passing
lines, and forty lines of remaining stderr pushed the `FAIL` lines past the cut — so it reported the
suite clean **twice, over several commits**, hiding the same two real failures: a fingerprint-path
collapse and a complexity ratchet breach. Both were gates working correctly; only the reader was
broken.

**So: capture the exit code — it is the gate — and use grep only to LOCATE the failures.** `head`,
`tail` and a `grep -v` for `ok` all fail open: they show you a suite that passed because the part
that failed is not on screen. ⚠️ **Grep is not a substitute for the exit code**, because a failure
that emits no marker at column 0 — a module-resolution error, a killed harness — leaves it silent.

```bash
set -o pipefail
go test ./... -count=1 2>&1 | tee /tmp/t.log; echo "exit=$?"
grep -nE '^(--- )?FAIL|^panic:' /tmp/t.log    # locate them; the exit code decides
```

⚠️ **`-count=1` is part of the recipe, not a nicety.** Two "green" logs from that same incident
report `(cached)` for 14 and 15 of 22 packages — a pass that executed almost nothing. That is the
same fail-open one layer down.

⚠️ **Do not end the grep with `|| echo "no failures"`.** On a mistyped log path it prints exactly
that over a "No such file" warning — the failure mode this whole block is about.

⚠️ **A red job with a SKIPPED test step failed at an EARLIER step — read which one.** PR #147 was
merged without reading CI at all. It broke the complexity ratchet on `main`, and the `test` job
reported `failure` with the Test step *skipped*, because the ratchet gate runs first and fails
first — so it read as a flaky test. **Four** subsequent PRs were red for that reason and not their
own. The job was never green; nobody looked.

## 2. Merge — and control the commit message yourself

```bash
gh pr merge <n> --squash --subject "<subject> (#<n>)" --body-file <file>
```

⚠️ **`--squash` takes the BRANCH's commit subject, not the PR title.** If you corrected the PR
title after opening it, the correction **does not reach `main`**. On 2026-08-30 this put
`"~10% of weekly SD"` — a figure the finding explicitly retracts — into `main`'s permanent history
as commit `2c4521d9`. History cannot be rewritten. It cost an extra PR (#159) to write the
correction into the diagnostic's own header instead.

**So: if the PR title or body was ever corrected, pass `--subject` and `--body-file` explicitly.**
The durable fix is the repository setting, and it is **two** settings — the body has the identical
defect. `--subject`/`--body-file` is the workaround until someone runs:

```bash
gh api -X PATCH repos/beeradb/FPL-Armband \
  -f squash_merge_commit_title=PR_TITLE -f squash_merge_commit_message=PR_BODY
```
Check what you are about to land, before you land it:

```bash
git log origin/main..HEAD --format='%s%n%b' | grep -nEi '<any figure you retracted>'
```

⚠️ **`gh pr edit` may fail with a `projectCards` GraphQL deprecation error.** Use the API instead:

```bash
gh api -X PATCH "repos/$(gh repo view --json nameWithOwner -q .nameWithOwner)/pulls/<n>" \
  -f title="..." -F body=@<file>
```

⚠️ **`--delete-branch` aborts if `main` is checked out in another worktree** —
`fatal: 'main' is already used by worktree at ...`. **The merge itself still succeeded.** Verify on
the remote rather than trusting the command's exit, and delete the branch separately.

## 3. Wait for `main` to go green before the next merge

Landing several PRs individually is the rule (each into `main` separately, in tested order). Each
one is green *against its own base*; that does not mean they compose. Wait:

```bash
for i in $(seq 1 20); do
  S=$(gh run list --branch main --limit 4 --json name,status,conclusion,headSha \
      -q '.[]|select(.headSha|startswith("<sha>"))|.name+":"+.status+":"+(.conclusion|tostring)' | tr '\n' ' ')
  echo "$S"; case "$S" in *in_progress*|*queued*) sleep 40;; *) break;; esac
done
```

⚠️ **`.conclusion` is `""`, not `null`, while a run is in progress**, so `(.conclusion // .status)`
does **not** fall through and a wait loop built on it exits immediately reporting nothing. Select on
`.status` explicitly, as above.

## 4. Merge the vault twin — "merge one, merge the other"

```bash
git -C <vault> merge --no-ff <branch> -m "<subject>

<why, and what the fpl side landed as>"
```

⚠️ **The vault merges into its LOCAL `main`**, not `origin/main`. Pushing the vault is manual and
lags badly.

Say in the merge message which fpl commit it lands with, and carry any correction the note picked up
on review. If the fpl squash subject ended up carrying a retracted claim (see §2), **say so here** —
this is the record that explains why the repo history disagrees with the note.

## 5. Retire both sides, as part of the same act

```bash
~/.claude/bin/research-worktree remove <branch>     # vault: worktree, then branch
git -C <fpl-repo> worktree remove <path>            # fpl worktree
git -C <fpl-repo> branch -d <branch>                # -d REFUSES if anything is unmerged
git -C <fpl-repo> push origin --delete <branch>     # --delete-branch may have failed; see §2
```

⚠️ **`-d`, never `-D` — with exactly one exception, and you must earn it.** `-d` declining is
normally the signal that the merge did not take everything, and forcing past it discards the
difference silently.

⚠️ **But a SQUASH merge always trips `-d`**, because the content landed as one new commit and the
branch's own commits never became ancestors of `main`. Every squash-merged branch therefore looks
unmerged, and "`-d`, never `-D`" taken literally would leave all of them behind forever.

**So verify the content, then force — never force on the assumption that it was a squash:**

```bash
git fetch origin
sq=$(gh pr view <n> --json mergeCommit -q .mergeCommit.oid)          # exact; works for merge commits too
base=$(git merge-base <branch> "$sq^")
git diff --stat <branch> "$sq" -- $(git diff --name-only "$base" <branch>)   # MUST be empty
```

Empty output means every file the branch touched is identical in what landed, and nothing is lost.
Only then use `-D`. **If it prints anything at all, stop** — that is the case the rule is
protecting, and the difference is real.

⚠️ **Both halves of that command are load-bearing, and this rule has been wrong twice.**

**Compare against the merge commit, not `origin/main`.** `origin/main` moves, so a two-dot diff
against it reports every *later* PR as your branch's unlanded work — measured at 6 files one PR
later and 68 files twenty-one PRs later, on a branch that had landed perfectly.

**And scope the diff to the files the branch touched.** A bare `git diff <branch> <squash-sha>` is
*still* wrong, because the squash commit's tree is main-at-merge-time plus your branch, so a branch
cut from an older main is behind on everything else. Measured across sixteen merged branches: the
unscoped form false-stops on **7 of 16**, all of which had landed fine — one printed
`46 files changed`. Scoping removes the staleness without hiding loss: a deleted file still appears
in `--name-only`, and another PR touching the same file between fork and merge still shows.

⚠️ **Get `$sq` from `gh`, not from `git log --grep`.** Grepping for `(#<n>)` returns nothing at all
on a PR that landed as a *merge* commit rather than a squash — leaving `$sq` empty, so the diff
silently compares against the worktree — and `-1` picks the wrong commit when a PR number appears
in another subject (measured: 2 of 95 live numbers here).

⚠️ **Never remove the worktree the current session is running in** — the harness cleans that at exit.

Leaving a merged pair in place is not neutral: the next session branches from a stale tip and
re-does landed work, a stale pair silently captures notes that never reach `main`, and
`git worktree list` is the only inventory there is.

## Done means

- [ ] fpl PR merged, and `main` verified green **after** it
- [ ] nothing retracted in the squash subject or body that landed
- [ ] vault twin merged to the vault's local `main`
- [ ] both worktrees removed (bar the one this session runs in); vault branch deleted with `-d`,
      fpl branch with `-d` or — squash-merged, scoped diff against the merge commit empty — `-D`;
      remote branch gone
- [ ] `git worktree list` no longer names either side
