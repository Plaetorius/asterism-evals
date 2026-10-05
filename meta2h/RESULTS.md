# Meta-analysis head-to-head — results (2026-10-03)

Protocol: `eval/META2HEAD.md` (v1 after review 1, pilot §9, frozen in commit fc74e85 before any test run).
Raw data: `data/meta2h/results.json` (git-ignored); every run transcript in `data/provenance/meta2h_transcripts/`.

## Execution
48/48 arm runs passed the use-based audit on the first attempt (0 voids). 35 verifier jobs (first pass + 25 %
second verifier); 1 verifier batch needed its pre-registered re-run (t02_v1_b04). 0 points left unverified.

| Arm | Runs | Mean time / run | Notional cost | Claimed points | **Verified relevant (V)** | Precision (median) | Distinct verified sources | Inaccessible to verifier |
|---|---|---|---|---|---|---|---|---|
| A model only | 12 | 2.1 min | $5 | 30 | **6** | 0.29 | 6 | 16 |
| B + corpus | 12 | 12.9 min | $59 | 121 | **63** | **0.97** | 46 | 1 |
| C + web | 12 | 21.3 min | $72 | 141 | **84** | 0.77 | 53 | 28 |
| D + corpus + web | 12 | 15.3 min | $70 | 116 | **75** | 0.82 | 47 | 1 |
Verifier jobs: $43. No arm cited a work that does not exist (`source_not_found` = 0 everywhere).

## Pre-registered primary (task level, 6 tasks, exact Wilcoxon, Holm)
| Comparison | Per-task differences in V (t01…t06) | Totals | p | Verdict |
|---|---|---|---|---|
| B vs C | −2, −3, +1, +5, −10, −12 | 63 vs 84 | 0.31 | **inconclusive (not evidence of equivalence)** |
| D vs C | −5, +6, −1, +9, −5, −13 | 75 vs 84 | 0.81 | **inconclusive (not evidence of equivalence)** |
Secondary: every tool arm beats the model alone on every task (B, C, D vs A: p = 0.031, the minimum attainable with
6 tasks); D vs B +12 points total (p = 0.44); distinct verified sources B 46 / C 53 / D 47 (B vs C p = 0.50).
Accessibility sensitivity (counting `inaccessible` relevant points as verified): B 64 vs C 106 (p = 0.16) — web
points are more often behind paywalls, so this widens C's lead without changing the verdict.

**Verifier agreement:** second verifier on 82 points, exact agreement 0.70, kappa 0.55 — **below the
pre-registered 0.6 flag**; agreement on whether a point counts (supported + relevant) 0.85. The primary uses the
first pass as pre-registered; this lower agreement is a limitation of every number above.

## Per task (V, run 1 / run 2)
| Task | A | B | C | D | Union of verified points |
|---|---|---|---|---|---|
| t01 L-H threshold vs Martin | 0/0 | 4/7 | 3/10 | 5/3 | 30 |
| t02 isotope mass exponent | 1/1 | 9/7 | 9/10 | 13/12 | 51 |
| t03 pedestal width coefficient | 1/0 | 8/8 | 8/7 | 8/6 | 26 |
| t04 λq vs B_pol (Eich) | 0/0 | 7/3 | 3/2 | 4/10 | 22 |
| t05 current-quench time | 1/0 | 2/2 | 10/4 | 7/2 | 20 |
| t06 runaway threshold field | 1/1 | 5/1 | 9/9 | 3/2 | 22 |
The web arm's lead comes from the two disruption/runaway tasks, where much of the literature is published outside
the corpus's two journals (Nuclear Fusion, PPCF); the corpus arms lead on t04 and tie on t03.

## Pooled estimates vs the reference (descriptive; references come from the task generator's memory)
- t04 (reference exponent −1.19): B −1.15, D −1.24, C −0.89 (run 1).
- t06 (reference E/E_CH ≈ 2–5): B ≈ 5, C ≈ 7.8, D 3–5 (run 1).
- t03 (reference C = 0.076): all tool arms ≈ 0.09 (device means), each noting 0.076 as the canonical value.
- t05 (reference ≈ 1.7 ms m⁻²): every arm, including A, ≈ 1.67–1.7.
Full syntheses for every run are in `results.json`.

## Reading
Evidence gathering is not decided by this study (6 tasks, low power, verifier kappa 0.55). What is measured: the
corpus arm's points are almost all verifiable (precision 0.97 vs 0.77), the web arm reaches more sources overall
and especially outside NF/PPCF, and no arm fabricated sources. Agents in both corpus and web arms reported that
much of the needed data exists only in figures — the motivation for `eval/FIGURES.md`.
