# Table facts — results (eval/TABLE_FACTS.md v1, run 2026-10-04)

69 items (one value printed in a table of each of 69 random 2025–26 NF/PPCF papers), 5 arms, all 345 runs passed
the audit (0 voids), code-graded under the frozen protocol.

| Arm | Access | Correct | 95 % CI | Abstained |
|---|---|---|---|---|
| A | model only | 2/69 (3 %) | 1–10 % | 66 |
| B | corpus, text only | **69/69 (100 %)** | 95–100 % | 0 |
| E | corpus + table layer | **68/69 (99 %)** | 92–100 % | 0 |
| C | web search | 39/69 (57 %) | 45–68 % | 26 |
| F | corpus + tables + web | 68/69 (99 %) | 92–100 % | 0 |

**Co-primary (exact McNemar, Holm):**
- **E vs B: −1.4 pts (0 vs 1 discordant), p = 1.0 → inconclusive.** Both arms are at the ceiling. The text corpus
  already contains every key (covariate: key in the text passages for 69/69 items), so structured table access had
  nothing left to add on this item set. E used the table commands in 13/14 batches and F in 14/14: the layer is used,
  but it does not change accuracy here.
- **E vs C: +42.0 pts (29 vs 0 discordant), p = 3.7 × 10⁻⁹ → corpus better.** Every item the web got right, the
  corpus also got right.

**Secondary:** F vs C +42.0 (29/0); B vs C +43.5 (30/0); E vs A +95.7; F vs E 0 (adding the web to the corpus changed
nothing).

**Sensitivities (pre-registered):**
- Items where the web arm fetched the paper itself (a DOI URL without a bot challenge): 16. The web scored 8/16 and
  the corpus 16/16 (+50, p = 0.008).
- The other 53 items: web 31/53, corpus 52/53 (+39.6).
- The corpus advantage holds when the web reached the paper. It is not only a bot-wall artefact: reading the paper
  page did not reliably give the web arm the table value.
- Void-excluded: identical (no voids). The caption-overlap split cannot be interpreted, because only 1 item was ≥ 0.5.

## Grader checks (pre-registered)
- **Random hand check:** 28 non-A grades (10 %, seed 20261023), all 28 agree with a human reading.
- **Review of every wrong, non-abstaining answer (6):**
  - **4 grader false negatives:**
    - "14.91 vol%" for key 14.91 %: E, F and C (t048). `vol%` is not parsed.
    - "5 × 10⁻⁶" in Unicode superscripts, C (t034).
  - 1 strict-rule miss: C t002 wrote "17174.3 J/mol (≈ 0.178 eV)". The first reading has the wrong unit, so the
    frozen rule grades it wrong; the eV value in parentheses is within 2 %.
  - 1 true error: C t066 gave 700 °C, the allowable maximum, instead of the tabulated 671 °C.
- **Corrected (descriptive, not the primary):**
  - B 69, E 69, F 69, C 40 (41 if t002 is credited).
  - E vs B: 0. E vs C: +41 (or +40).
- The verdicts do not change. The false negatives were spread across arms and slightly favoured the web.

## Conduct
- Freeze amended twice before grading (TABLE_FACTS §9):
  - an untracked dashboard file was edited by parallel work during the run;
  - the protocol's appended §9 note was verified append-only.
- No answer, item, prompt or grader changed.

## Reading
- On facts printed in tables of recent papers the corpus holds, the corpus (with or without the table layer) answers
  ~99–100 % vs the web's 57–59 %. This is the largest and cleanest gap in the programme, and it replicates the FACTS
  "body" result (+28.6 there).
- The table layer did not measurably help, because the text passages already carry the table text and the agent
  finds it. Its value, if any, would show on tables the text layer garbles, or on multi-table aggregation; neither
  was tested here.
- Scope: the extractor's coverage slice (88 % of captioned tables, ruled tables), and papers in the corpus.
