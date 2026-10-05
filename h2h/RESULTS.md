# Q&A head-to-head — results (2026-10-03)

Protocol: `eval/HEAD2HEAD.md` (v2, frozen in commit b31ea9a before any arm ran; post-freeze amendments 1–2 in §17,
code-only). Full tables: `RESULTS_tables.md`; pre-registered chart: `chart.png`; raw data `data/h2h/results.json`.
Every arm on every set is reported. Same model (Claude Opus 5.5) in every arm; graded blind by two judges (Gemini
3.1 Pro, Claude Opus 5.5) on neutralised claim lists; **correct = both judges say correct** (strict consensus).

## Execution
140 questions × 4 arms = 560 answers; 112/112 batches passed the use-based audit on the first attempt; 0 missing or
malformed answers. Judges: 560 gradings each; Cohen's kappa 0.77, 74 disagreements. Keys: 10 of the 60 P questions
contested (the two independent keys conflict); 1 key ungrounded (h005).

## Primary (pre-registered): B (+ corpus) vs C (+ web) on set P (60 researcher questions)

| A model only | B + corpus | C + web | D + corpus + web |
|---|---|---|---|
| 16/60 (27 %) | 28/60 (47 %) | 31/60 (52 %) | **37/60 (62 %)** |

B vs C: 9 vs 12 discordant, −5.0 points (bootstrap 95 % CI −20 to +10), exact McNemar p = 0.66.
**Verdict (pre-registered rule): inconclusive — not evidence of equivalence.**

## Secondary contrasts on P (exact McNemar; difference in points, 95 % CI)
| Contrast | Discordant | Difference | p |
|---|---|---|---|
| D vs C (corpus + web vs web) | 13 vs 7 | +10.0 (−5 to +25) | 0.26 |
| D vs B (corpus + web vs corpus) | 11 vs 2 | +15.0 (+3 to +27) | 0.023 |
| B vs A | 17 vs 5 | +20.0 (+5 to +35) | 0.017 |
| C vs A | 18 vs 3 | +25.0 (+12 to +38) | 0.0015 |
| D vs A | 26 vs 5 | +35.0 (+18 to +50) | 0.0002 |
Sensitivity, P without the 10 contested keys (50 questions): A 28 %, B 48 %, C 52 %, D 70 %; D vs C +18 points,
p = 0.049 (a sensitivity analysis, not the primary). Each judge alone gives the same ordering (D > C ≥ B > A).

## Other sets (strict consensus)
| Set | n | A | B | C | D |
|---|---|---|---|---|---|
| S1-A seed, in scope | 19 | 79 % | 79 % | 84 % | 100 % |
| S1-OOC not answerable | 6 | 67 % | 67 % | 67 % | 67 % |
| S1-B known-paper lookup ⚑ | 25 | 12 % | 92 % | 60 % | 92 % |
| S2 exact facts, 2026 papers ⚑ | 30 | 0 % | 100 % | 23 % | 100 % |
⚑ favours the corpus by construction (questions written from corpus papers): B vs C on S2 +77 points
(p < 10⁻⁶), on S1-B +32 points (p = 0.008). A fair version of this comparison (random papers) is `eval/FACTS.md`.

## Hallucination and citations (secondary; all 140 questions)
Answers contradicting the key on P (either judge): A 12, B 12, C 14, **D 5**. Over the answerable questions, flagged
by both judges: A 10, B 4, C 11, D 1. Answers with a fabricated citation (doi.org/Crossref/OpenAlex check): A 4,
B 1, C 1, D 1 of 140. Abstentions on P (both judges): A 4, B 4, C 1, D 2.

## Reading
The corpus alone is about as good as web search on researcher questions (not shown to be better or worse). The
combination is the best setup on every set and contradicts the key least; with 60 primary questions its lead over
web search alone (+10 points) is not statistically established. Every tool arm clearly beats the model alone. The
corpus's decisive advantage on exact facts from recent papers is, in this study, by construction; the fair test is
running. Limitations: 60 primary questions (low power for 10-point effects); keys built by models (two independent
keys, contested ones reported separately); one model family in the arms.
