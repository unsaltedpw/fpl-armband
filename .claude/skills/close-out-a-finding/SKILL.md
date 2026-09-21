---
name: close-out-a-finding
description: Turn a completed measurement into a vault finding that will survive review — verdict, detection thresholds, multiplicity, and the specific self-checks that have caught real errors here. Use after any measurement reaches a conclusion, and before requesting fpl-stats-review or fpl-docs-review.
---

# Closing out a finding

**Record the VERDICT, not the title.** A title alone does not stop an idea being rebuilt — that is
the failure this store exists to prevent. Use `templates/finding.md`.

Every check below is here because it caught a real error in a real note. Work the list; it is
faster than the review round-trip.

## Frontmatter

- `verdict:` — `measured-positive` / `measured-neutral` / `retracted` / … Not optional.
- `measured_at:` — the grid, the cluster count and `df`, the gates, the **cells path**, and the
  **analysis script**. ⚠️ If per-column `n` differ, list them. One note said "83,994 pairs" while
  its table showed five different `n` (80,999 / 76,620 / …) because the script dropped exact zeros
  per predictor. Nothing explained the gap.
- `commit:` — ⚠️ **update this when the branch merges.** Three notes sat on the vault's `main`
  saying "⚠️ UNMERGED at the time of writing" after their code had landed, telling readers the code
  was unavailable when it was on `origin/main`.

## Every number needs its threshold

State `t_crit(df) × SE` beside the estimate, always. "Not detected" is a statement about the design
as much as the world — say so when power is the binding constraint.

⚠️ **Do not borrow a threshold from the wrong column.** A bullet arguing the *rate* trends were
power-limited quoted `±0.56`, which was the *minutes* trend's threshold in the dearest band — the
very column the next sentence called well-powered.

⚠️ **A measured-positive with no interval on any number is not finished.** Cluster your unit of
analysis: 162,619 pairs drawn from 16,242 managers is ~10 per manager, not 162,619 independent
draws.

## The four self-checks that caught the most

**1. Read the multiplicity family size off the OUTPUT, not off memory.** A note said "eleven
comparisons"; the script printed thirteen. The arithmetic had always used thirteen — only the prose
miscounted, and it happened to err conservative. It will not always.

**2. Is any figure a RANGE STATISTIC being quoted as though it were tested?** A frontier was
headlined at "9.6% of weekly SD" — a spread over a mix of resolved and unresolved deltas, with no SE
of its own, carried entirely by its two extremes. Drop the highest and lowest arm and it halves.
**Quote the arms that individually clear their own threshold.**

**3. Does every trend figure state its METHOD and WINDOW?** "Mean score decays 41.3 → 30.4 across a
season" was quoted with no stated window. Chasing the missing method found there was **no decay at
all** — over halves the series goes slightly *up*. The endpoint pair had manufactured the trend. A
figure with no method is not reproducible and hides its own error.

**4. Is "indistinguishable" doing work that "equal" cannot?** Nine arms were reported EV-neutral
because none crossed its own threshold — while **8 of 9 drifted negative**: mean **−21.4**
pts/season across all nine, **−27.4** across the eight, under thresholds two to four times the
project's own detection floor. ⚠️ Say which denominator you mean — printed next to "8 of 9", a bare
−21.4 reads as the mean of the eight, which is **28% larger**. Say "cheap at a resolution
this design cannot see past", not "free".

## Corrections, and what they must reach

**Never silently overwrite.** Mark the correction in place, quote the withdrawn text before
withdrawing it, and date it. Then check the correction actually landed everywhere:

- [ ] **The title.** ⚠️ A title is what travels between sessions. One note kept the retracted figure
      in its own title while its body retracted it — rename the file and record `renamed_from:`.
- [ ] **The frontmatter**, the summary line, and any table caption.
- [ ] **Later sections.** A retracted "10%" survived in the same note's "What this does NOT show",
      three lines after the retraction.
- [ ] **Inbound wikilinks after a rename.** Two notes were left pointing at a filename that no
      longer existed. Nothing in the vault checks a link:
      `grep -rl "\[\[<old-name>\]\]" <vault>`
- [ ] **The repo, if the claim reached it.** A squash commit message cannot be rewritten. Put the
      correction in the code's own header comment instead — with figures regenerable from a fresh
      checkout, never sourced from the vault.

## Disclose the deviations, especially under a null

Say what differed from the pre-registration even when every cell is null — a prereg promised three
predictors stratified and only two were. Say what the stratifier actually measures: one banded on
*price at the decision gameweek*, while the claim under test was about *starting* price, which
attenuates the conjunction toward the null that was obtained.

⚠️ **Absolute totals from `sweepConfig` are not comparable with a real replayed season** — it runs a
five-transfer bank in every season. Paired arm-vs-arm deltas survive that; levels and percentiles do
not.

## The one-way arrow

The vault may name the repo. **The repo may name this store too** — `AGENTS.md:19` says so
outright: *"The user-facing docs never reference the vault; this file and the other agent-facing
surfaces may."* Tracked lines on `origin/main` already do, including five diagnostic
headers carrying `see the vault note "<title>"`. **Do not propose deleting those.**

What may never cross:

- **A machine path — and the store's own LOCATION. Name it; never locate it.** `/home/<user>/…`,
  `/Users/<user>/…`, `<vault-mount>/vault`, `<vault-mount>/vault-worktrees/<branch>`. The user's 2026-08-21
  ruling was "no VM path in the public repo", and the store's path is a VM path. Tracked files may
  name `~/.claude/bin/research-worktree` and nothing more locating than that.
- **A store reference on a USER-FACING surface.** `AGENTS.md`'s permission covers agent-facing files
  only — `README.md`, `docs/` and anything the CLI prints are outside it, and are clean today.
- **A claim sourced only here.** A figure in a repo commit message, review record or code comment
  must be re-derivable from a fresh checkout. Bank the script that produces it — see
  `stats/trajectory_channel.R` and `stats/variance_frontier.R` — or do not cite the figure.

## ⚠️ Then review the CORRECTION, not just the original

**A correction is at least as error-prone as the thing it corrects.** On 2026-08-30 these skills
went through repeated rounds of correction, and **every round introduced defects of its own, each
caught by the review pass after it and never by the author.**

⚠️ **This paragraph gives no counts, deliberately — not of defects and not of rounds.** It tried
three times and undercounted three times: first three rounds of one defect each, then two rounds and
five defects, then three named rounds when `git log --oneline -- skills/` shows six. **Run that
command; it is the record.** `a6a0c0d`'s message is the fullest single account of what one round
broke, and even its diff fixes more than its message names. Four examples worth learning from:

`df4e488` — a rule written to stop merged branches being stranded compared against a moving
`origin/main`, so it stranded every branch by a different route.

`6f52615` — its replacement compared against the merge commit but not the files the branch touched,
still false-stopping on **7 of 16** landed branches; and it found that commit with `git log --grep`,
which returns nothing at all when the PR landed as a *merge* commit, leaving the comparison silently
against the worktree. Separately: a retracted "twelve-cell floor" became "no such rule exists", when
the rule does exist and only its **polarity** was wrong; a "the repo may never name the vault"
reversal licensed naming it *and* quietly licensed the store's own VM path; and a restored `branch:`
exception told reviewers to fault the same-name case the rule tells people to aim for.

⚠️ **Two shapes, and only one is overcorrection.** The twelve-cell denial, the arrow reversal and
the `branch:` exception each travelled past the defect onto the **opposite** error. The two `-D`
bugs did the reverse — each **narrowed the defect without escaping it**, so the same gate
false-stopped landed branches three versions running. **An overcorrection is caught by re-reading
the source; a recurrence is caught only by running the check on a real case.**

⚠️ **This paragraph undercounted three times running, and that is the lesson in it.** `5f43539`:
*"three consecutive rounds of corrections … each introduced a new defect"* — three rounds of one,
when two examples came from a single commit. `1fb6321`: *"two … rounds … introduced **five** new
defects between them"* — still low, because five came off `a6a0c0d`'s headline paragraphs rather
than its diff. `d67e068` dropped the defect total but kept *"three rounds … `df4e488`, `6f52615`,
`a6a0c0d`"*, which is an enumeration and so the same error one clause over; the log shows six.
Withdrawn 2026-08-30 in favour of asserting nothing countable.

**A tally of one's own errors is not exempt from being one.** Each fix here re-derived the quantity
and got it wrong again; the one that held was deleting the quantity and pointing at `git log`. ⚠️ **When
a number about your own work has been wrong twice, stop computing it and cite the source that
holds it.**

So: when you apply review findings, **send the applied diff back through review** rather than
treating the pass as spent. And when you correct a *rule*, ask the question that would have caught
the three reversals — *is the original wrong, or merely stated backwards?* It would not have caught
the other two: a gate that strands branches which have landed is found only by running it against a
branch that has. Withdrawing a real rule costs more than restating it, because the next reader has
no way to recover it.

## Then request review

`fpl-stats-review` for the inference, `fpl-docs-review` for the record and the routing. **Reviews
are wanted by default** on findings and verdicts — run them without asking. Apply what survives your
own judgement, say what you rejected and why, and do not silently drop a finding.
