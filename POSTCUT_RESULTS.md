# Post-cutoff study — results (eval/POSTCUT.md v1, run 2026-10-04)

**Run.** 185 items from 100 random NF/PPCF papers whose earliest public appearance is on or after 2026-06-01, the
answering model's training cutoff. 4 arms; all 148 runs passed the audit (0 voids, 0 wrapper errors). Graded by the
frozen grader. Fact items are ABS + BODY + TABLE (124 items, 75 papers).

| Items | A model only | B corpus | C web | D corpus + web |
|---|---|---|---|---|
| **Facts (124)** | 3 (2 %) | **121 (98 %)** | 60 (48 %), 58 abstained | 118 (95 %) |
| ABS (44) | 3 | 44 | 38 | 43 |
| BODY (56) | 0 | 53 | 16 | 52 |
| TABLE (24) | 0 | 24 | 6 | 23 |
| LOOKUP (61) | 0 | 61 | 47 | 61 |

**Co-primary** (paper-clustered sign-flip permutation, Holm):
- **B vs C: +49.2 points** (61 vs 0 discordant items), **p = 5 × 10⁻⁶ → corpus better than web.**
- **D vs C: +46.8 points** (59 vs 1), **p = 5 × 10⁻⁶ → corpus + web better than web.**

**Secondary:**

| Comparison | Result |
|---|---|
| B vs C, ABS | +13.6, p = 0.031 |
| B vs C, BODY | **+66.1** |
| B vs C, TABLE | **+75.0** |
| B vs C, LOOKUP | +23.0, p = 1.2 × 10⁻⁴ |
| D vs B | −2.4, p = 0.38: adding the web did not help |

Every tool arm beat A.

**Sensitivities (pre-registered):**

| Slice | Items | Corpus (B) | Web (C) | Difference |
|---|---|---|---|---|
| No arXiv preprint found for the paper at all | 121 | 118 | 57 | +50.4 |
| The web arm fetched the paper and saw the key value in the fetched text | 17 | 17 | 17 | 0 |
| The web arm did not | 107 | 104 | 43 | +57.0 |
| Abstract-only papers (corpus has metadata + abstract only) | 12 | 12 | 10 | +16.7, p = 0.5 |
| Full-text papers | 112 | 109 | 50 | +52.7 |
| Key also found in the PDF's own text (independent of our extraction) | 122 | 119 | 60 | +48.4 |

- **The web loses because it does not reach the text.** When the web agent reached the value, it was as accurate as
  the corpus.
- **Question copy length and title overlap:** the corpus leads by 44–62 points in every stratum, so the lead is not
  driven by wording that suits lexical search.
- **By month:** the lead is +44 to +63 points in every month with more than one item.
- **Batches without voids:** identical to the main result, because there were no voids.

**Contamination check.** The model alone answered 3/124 (2.4 %). On the pre-cutoff FACTS slice it answered 3/129
(2.8 %, 3/106 pre-cutoff facts). At this floor (about 97 % abstentions) the check cannot detect memorisation; it
shows only that the bare model does not offer these values.

## Grader checks (pre-registered)
- **Random hand check:** 37 B/C/D grades (10 %, seed 20261107). 36 agree with a human reading; 1 is a grader false
  negative (below).
- **Review of every wrong non-abstaining answer (19):**
  - **7 grader false negatives** (B 2, C 1, D 4):
    - an answer written as "R² = 0.9998", "R^2 = 0.990" or "b² = 0.20" is read as the number 2 (B q058; C q113; D q058,
      q113, q181);
    - "α = Z²f ≈ 0.5" is read as 2 (B q020);
    - a unit after a bracketed range, "σ ∈ [0.056, 0.504] m", is not read (D q083).
  - **12 true errors** or strict-rule misses:
    - C gave a different value (5) or a different paper (3), and once cited the arXiv DOI of the right paper (q065;
      counted wrong by rule);
    - B and D answered "~1 T" for 0.9 T (q107);
    - D gave ≈6 for 10 (q137).
- **Corrected (descriptive):** B 123/124, C 61/124, D 122/124. B vs C becomes +50.0. Verdicts unchanged.

## Reading
On exact facts from papers that became public only after the model's training cutoff, the corpus answered 98 %
and web search 48 %. That is the largest gap in the programme.
- The gap is concentrated where the answer lives in the full text: body +66, tables +75.
- Lookup of a described paper: +23.
- Facts stated in abstracts: +14.
- On papers whose full text the corpus lacks (12 items), the corpus still matched or beat the web from abstracts and
  metadata.
- The mechanism is access. Wherever the web agent did fetch the paper and the value was in what it read, it was as
  accurate as the corpus (17/17); in the other 107 items it scored 40 % against 97 % (58 abstentions). The 17-item
subset is selected on the web arm's success, so this is descriptive, not causal.

**Scope:**
- *Nuclear Fusion* and *PPCF* papers from June to October 2026.
- Claude Opus 5.5 with its built-in web tools.
- Model-written questions, validated against the sources, with a 25/25 hand check.
