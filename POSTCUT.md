# Facts the model cannot have seen: papers published after its training cutoff (pre-registration v1, 2026-10-04)

**Question.** Does the corpus help an assistant answer exact factual questions about papers published **after the
answering model's training cutoff** (June 2026), and how does that compare with web search and with the model alone?
This is the corpus's core use case, a body of literature newer than any model. The question is asked of the *corpus
as built*: the sample includes papers whose full text the corpus does not hold. The corpus's coverage of the newest
papers is therefore part of the result, not excluded by design. This answers the main limitation of `eval/FACTS.md`
and `eval/TABLE_FACTS.md`, whose post-cutoff slices held only 23 and 14 items.

The harness, rules, writers and validation are those of FACTS v1 and TABLE_FACTS v1, whose reviews and lessons carry
over. The differences are listed in §6.

## 1. Sample
- **Frame:** research articles of *Nuclear Fusion* and *PPCF* published 2026-06-01 to 2026-10-01 (the newest record
  in the catalog). Each needs an abstract longer than 200 characters. The papers of the FACTS and TABLE_FACTS samples
  (including practice) are excluded.
- The frame holds **475 papers, 348 (73 %) with full text** in the corpus. The rest are papers the corpus knows only
  by metadata and abstract.
- **100 papers**, seeded random draw (seed 20261101), plus 3 practice papers.
  - 60 of the 100 also get a lookup item (seed 20261102).
  - Each full-text paper with a reconstructed table gets one random table (seed 20261103).
- Drawn: 73 full text, 36 with a table.

## 2. Items
Two blind writers per paper. Each is a headless Claude process with only Read and Write, so neither can search
anything.
- **Facts writer** (FACTS prompt; the only change is "published in 2026"). It sees the title, abstract and, for
  full-text papers, the body text. It writes:
  - **ABS**: an exact quantitative result stated in the abstract;
  - **BODY**: a result stated in the body but not in the abstract (full-text papers only);
  - **LOOKUP**: "which paper reported …, give its DOI" (60 papers).
- **Table writer** (TABLE_FACTS prompt, same change), for papers with a table. It sees the rendered image of the
  table and writes **TABLE**: a single cell value with its row and column.
- **Validation** reuses the FACTS and TABLE_FACTS checks, imported unchanged:
  - the verbatim quote is in the abstract or body as required, or the table key is in the PDF text of the table
    region;
  - the unit parses and the key grades itself correct;
  - no title, author or DOI leak;
  - no copy of ≥ 7 consecutive words.
- A writer whose item is rejected is regenerated once; an item still failing is dropped and listed.
- Expected: ≈ 95 ABS, ≈ 70 BODY, ≈ 33 TABLE, ≈ 58 LOOKUP.
- qids are shuffled (seed 20261104). Batches hold ≤ 5 questions, never two from one paper.

## 3. Arms (same model, effort, isolation, brief and audit as TABLE_FACTS)
| Arm | Access |
|---|---|
| A | model only |
| B | corpus: the frozen full-corpus text snapshot (2026-10-03) **plus** the frozen table store; commands search, passage, works, paper, count, tables, table |
| C | web search and web fetch |
| D | corpus (as B) + web |

Prompts differ only in the tools paragraph, with an automatic parity check. Model: `claude-opus-5-5`, effort
`xhigh`. Every run is headless and isolated, and a use-based transcript audit voids any run that leaves its arm's
tools. Each batch gets 2 attempts. A usage-limit failure is infrastructure, not an attempt.

## 4. Grading
Code grader: unit-aware ±2 % on the `value` field (first reading), range-width cap, a missing unit is wrong unless the
quantity is dimensionless, abstention is wrong. LOOKUP is correct if the paper's DOI is cited.
- **The grader is the TABLE_FACTS grader plus the two fixes for false negatives that its review found:**
  - composition percentages ("14.91 vol%", "at.%", "wt%") are read as %;
  - Unicode superscript exponents ("5 × 10⁻⁶") are read as written.
  - Nothing multiplies a percentage ("2 at.% W" is 2 %).
  - Tests are in `tests/test_postcut.py`.
- **Grader checks reported:**
  - a 10 % random hand check of the A–D grades;
  - a review of every wrong answer that is not an abstention.
- These checks are descriptive. The primary result uses the frozen grade.

## 5. Pre-registered analysis
Fact items are ABS + BODY + TABLE. Items of one paper are not independent, so tests are paper-clustered: a two-sided
sign-flip permutation on per-paper differences (exact up to 16 non-zero papers, otherwise 200,000 permutations, seed
as FACTS).

**Co-primary (Holm over the two):**
1. **B vs C**, corpus vs web, on fact items.
2. **D vs C**, corpus + web vs web, on fact items.

**Verdict:** better or worse if Holm-significant with |difference| ≥ 10 points; otherwise inconclusive (not
evidence of equivalence).

**Secondary, descriptive with the same tests:**
- each kind (ABS, BODY, TABLE, LOOKUP);
- B vs A, C vs A, D vs B;
- full-text papers vs abstract-only papers. The corpus can only help on the abstract and the metadata of the latter,
  so this split shows where coverage limits it;
- items where the web arm fetched the paper (a WebFetch of a URL containing the DOI that did not return a bot
  challenge or error) vs not;
- by publication month;
- batches without voids;
- abstention rates and per-arm void counts.

## 6. Differences from FACTS and TABLE_FACTS
- Frame restricted to post-cutoff papers.
- One study combines the abstract, body, lookup and table items.
- The corpus arm has the table layer: it is the product as built, and TABLE_FACTS showed it changes nothing on
  accuracy.
- Grader false-negative fixes are applied before the freeze.
- **The freeze hashes exactly the code modules the study loads**, rather than the whole source tree. In TABLE_FACTS,
  the whole-tree hash was tripped by unrelated dashboard work during the run.
- The coverage check asserts that full text and tables in the frozen snapshot match the sample's record, both ways.

## 7. Controls
- A pilot on 3 practice papers outside the sample.
- An adversarial review of this protocol and the code, with fixes before the freeze.
- A hand check of the item→key mapping (25 items) before the freeze.
- The freeze (sample, items, writer outputs, prompts, protocol, loaded code, wrapper, snapshots, versions) is
  committed before any arm runs.
- Everything is reported, including the null results.

## 8. Review 1 (adversarial, before the freeze) and v1

There was 1 critical finding and 5 high. All of them were addressed before any test item was answered, and before
the date data below were looked at.

- **C1, the "post-cutoff" frame was not clean.** The catalog's `published` date is often the issue date (90 % of
  PPCF rows fall on day 01). IOP posts the accepted manuscript online when the DOI is registered, weeks earlier, and
  arXiv preprints may be older still.
  - **v1 eligibility.** A paper is post-cutoff only if its **earliest public date** is on or after 2026-06-01. The
    earliest public date is the minimum of five dates:
    - Crossref DOI registration (`created`);
    - online date;
    - print date;
    - catalog date;
    - first version of a matching arXiv preprint (title similarity ≥ 0.90).
  - These dates are fetched from the public Crossref and arXiv APIs for every frame paper
    (`src/fusion_corpus/postcut/dates.py`). The raw responses are cached, and the table is hashed in the freeze.
  - **Sample correction.** Drawn papers that fail the rule are dropped and listed. Replacements come from further
    seeded draws (seed 20261105) among eligible frame papers not yet drawn, until 100 papers are reached. This is
    still a uniform random sample of the eligible frame. Writer outputs of dropped papers stay on disk and give no
    items.
  - **Sensitivities:**
    - papers with no arXiv preprint found at all;
    - by publication month.
  - **Contamination check:** model-alone accuracy (A) is reported against its accuracy on the pre-cutoff FACTS
    slice.
  - **Limit:** a preprint with a different title is not detected.
- **H2, freeze holes.** v1 hashes the following, and the wrapper Python process is checked separately:
  - an explicit file set: every fusion_corpus module loaded by the study, the study file itself (it runs as
    `__main__`), and every module the wrapper's own Python process loads (found by importing them in a fresh
    process);
  - the snapshot file itself;
  - the date table.

  The freeze also asserts that `.env` forces no `FUSION_*` path, since `.env` overrides the environment. A new study
  wrapper (`data/postcut/bin/corpus`) clears corpus environment overrides and logs Python errors to
  `data/postcut/wrapper_errors.log` instead of discarding them. Wrapper error replies in B/D transcripts are counted
  and reported. `grade` writes a manifest of the graded answer files and the grades hash.
- **H3, the real run was untested.** The pilot did complete all four arms (12/12 runs, all audits passed). The
  reviewer read it mid-run. `score` now runs on practice data and was exercised end to end. Usage-limit detection
  now treats all of the following as infrastructure, never as attempts, and logs every one:
  - HTTP 429/529;
  - limit or overload wording;
  - a crash with no result line.
- **H4, the design leans toward the corpus.**
  - This is disclosed and measured, not removed. The study asks about the corpus *as built*, and abstract-only
    papers are included.
  - New covariate: the key number occurs in the PDF's own text layer, independently of the corpus's cleaned text. A
    sensitivity is restricted to those items.
  - New strata: question copy-run ≤ 4 vs 5–6 words (the writer prompt says ≤ 4, the validator rejects ≥ 7), and
    title overlap < 0.3 vs ≥ 0.3.
- **M1, void asymmetry.** Per-arm void counts are reported. A verdict is flagged if any arm voids more than 10 % of
  batches, with the void-excluded group shown.
- **M2, web access sensitivity redefined.** The web arm counts as having had the answer only if it fetched the
  paper (URL with the DOI or the arXiv id, from the graded attempt) and the fetched text contained the key number.
- **M3, validation holes closed.**
  - A missing table-writer output is listed and regenerated.
  - Unrequested lookups are ignored.
  - Writer fields can no longer overwrite study fields.
  - The self-grade still uses the FACTS grader, as in FACTS. It is stricter, so it can only drop items.

## 9. Eligibility, item set and hand check (before the freeze)
- **Earliest public dates** were fetched for all 475 frame papers (Crossref 475/475, arXiv search 475/475).
  - **105 (22 %) had been public before the cutoff:** an accepted manuscript or an arXiv preprint dated before
    2026-06-01. 58 frame papers have a matching arXiv preprint.
  - **Eligible frame: 370 papers.**
- **Sample:**
  - 24 of the 100 drawn papers were public before the cutoff. They are dropped and listed in
    `private/dropped_pre_cutoff.json`.
  - 24 seeded replacements were added (`r001`–`r024`).
  - Final sample: 100 papers, 69 with full text, 36 with a table.
- **Writers:** 169 runs, all passed the audit. 83 writers with a rejected or missing item were regenerated once.
  Final: **185 items**:

  | Kind | Items |
  |---|---|
  | ABS | 44 |
  | BODY | 56 |
  | TABLE | 24 |
  | LOOKUP | 61 |

  That gives **124 fact items**, from fewer abstract items than expected. 81 items are dropped and listed; the
  commonest reason is an abstract with no paper-specific number, where the writer correctly wrote none. The batches
  number 37. LOOKUP has 61 items rather than 60 because replacement papers get a lookup with probability 0.6.
- **Hand check:** 25 random items (seed 20261106) were checked against their quote, or for tables against the
  rendered table image. **25/25 are correct:** each question identifies the value and the key matches the source.
- **Power:** 124 fact items in about 100 papers (paper-clustered). The earlier studies detected +19 and +42 points
  with similar sizes.
