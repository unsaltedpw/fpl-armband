---
name: pre-register-a-measurement
description: Write and commit a pre-registration before running a measurement — hypothesis, arms, outcome, horizon, gates, decision rule and multiplicity family, fixed before any number exists. Use before any diagnostic, sweep, replay grid or cohort analysis whose result will become a finding.
---

# Pre-registering a measurement

**Write it to the vault and COMMIT IT before any number exists.** The commit timestamp is the
evidence; a prereg written after the first look is not one. Use `templates/finding.md`'s sibling
conventions and name the file `<date>-prereg-<question>.md`.

This discipline is why the in-band nulls are trustworthy. **Every place it slipped is a place the
work had to be corrected or thrown away** — the incidents below are all real.

## Fix these before you run anything

**Hypothesis** — one sentence, falsifiable, in the owner's terms as well as the model's.

**Arms** — every arm, including the baseline and any dose ladder. If an arm deliberately exceeds
the model's stated resolution (`MinSeparableGain`), say so and say why.

**Outcome** — the exact quantity, with its unit and its denominator. ⚠️ `Score` and
`PointsPerTenth` are **per-gameweek**. A season-scaled numerator over a fixed-window denominator is
the enabler bug: it produced `−0.095` where the truth was `−0.038`.

⚠️ **HORIZON. Name it explicitly.** The enabler measurement's horizon was *never pre-registered*,
which is how a window mismatch survived into a finding.

**Gates** — any row filter, with its threshold. ⚠️ **If you gate rows, emit the UNGATED column
too.** The trajectory run gated rate trends on ≥900 minutes and wrote `NA` into both rate columns on
all **55,669** gate-failed rows of 83,994. No ungated version exists anywhere, so "does the gate
move the *estimate*, or only shrink the *sample*?" is now **permanently unanswerable from that
archive**. One extra column at write time would have settled it.

**Detection threshold** — `t_crit(df) × SE`, season-clustered, with `df` stated. Six seasons → df 5,
t = 2.571. Four seasons → df 3, t = 3.182. Write the threshold down *before* you see the estimate.

**Cell count** — how many cells, and how many *clusters* the SE is computed over.
⚠️ **The twelve-cell rule is a CEILING OF UNTRUSTWORTHINESS, not a floor you clear.** The record's
rule is that *any verdict reached at twelve cells or fewer is unverified and must be re-judged* —
`stats/findings/2026-08-13-benchshape.md`, restated in `internal/backtest/availability_test.go`'s
`statusAt` header, which gives the provenance: a transfer-threshold sweep went from "noise" at
twelve cells to t = +3.36 at twenty-four. `AGENTS.md`'s closed-lines list records the consequence —
**"Twelve cells could not resolve 37 points a season."**

⚠️ **This line has now been wrong twice, in opposite directions.** It first read "the twelve-cell
floor", which inverts the polarity into a bar you pass. The correction then claimed no such rule
existed, which is worse — it does, in two tracked files. Read it as: **≤ 12 cells ⇒ unverified.**
And `AGENTS.md`'s cell row still applies on top: *"Take the cell count from the figure, never from
this row"* — state your own design's count and clusters, and carry no remembered number in either
direction.
⚠️ A six-cluster design cannot establish a null. It can support "ruled out for shipping"; it cannot
support "ruled out". Decide which you are entitled to claim before you run, not after.

**Multiplicity family** — how many comparisons, enumerated. ⚠️ **List every predictor × stratum you
intend to report.** The trajectory prereg promised all three predictors stratified by price band;
only two were, so a 17-comparison family became 13 and the deviation went undisclosed until review.

**Decision rule** — what result means what.

## ⚠️ Write a decision rule the design can actually support

This is the failure mode that costs the most, because it is invisible until the numbers arrive.

The variance-frontier prereg required both that the SD range resolve **and** that the arm maximising
`P(season points > p90)` differ from baseline. The first clause was met three times over. **The
second failed** — baseline won outright at p90 and p95. But the clause was never testable: it
computed `P(>T)` from the spread of **six season totals per arm**, which measures between-season
difficulty, not the strategy variance the first clause measures, and n=6 across ten arms cannot
discriminate.

Before committing a clause, ask: **what data does this compute over, and is that the same quantity
the hypothesis is about?** If a clause would be uninformative under *every* outcome, it is not a
decision rule.

⚠️ **A badly specified clause still counts.** Report that it failed, then explain why it was the
wrong test. Do not quietly re-read it as satisfied.

## Amendments

Amend **only before a number exists**, in a marked, dated `## ⚠️ AMENDED` section. Never edit the
original ask in place — append. An amendment made after the first look is a post-hoc hypothesis and
must be labelled one.

## Bank the analysis alongside the cells

Write the analysis script beside its output directory and **name it in `measured_at`**. A finding
whose thresholds and multiplicity values exist only in one un-backed-up scratch directory is not
reproducible, and two findings reached exactly that state before the scripts were moved into the
repo's tracked `stats/`.

## Then commit it, before you run

```bash
git -C <vault-pair> add memory/<date>-prereg-<question>.md
git -C <vault-pair> commit -m "Pre-register <question>, with <what is fixed> fixed in advance"
```

Say in the finding note afterwards that the prereg was **committed first**, and link it. That link
is what makes the null worth anything.
