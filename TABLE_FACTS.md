# Facts in tables — does a table layer help, and does the corpus beat the web on table facts? (pre-registration v1, 2026-10-04)

**Question.** The corpus now has a table layer: every captioned table of the 3,494 full-text papers reconstructed
into rows and cells (`src/fusion_corpus/tables/`) and searchable with two new corpus commands (`tables`, `table`).
(1) Does **structured table access** (the table layer) help an assistant find values printed in tables, compared
with the text-only corpus? The text corpus already contains each table's text as flattened lines (and every key is
validated in the PDF text), so this measures retrieval of *structured* table data — not whether the layer adds
information that was absent. (2) With it, how does the corpus compare with web search and with the model alone **on
facts printed in tables of papers the corpus holds and the extractor covers** (the layer's coverage slice: 88 % of
captioned tables; ruled tables only)?
Same harness and rules as `eval/FACTS.md` (whose design, review and lessons carry over).

## 0. The table layer and its quality (built and checked before this protocol was frozen)

- Method (`src/fusion_corpus/tables/extract.py`, free, local, sandboxed): journal tables here have no vertical lines,
  so PyMuPDF's `find_tables` found 0/40 tables with the line strategy and garbled the text strategy (split words,
  caption mixed in). Tables are therefore reconstructed: the region between the horizontal rules below a
  "Table N." caption; rows from word positions; columns from vertical white channels shared by the body rows;
  superscripts/subscripts kept from character-level font data (`10^{19}`, `T_{e}`) so exponents are not flattened
  ("1019"); a band of running text (≥ 10 words with only word-sized gaps) ends a table.
- Coverage on all 3,494 full-text papers: **3,893 of 4,429 captioned tables reconstructed (87.9 %)**, 30,080 rows;
  not reconstructed: 457 without rules below the caption, 79 single-rule or empty; 0 extraction errors.
- Quality checks during development: visual comparison of random tables with their rendered images; defects found
  and fixed (exponents flattened; rules wider than the caption missed; figure-caption text spilling into a table;
  word-count prose rule cutting wide headers → gap-based rule); a residual defect: adjacent columns separated by
  less than ~4 pt can merge into one cell (values intact and in order) — 1 of 4 sampled tables showed it.
- Access: `corpus tables "<query>"` (FTS over caption, header and cells, symbols normalised) and
  `corpus table <id>`; output is the reconstructed table with page and DOI.

## 1. Sample
- Frame: research articles of *Nuclear Fusion* and *PPCF* published 2025-01-01 to 2026-10-01 whose full text is in
  the corpus and which have at least one reconstructed table, **excluding the 100 papers of the FACTS sample**.
- **100 papers**, seeded random draw (seed 20261019); for each, one table drawn at random (seed 20261020) among its
  reconstructed tables.
- This stratum is by definition papers the corpus holds; the study measures the value of the table layer and the
  corpus-vs-web gap *on table facts*, not coverage (coverage was measured in FACTS).

## 2. Items — keyed from the rendered table, checked against the PDF text
A headless Claude process with only Read and Write sees the **rendered image of the table** (crop of the PDF page at
200 dpi), its caption and the paper's abstract — not our reconstruction — and writes one question whose answer is a
single cell value (with unit and the row/column it belongs to), paraphrased, self-contained (device, quantity,
conditions, "published 2025–2026 in Nuclear Fusion or PPCF"), without title, author or DOI.
Automatic validation (as FACTS v1): the key number must occur in the PDF text **inside the table's region** of the page
(independent of our cell reconstruction), the unit must parse and the key must grade itself correct; leak and copy checks; one regeneration,
then drop. Expected ≈ 90 items.

## 3. Arms (same model, effort, isolation and brief as FACTS)
| Arm | Access |
|---|---|
| A | model only |
| B | corpus, **text only** (the FACTS snapshot and commands; no table commands) |
| E | corpus **+ table layer** (same snapshot + `tables`/`table` on a frozen copy of the table store) |
| C | web search |
| F | corpus + table layer + web |
B and E differ only in the two table commands and one sentence describing them.

## 4. Grading
The FACTS code grader, unchanged: unit-aware ±2 % on the `value` field (first reading), range-width cap, no unit
wrong unless dimensionless; abstention wrong. The FACTS grader's known false-negative pattern (trailing words read as
units) is fixed before this freeze with tests, and a 10 % random hand check plus a review of every wrong answer that
contains the key number are reported.

## 5. Pre-registered analysis
One item per paper (no clustering needed). **Co-primary, exact two-sided McNemar, Holm over the two:**
1. **E vs B** — value of the table layer;
2. **E vs C** — corpus with tables vs web.
Verdict for each: better/worse if Holm-significant with |difference| ≥ 10 points, else inconclusive (not
equivalence). Secondary: F vs C, E vs A, B vs C, F vs E, abstention rates, per-arm void counts, and how often E and F
used the table commands. **Pre-registered sensitivities** (descriptive, same tests): batches without voids; items
where the web arm actually fetched the paper (a WebFetch of a URL containing the DOI that did not return a bot
challenge or error) vs items where it did not; items whose key number occurs in the text corpus's passages of that
paper vs not; question–caption/header word overlap ≥ 0.5 vs < 0.5.
Disclosed hint: the writer often names the journal and year ("Nuclear Fusion (2025)"); it is given to every arm and
helps the corpus arms' year filter and the web arm's queries alike.

## 6. Controls
Table extraction quality reported first (coverage vs captions; visual check). Adversarial review of this protocol and
code; a pilot on 3 practice papers outside the sample; freeze (sample, items, prompts, code, snapshot and table-store
hashes) committed before any arm runs; everything reported.

## 7. Review 1 (adversarial, before the freeze) and v1

No critical finding; 4 high, 7 medium, all addressed before any test item was answered:
- **E vs B reworded** (the text corpus already holds the table text; E vs B measures structured access) and the
  key-in-text-corpus covariate added.
- **Web access sensitivity** (items where the web arm reached the paper) pre-registered, so a corpus win cannot be a
  bot-wall artefact unnoticed.
- **Table-use counter fixed** (its regex could never match the wrapper path) with a test.
- **Grader:** a divisor that cannot be read ("5 MW / m^2", "100 kA per m") made the old copy credit "5 MW"; now such a
  reading never matches. Units with parentheses ("W/(m K)"), the `c` prefix ("per cm^3") and "q_{95} = 3.2" fixed;
  all with tests (21 in `tests/test_tabfacts.py`).
- **Validation clipped to the table region** (small numbers elsewhere on the page no longer validate a key).
- **Void-excluded sensitivity and per-arm void counts**; usage-limit failures detected by status or message text and
  never counted as attempts.
- **Caption-overlap covariate**; the year hint disclosed; claims restricted to the layer's coverage slice; the
  undefined "defect" breakdown dropped.
- **Freeze** hashes the whole `src/fusion_corpus` tree, the Python and pint versions, both wrappers, the table store,
  writer outputs, rejected list and prompts, and asserts every sampled paper is in the text snapshot and the table
  store; `score` records the hash of `grades.json`. The sample was drawn from the live catalog of 2026-10-04 (the
  text snapshot is the 2026-10-03 full-corpus snapshot; coverage asserted).
- **Hand check of question→cell mapping:** 25 random items checked against their table images before the freeze
  (reported below).
  Result: **25/25 correct** — each question identifies a unique cell and the key equals the printed value (two
  small tables, t041 and t043, were checked on full-resolution crops).

## 8. Item set (final, before the freeze)
Writer over 100 sample papers (100/100 runs ok), validation, one regeneration for 40 papers, validation again:
**69 items from 69 papers** (31 dropped and listed: units that are not units, values not printed as such in the table
region — e.g. derived by the writer or written with thousands spaces —, copies of ≥ 7 words, title words, no
suitable cell). 14 batches of ≤ 5. Fewer than the ≈ 90 expected: the analysis is unchanged (exact McNemar on 69
paired items); power is lower and is reported.
Pilot (3 practice papers, 2 items; 5 arms, all audits passed): tool arms 2/2, model alone 0/2.

## 9. Post-freeze amendment (before grading)
All 5 arms ran under a verified freeze (11:33). Grading then refused: `FREEZE violated: ['src_tree']`. The only
difference was `src/fusion_corpus/map_export.py` (an untracked dashboard citation-map exporter, edited at 11:42 by
parallel dashboard work); no study module imports it. Verified that this single file hash was the only change in the
whole freeze payload, then the freeze was updated for that file only. No answer, item, prompt or grader code changed.
