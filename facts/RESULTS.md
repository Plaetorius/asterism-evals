# Recent exact facts and paper lookup — results (2026-10-04)

Protocol: `eval/FACTS.md` (v1 after adversarial review, frozen in commit 81ae369; post-freeze: an execution note was
appended, §9, and the freeze re-written for the protocol hash only, commit d455a9e). Raw data `data/facts/results.json`,
grading traces `data/facts/private/grades.json`.

## Design in one paragraph
100 papers drawn at random (seed 20261013) from all 1,873 research articles of *Nuclear Fusion* and *PPCF* published
2025-01 to 2026-10, regardless of open access. A tool-less writer saw one paper at a time and wrote paraphrased
questions (no copied runs > 4 words except names); automatic validation; 188 items from 97 papers: 45 ABS (fact in
the abstract), 84 BODY (fact only in the body text; full-text papers only), 59 LOOKUP (give the DOI). Four arms, same
model and brief; grading by code (unit-aware ±2 %, DOI match), no judge model.

## Execution
152/152 arm runs passed the audit (95 attempts lost to a Claude session limit were infrastructure failures,
archived, and re-run after the reset — §9). No batch was voided.

## Primary (pre-registered): exact facts (ABS + BODY, 129 items, 92 papers), paper-clustered test, Holm

| A model only | B + corpus | C + web | D + corpus + web |
|---|---|---|---|
| 3/129 (2 %) | **120/129 (93 %)** | 95/129 (74 %) | **121/129 (94 %)** |

- **B vs C: +19.4 points** (28 vs 3 discordant items), paper-clustered permutation p = 1.0 × 10⁻⁵ → **corpus better than web**.
- **D vs C: +20.2 points** (28 vs 2), p = 5.0 × 10⁻⁶ → **corpus + web better than web**.
Both pass Holm and the 10-point bar. This is the first pre-registered comparison in our studies where the corpus
wins on a fair sample.

## Where the difference comes from (pre-specified breakdowns)
| Items | n | B corpus | C web | D both | B vs C |
|---|---|---|---|---|---|
| ABS — fact stated in the abstract | 45 | 93 % | 91 % | 96 % | +2.2, p = 1 |
| BODY — fact only in the body text | 84 | 93 % | 64 % (28 abstentions) | 93 % | **+28.6, p = 5 × 10⁻⁶** |
| LOOKUP — give the DOI | 59 | 98 % | 83 % | 100 % | +15.3, p = 0.012 |
| Abstract-only papers (ABS + LOOKUP) | 14 | 100 % | 93 % | 100 % | +7.1, p = 1 |
| Facts published ≥ 2026-06 (after the model's training cutoff) | 23 | 96 % | 57 % | 91 % | — |
| Facts published < 2026-06 | 106 | 92 % | 77 % | 94 % | — |
The model alone answers almost nothing (it abstains on 125/129 facts) — these papers are recent.

**Reading.** On facts any search engine can see (abstracts) web search is as good as the corpus. The corpus wins on
facts inside the paper body (+29 points) and on identifying the paper (+15). The web arm's 28 BODY abstentions
come from not reaching the full text: of its 171 fetches of iopscience.iop.org, 46 (27 %) hit IOP's bot challenge
and many others returned landing pages; it fell back on preprints (arXiv 109 fetches, OSTI 63), which often lack
the published details. **The corpus's advantage is reliable access to the full text of open-access papers that a
web agent often cannot read** — a real, product-relevant difference, measured on a random sample. The BODY stratum
exists only for papers whose full text the corpus holds (88 % of the sample, 78 % of the frame), so the pooled
primary weights that stratum by its share; the ABS and abstract-only results show no corpus advantage where the web
can see the same text.

## Grader checks
- Pre-registered random 10 % hand check (75 gradings, seed 20261017): **74/75 agree**; the 1 disagreement is a
  grader false negative against the corpus arm.
- Every incorrect answer containing the key's number was hand-reviewed (reviewer's request): 21 answers (A 1, B 9,
  C 4, D 7); 2 are genuinely wrong (A f185; C f044, a range rejected by the pre-registered width rule), **19 are
  grader false negatives** — unit parsing kept trailing words ("% neon", "days (second …)", "3600 K (W …)"), read
  "(y,2)" in "H98(y,2)", or pint read the key unit "W mK⁻¹" as per-millikelvin — by arm B 9, C 3, D 7, A 0.
  These errors go **against the corpus arms**; the frozen grade remains the primary result. Sensitivity with the
  hand corrections: B 129/129, C 98/129, D 128/129 (B vs C ≈ +24 points).

## Limitations
One random sample of 100 papers (97 contribute); the BODY stratum is defined by the corpus holding the text; the
web arm's access is the real-world access of an agent with WebSearch/WebFetch on 2026-10-03/04 (publisher bot
walls included); keys come from the corpus's text layer of the published PDF (value–unit adjacency checked); one
model family; the grader's unit parser has a measured false-negative pattern (biased against the corpus arms).
