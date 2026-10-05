# Recent exact facts and paper lookup — a fair test (pre-registration v0, 2026-10-03)

**Question.** In the Q&A head-to-head the corpus won outright on exact facts from 2026 papers (100 % vs 23 %) and on
known-paper lookup (92 % vs 60 %), but those questions were written *from corpus papers* ("favours the corpus by
construction"). This study asks the same thing without that bias: on facts from a **random sample of recent papers
in the corpus's stated scope** — including papers we do not hold in full text — does the corpus beat web search?

## 1. Sampling frame and sample (fixed before anything is run)
- Frame: every research article of *Nuclear Fusion* and *Plasma Physics and Controlled Fusion* published 2025-01-01
  to 2026-10-01 in the catalog (Crossref-complete for both journals): 1,875 papers; full text held for 1,471
  (78 %; PPCF ≈ 55 %), the rest abstract-only (not open access).
- Sample: **100 papers**, seeded random draw (seed 20261013), without regard to full-text status.
- Coverage is recorded per paper (full text held / abstract only) and analysed, never used to select.

## 2. Questions (written blind; answer keys come from the paper itself)
For each sampled paper a headless Claude process with **no tools except writing one file** (it cannot search the
corpus or the web, and does not know the arms) receives only that paper's text and writes:
- **ABS** — one exact quantitative fact stated in the **abstract** (every paper);
- **BODY** — one exact quantitative fact stated in the **body text but not in the abstract** (papers with full text
  only; abstract-only papers have no body to key from);
- **LOOKUP** — for a seeded 60 of the 100 papers: "which recent paper reported …? Give its DOI", describing the
  finding without the title, author names or DOI.
Each item carries: the question (self-contained: device, quantity, conditions, year range "published 2025–2026"),
the key value and unit (or DOI), and the **verbatim quote** that states it.
**Automatic validation before the freeze:** the quote must occur in the source text (whitespace-normalised); the
key value must occur in the quote; ABS quotes must be in the abstract and BODY quotes must not; questions must not
contain the title (≥ 5 consecutive title words), any author surname or the DOI. Items failing validation are
regenerated once, then dropped and listed (before any arm runs). The writer returns null when the abstract (or
body) states no specific number — the practice papers showed many abstracts are qualitative — so the ABS count is
measured, not fixed (pilot: 0/3 abstracts with a specific number; BODY 2/2 full-text papers; LOOKUP 3/3).
Sample drawn: 100 papers from a frame of 1,873; 88 with full text (frame 78 %; recorded, not redrawn).

## 3. Arms and execution
A model only · B + corpus · C + web · D + corpus + web — same model (Claude Opus 5.5, `--effort xhigh`), same brief as
the Q&A head-to-head (answer + value + citations, ≤ 200 words, abstain allowed), same isolation and use-based audit,
against the full-corpus snapshot of the meta-analysis study (3,494 papers; abstracts of all 24,101 works, so the
corpus arm can find abstract-only papers too). Questions are shuffled (seed 20261014) into batches of 5;
A, B, C, D run each batch concurrently.

## 4. Grading — objective, no judge model
- **ABS/BODY:** correct if the answer's `value` (or, if absent, the first number with a unit in the answer) equals
  the key within ±2 % (or within the rounding of the key as printed) **after unit conversion** (pint, with the
  project's offset-unit handling); a missing or incompatible unit is incorrect; an abstention is incorrect.
- **LOOKUP:** correct if the key DOI appears (normalised) in the answer's citations or text.
- The grader is code, frozen with the rest; every item's grading trace is stored. A random 10 % of gradings is
  hand-checked by the lead and reported (agreement), with no change to the rules after seeing results.

## 5. Pre-registered analysis
- **Primary:** B vs C, all items pooled, exact two-sided McNemar on paired correctness. Verdict "corpus better / web
  better" if |difference| ≥ 10 points and p < 0.05, else "inconclusive (not evidence of equivalence)".
- **Key secondary:** D vs C (same rule).
- **Pre-specified breakdowns (descriptive, with Wilson CIs and paired tests):** ABS, BODY, LOOKUP; full text held vs
  abstract only; publication date before vs after the model's training cutoff (2026-06) and by quarter; journal.
- Abstention and wrong-answer rates per arm; fabricated DOIs (checker from the Q&A study) per arm.
- Power: with ~240 items, a 10-point difference at the observed discordance is detectable; the S2 effect seen
  earlier (77 points) would be detectable with far fewer.
- Everything is reported; nothing re-run or dropped after the freeze except audit voids (max 2 attempts).

## 6. Controls
Adversarial review of this protocol and the code before the freeze; a pilot on 3 practice papers outside the sample
(all four arms + grader) before the freeze; `FREEZE.json` (sample, items, prompts, code, snapshot) committed before
the first arm run; every transcript and grading trace kept for the paper.

## 7. Review 1 (adversarial, before the freeze) and protocol v1

The reviewer blocked the freeze (2 critical, 4 high). Changes — all before any test question was answered:
- **Primary redefined (§5 v1):** B vs C on **ABS + BODY** items (exact facts), tested with a **paper-clustered**
  sign-flip permutation test (items of one paper are not independent; exact for ≤ 16 non-zero papers, else
  200,000 seeded permutations); D vs C co-primary; Holm over the two; verdict needs Holm significance and
  |difference| ≥ 10 points. LOOKUP (near ceiling in the pilot) and the **abstract-only slice** (the slice the corpus
  cannot win by holding the text) are reported separately; BODY is labelled "full-text search" because BODY exists
  only for papers the corpus holds. Void-excluded sensitivity (batches voided in any arm dropped).
- **Grader (§4 v1):** DOIs parsed from any citation style (publisher URLs with /meta, /pdf, query strings,
  URL-encoding, markdown) — the old parser undercounted web citations, a bias toward the corpus; LOOKUP credited if
  the DOI is cited, or is the only DOI in the text. Values: only the `value` field's first reading counts (the
  answer text only when `value` is empty, first number only); a predicted range wider than 1.2 × the key's width
  (+ tolerance) is wrong; no unit is wrong unless the key is dimensionless; units normalised (superscripts, braces,
  "per", "5-keV", thousands separators). Every key must grade itself correct (pre-freeze self-test).
- **Items (§2 v1):** the writer must paraphrase (no run of > 4 words from the paper except names), never state the
  key in the question; validation rejects copies of ≥ 7 consecutive words, keys whose value is not next to a
  compatible unit in the quote, surnames of ≥ 2 letters (case-sensitive), key values stated in the question;
  title-word overlap is recorded as a covariate; one regeneration for papers with a rejected item, then drop;
  batches never contain two items from the same paper. The first writer pass (82 sample + 3 practice papers) was
  run under v0 instructions and is archived as void (`data/facts/void_qgen_v0/`), unused.
- **Runs:** audit call cap 20 per question (brief says 15); an attempt fails if its answer file is not valid JSON
  covering every qid; MCP config written once (no concurrent rewrites).
- **Freeze** also hashes the shared audit modules, the corpus wrapper, `units.py`, the pint version, all writer
  outputs and the rejected list, and asserts every sampled paper has the same full-text status in the arm snapshot;
  `grade` and `score` verify it.
Known limitations kept: keys come from the corpus's text layer of the publisher PDF (text-layer artefacts could
in principle favour the corpus — mitigated by the value-unit adjacency check; residual risk reported); questions say
"published 2025–2026", which helps every arm (web query terms, corpus year filter).
Pilot under v1 (3 practice papers outside the sample; 5 items: 2 BODY, 3 LOOKUP; 8 runs, all audits passed): one
practice paper needed its regeneration (8-word copy); a unit-parsing bug ("1 kHz appeared" read the word as part of
the unit) was found by the validator and fixed in the grader (units grow token by token only while they parse),
with a test. All 20 pilot gradings were checked by hand and are correct. Pilot scores are not results.

## 8. Item set (final, before the freeze)

Writer v1 over 100 sample papers (100/100 runs ok), validation, one regeneration for the 26 papers with a rejected
item (per item: passing first-pass items kept), validation again. **188 items from 97 papers: 45 ABS, 84 BODY,
59 LOOKUP; 20 dropped** (listed in `private/rejected.json`: numbers written as words, units that are not units,
copies, key stated in the question, quotes not found). 174 items come from full-text papers and 14 from
abstract-only papers (the sample has 12 abstract-only papers, by chance below the frame's 22 %), so the unbiased
abstract-only slice is small and is reported descriptively. Paraphrase covariates: longest run copied from the paper
median 4 words (max 6); title-word overlap median 0.36. 38 batches of ≤ 5, no paper twice in a batch.
Validator and grader bugs found on the real items and fixed before the freeze (each with a test; every fix applies
to keys and answers alike): inline maths broken across lines in the text layer ("3\n/\n1") → quotes compared
without whitespace/markup after Unicode NFKC; "3e+19" (plus sign) not parsed; "3 · 10^19" (middle dot); values
with uncertainties "(2.9 ± 0.06) mm"; the unit capture swallowing the next numbers and digits inside names (t0,
H98); ranges "10%–15%"; trailing punctuation in units; "◦" as degree; units written as words for pure numbers
("factor", "dimensionless", "Greenwald fraction") treated as dimensionless.

## 9. Execution note: usage-limit interruption (2026-10-03)

Arm runs started 15:40. From batch 20 onward (~17:05) the Claude session limit returned HTTP 429 on every call
("You've hit your session limit"), so 95 attempts ended in < 1 s with no tool call and no answer file. The chain
was stopped; those 95 records were identified from their transcripts (429 / "session limit") and moved to
`data/facts/void_usage_limit/` as infrastructure failures, so they do not count as attempts. 80 earlier runs
(batches 1–19 complete, batch 20 partial) are kept as run. The chain resumed at 23:43 with the frozen materials
unchanged (`verify_freeze` passed), running only the batches without a passing attempt.
Post-freeze note: §9 above was appended to this protocol after the freeze (an execution record, no rule changed);
`verify_freeze` then reported only `protocol_sha256` changed, the diff against the freeze commit 81ae369 is 9 added
lines, and the freeze was re-written for that single key before grading.
