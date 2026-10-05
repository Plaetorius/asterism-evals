# When the answer is in the paper: pre-registered, audited tests of an open-access literature corpus against web search for an LLM research assistant in fusion science

*Article draft v2 (2026-10-04). It is the concise version of the technical report `eval/paper/PAPER.md`, which holds
every design detail, threat and incident. Sections marked ⟨PENDING⟩ are filled from the second-model replication
when it finishes.*

Authors: <author name(s): to be confirmed by the owner> (with Claude Opus 5.5 as research engineer)

---

## Abstract

Retrieval-augmented assistants are usually tested against the same model without retrieval. That comparison is not
what a researcher actually faces: today's assistants already search the web. We built an open-access literature
corpus for fusion science and measured it against web search in a series of pre-registered studies with a common,
audited execution harness:
- **The corpus:** metadata of 24,101 *Nuclear Fusion* and *Plasma Physics and Controlled Fusion* papers, the full
  text of the 3,494 that are openly licensed, and 3,893 tables reconstructed from them.
- **The tool:** a small BM25 command-line search.
- **The arms:** the same model (Claude Opus 5.5) answered with no tools, with the corpus, with web search and fetch,
  or with both.
- **The harness:** isolated headless agents, native tool whitelists plus a scripted use-based transcript audit,
  hash freezes checked by every later stage, pilots, and an adversarial review before every freeze.

The results split by the kind of question:
- **Exact facts from random recent papers** (188 items from 97 papers): the corpus answered 93 % against web
  search's 74 % (+19 points, paper-clustered p = 1 × 10⁻⁵). The gain was entirely in facts from the paper body
  (+29) and in finding a described paper (+15); facts stated in abstracts were equal.
- **Values printed in tables** (69 items): 99–100 % against 57 % (+42 points, p = 3.7 × 10⁻⁹). The gap held when the
  web agent had fetched the paper (16/16 against 8/16).
- **Papers made public only on or after the model's training cutoff** (185 items from 100 papers, dated by their
  earliest public appearance): **98 % against 48 % (+49 points, p ≤ 5 × 10⁻⁶**, the resolution of the permutation
  test). Body facts +66, table values +75. On the 17 items where the web agent's fetched text contained the value, it
  matched the corpus. On the other 107 it scored 40 % against the corpus's 97 %, abstaining on 58.
- **A second model family** (Gemini 3.1 Pro with Google Search grounding, a conceptual replication): pre-registered
  and running; ⟨PENDING⟩.

On open researcher-style questions (60 blind questions, two blind judges from two vendors) and on evidence
gathering for meta-analysis (6 tasks), corpus and web were **not distinguishable**: 47 % against 52 %, p = 0.66; and
63 against 84 verified data points, p = 0.31. The model alone answered almost none of the exact facts (2–3 %).

The harness found and removed biases that would have decided outcomes on their own:
- an audit rule that voided corpus queries containing `Q>1`;
- a lost reasoning-effort setting that cut tool use threefold;
- arm-identifying fields shown to judges;
- grader rules that credited "5 MW" for "5 MW/m²";
- a "post-cutoff" frame in which 22 % of papers had been public earlier.

We release the code, protocols, prompts, transcripts and the open-access corpus (CC BY).

---

## 1. Introduction

A researcher choosing an assistant for the literature does not choose between a curated corpus and nothing. They
choose between a corpus and the web search already built into their assistant. Builders of domain corpora, which
includes us, have an incentive to show that a corpus helps. The easy way to show it is against the bare model, and
the easy way to bias it is to write questions from the corpus. We set out to test the useful comparison, the corpus
against web search, under rules that make a favourable result hard to manufacture:
- every study is pre-registered;
- every protocol is reviewed adversarially before its freeze;
- every input and line of study code is hashed;
- every agent run is isolated and audited;
- every result is reported, including the nulls.

The contributions are:
1. **An execution harness for agentic evaluation (§3)**, with the incidents it caught. We think this transfers
   beyond this domain.
2. **A map of where a corpus beats web search and where it does not (§5).**
   - It does when the answer is a specific number inside a recent paper's body or tables. That is the case where
     Claude's web tools, on this publisher in October 2026, did not reliably reach or read the full text.
   - It does not show a difference on open questions or broad evidence gathering, where the web reaches more
     sources.
3. **The corpus's core use case (§5.3):** knowledge newer than the model. On papers public only after the model's
   cutoff, dated by earliest appearance, the corpus answered 98 % of exact facts against web search's 48 %. A
   replication with a second model family is pre-registered (§5.4); its results are pending.
4. **An open-access fusion corpus** (full text, passages, tables and citations, CC BY) and a reproducible benchmark
   (§7).

---

## 2. System under test

**Corpus.** Metadata for all 24,101 research works of *Nuclear Fusion* (since 1960) and *PPCF* come from Crossref
and OpenAlex. The full text of 3,494 papers, every one CC BY or CC BY-NC-SA, was downloaded politely from the
publisher with stop-on-challenge. Bot challenges were passed by hand and never automated. Text is extracted
locally with layout cleaning, the publisher cover pages are removed, and the text is chunked into 165,951
passages. Coverage is skewed to the 2020s: there is no openly licensed full text before 2010.

The Q&A study (§5.5) used an earlier snapshot: 1,815 full texts and five commands, before the download finished and
before the table layer existed. All later studies used the full 3,494-paper snapshot.

**Tables.** Journal tables here have no vertical rules, and off-the-shelf extraction failed (0/40). We reconstruct
tables from the horizontal rules under each "Table N." caption, with rows from word positions, columns from shared
white channels, and sub- and superscripts kept from character-level font data. This recovered 3,893 of 4,429
captioned tables (88 %).

**Retrieval.** SQLite FTS5 with BM25 over symbol-normalised text. The command-line tool has seven commands: search,
passage, works, paper, count, tables and table. The assistant composes the queries itself. The corpus never
generates text; the calling model does.

---

## 3. The execution harness

An agentic arm's result depends on its permissions, environment, reasoning settings and the rule that decides
whether a run counts. Any of these can differ between arms by accident. The harness is built to make such
differences impossible or visible.

- **Isolation.** Every answer batch is a separate headless `claude -p` process with project-only settings, a
  pinned model and effort, an empty MCP configuration and a private temporary working directory. The tools are
  whitelisted natively per arm.
- **Use-based audit.** A script reads every transcript and checks what each call did. It parses shell commands into
  tokens, so a quoted `Q>1` is not a redirect, and it allows only the arm's wrapper and commands, writes only to the
  arm's own output file, and fetches only public URLs. A violation voids the batch. Each batch gets two attempts and
  the first passing attempt counts. Usage-limit failures are infrastructure, not attempts.
- **Freeze.** Items, writer outputs, prompts, protocol, snapshots and the study code are hashed into a committed
  file before any arm runs, and every later stage verifies it. After the table study was tripped by an unrelated
  edit, the hash covers exactly the modules the study loads, including those of the wrapper's own process.
  Post-freeze changes are documented amendments, made after verifying that only the intended hash changed.
- **Blind item writing.** Questions are written by a tool-less process from the paper text, or from the rendered
  table image, never from our reconstruction. They are validated automatically: quote or key present in the
  source, unit parseable, no title, author or DOI leak, no long copied phrase. A failing item gets one
  regeneration, then is dropped and listed.
- **Adversarial review before every freeze**, plus a pilot on practice items. These reviews found, among other
  things:
  - judges seeing arm-identifying fields;
  - a reasoning-effort setting silently lost under isolation;
  - grader false positives and negatives;
  - "web reached the paper" heuristics that counted bot-challenge pages;
  - a post-cutoff frame contaminated by accepted manuscripts and preprints.

The technical report (§5–6 and 12) lists every incident and lesson.

---

## 4. Studies

| Study | Question | Items | Arms | Primary | Grading |
|---|---|---|---|---|---|
| Q&A head-to-head | open researcher questions | 60 (+80 secondary) | A, B, C, D | B vs C, McNemar | 2 blind judges, 2 vendors, dual keys |
| Meta-analysis | gather verified data points | 6 tasks | A, B, C, D | B vs C, D vs C, Wilcoxon, Holm | verifier agents (κ reported) |
| Recent facts | exact values from random 2025–26 papers | 188 (97 papers) | A, B, C, D | B vs C, D vs C, paper-clustered permutation, Holm | code, unit-aware ±2 % |
| Table facts | values printed in tables | 69 (69 papers) | A, B, E (+tables), C, F | E vs B, E vs C, McNemar, Holm | code |
| Post-cutoff | facts from papers public only on or after 2026-06-01 | 185 (100 papers) | A, B, C, D | B vs C, D vs C, clustered, Holm | code (+2 review fixes) |
| Replication | recent and table facts, Gemini 3.1 Pro + Google grounding | 188 + 69 | as originals | as originals | frozen graders |

Arms: A is the model only; B is the corpus; C is web search and fetch; D is corpus + web. In the table study, E and F
add the table layer. Verdict rule everywhere: "better" or "worse" requires Holm significance and a difference of at
least 10 points; anything else is inconclusive and is not evidence of equivalence.

---

## 5. Results

### 5.1 Exact facts from recent papers
- **Design.** 100 random NF/PPCF research papers from 2025–26, drawn from the catalog whether or not the corpus
  holds their full text. 97 of them yielded items: 129 facts from the abstract or body (92 papers) plus 59 "which
  paper reported…" lookups.
- **Result.** Corpus 93 % against web 74 % (+19.4 points, p = 1 × 10⁻⁵). Corpus + web scored 94 %, the model alone
  2 %.
  - Abstract facts were equal (93/91/96 %).
  - Body facts: 93 % against 64 % (+28.6). The web abstained 28 times.
  - Lookup: 98 % against 83 % (+15.3).
- **Mechanism.** In the transcripts, the web agent fetched the publisher site 171 times. 27 % of those fetches hit a
  bot challenge, and the agent fell back on preprints and abstracts.
- **Grader check.** A hand review found 19 grader false negatives, mostly against the corpus arms. The frozen
  result therefore understates the corpus lead (≈ +24 points when hand-corrected).

### 5.2 Values printed in tables
- **Result.** Corpus 100 % (text only) and 99 % (with tables), against web 57 % and the model alone 3 %.
  - Corpus + tables vs web: +42.0 points, 29 discordant pairs against 0, p = 3.7 × 10⁻⁹.
  - Corpus + tables vs text-only corpus: inconclusive at the ceiling. The text passages already contain each
    table's text, so the structured layer added nothing measurable.
- **Web sensitivity.** When the web agent fetched the paper itself, it still found the value in only 8 of 16 cases;
  the corpus found all 16.
- **Grader check.** Of 6 wrong non-abstaining answers, 4 were grader false negatives ("vol%", Unicode exponents)
  spread across arms. Verdicts are unchanged.

### 5.3 Papers made public only on or after the model's training cutoff
- **Design.** The frame is NF/PPCF papers whose earliest public date is on or after 2026-06-01. The earliest public
  date is the minimum of Crossref DOI registration, online, print and catalog dates, and the first version of a
  matching arXiv preprint.
  - 105 of 475 papers (22 %) dated after the cutoff by issue had been public earlier, as accepted manuscripts or
    preprints, and were excluded.
  - 100 random eligible papers, coverage-inclusive: 31 have no full text in the corpus. They contribute only 12 of
    the 124 fact items, because body and table items need full text.
  - 185 items: 44 abstract, 56 body, 24 table, 61 lookup. 81 drafted items failed validation and were dropped and
    listed. A hand check of 25 found all correct.
- **Result.** Corpus 121/124 (98 %), web 60/124 (48 %, 58 abstentions), corpus + web 118/124 (95 %), model alone
  3/124 (2 %).
  - B vs C: +49.2 points (61 vs 0 discordant items), p ≤ 5 × 10⁻⁶.
  - D vs C: +46.8, p ≤ 5 × 10⁻⁶.
  - By kind: abstract 44/38 (+14), body 53/16 (+66), table 24/6 (+75), lookup 61/47 (+23).
  - Adding the web to the corpus did not help (D vs B −2.4, p = 0.38).
- **Sensitivities:**
  - Papers with no arXiv preprint at all: +50.4.
  - Keys also present in the PDF's own text layer, independent of our extraction: +48.4.
  - Every stratum of question copy length and title overlap: +44 to +61.
  - Every month with more than one item: +44 to +63.
  - Papers whose full text the corpus does not hold (12 items): 12/12 against 10/12.
  - Items where the web agent fetched the paper and the value was in the fetched text (17): 17/17 for both. This
    subset is selected on the web arm's own success, so it is descriptive, not a causal test.
- **Contamination check.** The model alone scored 2.4 % here and 2.8 % (3/106) on the pre-cutoff facts of §5.1. It
  abstained on about 97 % of items, so at this floor the check cannot detect memorisation. It shows only that the
  bare model does not offer these values.
- **Grader check.** 36 of 37 random grades agreed with a human reading. 7 grader false negatives were found (B 2,
  C 1, D 4; e.g. "R² = 0.9998" read as 2). Hand-corrected: 123 against 61. Verdicts unchanged.

### 5.4 A second model family ⟨PENDING⟩
- **Design.** The frozen recent-facts and table-facts items, answered by Gemini 3.1 Pro with function-calling
  corpus tools (copies of the frozen wrappers) and Google Search grounding.
- **Scope.** This is a conceptual replication, not a like-for-like repeat. Google grounding returns snippets and
  made no page fetches in the pilot (weaker on page content), and its multi-query search steps are capped per step
  (looser on search volume). The two differences bias in opposite directions and are reported.
- ⟨Results⟩

### 5.5 Where the corpus does not win
- **Open questions** (60 blind researcher-style questions, strict consensus of two blind judges): corpus 47 %
  against web 52 % (p = 0.66, inconclusive). Every tool arm beat the model alone (27 %). Corpus + web scored highest
  (62 %) and contradicted the key least (5/60), but its lead over web was not significant (p = 0.26). It did beat the
  corpus alone (+15 points, p = 0.023, secondary).
- **Evidence gathering** (6 meta-analysis tasks): 63 against 84 verified points (p = 0.31, inconclusive).
  - The web reached more sources, especially outside the two journals.
  - The corpus's points were more often verifiable: median precision 0.97 against 0.77.
  - Verifier agreement, κ 0.55, was below our pre-registered 0.6 flag.
- **Stability:** rerunning the 112 Q&A batches, web answers flipped correctness on 22 % of questions against 12 % for
  the corpus arms (exploratory).

---

## 6. What explains the pattern

The corpus wins exactly where the answer is a specific statement inside a paper's full text and the paper is in
the corpus: body facts, table values, and identifying a paper from a described result. It shows no reliable
difference, or a small one, where the answer is in an abstract, which is indexed everywhere: equal in the recent-facts
study, +14 points (p = 0.03) post-cutoff. It also shows no reliable difference where breadth matters more than depth:
open questions and multi-journal evidence gathering.

Two mechanisms are visible in the transcripts:
- **Access.** Publisher pages often present bot challenges to automated fetchers, and preprints may differ from the
  published version.
- **Reading.** Even with the page fetched, values in long papers and tables are often not in what the fetch tool
  returns.

The model alone almost never knows these values, even for papers published before its cutoff. Exact numbers from
recent specialised papers are rarely offered: the model alone scored 2.8 % before its cutoff and 2.4 % after it.
That floor is too low to detect memorisation, so it is not evidence about memorisation either way. The transcripts
support the access mechanism descriptively:
- On the 17 post-cutoff items where the fetched text held the value, the web agent matched the corpus.
- On the other 107 it scored 40 % against 97 %.

The 17-item subset is selected on the web arm's success, so this is not a causal test. In the table study, fetching
the paper gave the web arm 8 of 16 values, not 16 of 16.

---

## 7. Limitations
- **One answering model family**, except for the replication (§5.4).
- **Model-written questions and keys; one model family in the loop.** Claude writes the questions and keys, from
  text the corpus indexes, and Claude answers them. In the Q&A study Claude is also one of the two judges.
  - The validation is source-anchored.
  - Item hand checks were 25/25 in the table and post-cutoff studies; FACTS had grade checks only (74/75).
  - The post-cutoff study adds a PDF-text check of each key that does not depend on our extraction.
  - A risk remains that questions are phrased in ways that suit lexical search. We report copy-length and
    title-overlap strata; 93 of the 124 post-cutoff fact items have title overlap ≥ 0.3.
- **Sampling frame equals the corpus catalog.** For lookup items, the right paper is always retrievable from the
  corpus's metadata.
- **Grader review is asymmetric.** We looked for false negatives among wrong, non-abstaining answers. The web arm
  abstains more often, so it has fewer answers that a review could correct.
- **Study order and multiplicity.** The fact studies were designed after the Q&A null and its corpus-favouring
  secondary sets. Holm is applied within each study. Across all 9 primary tests of the programme, the three fact-study
  wins (p ≤ 1 × 10⁻⁵) survive a Bonferroni correction (threshold 0.0056).
- **Corpus coverage.** The fact studies sample papers from the two journals the corpus covers, and the table study
  samples only papers with extractable tables. The results therefore say what the corpus does on its own
  literature. The recent-facts and post-cutoff studies include papers without full text, so coverage is part of
  those numbers.
- **"Web" means one tool set.** The web arm used the assistant's built-in search and fetch (Claude), or Google
  grounding (Gemini). A human with library access, or a scholarly search engine, may do better.
- **Small samples for some strata, and grader noise.** Hand reviews found grader false negatives in both directions,
  and we report hand-corrected figures alongside the frozen ones.

---

## 8. Data and code availability
- **Code, protocols, prompts, freeze files, results and the benchmark bundle:** ⟨repository, after the owner's
  decision⟩. The bundle holds items, answers, grades and audited transcripts, with IP addresses redacted.
- **Open-access corpus release** (CC BY; the 3 CC BY-NC-SA papers are separated):
  - metadata of 24,101 works, with abstracts only where openly licensed;
  - the full text of 3,494 papers, 165,951 passages (after removing 5 stamped cover pages) and 3,893 tables;
  - citations;
  - a ready-to-query SQLite database for the same command-line tool.

  No PDFs are distributed. A privacy scan of every released string (8.2 M fields) found no download stamps, owner
  identifiers or local paths after removing the cover pages of 5 papers that carried a second, bound-in cover sheet.

---

## Appendix A. Lessons for evaluation harnesses
Condensed from the technical report §12:
1. Compare against the alternative users have (web search), not only the bare model.
2. Assume every arm-specific mechanism is biased until tested.
3. Enforce permissions twice, natively and by audit, and test the first line.
4. Audit uses, not words.
5. Voids are draws: fix the re-run policy in advance.
6. Pin every setting isolation can drop.
7. Give judges an allow-list of fields.
8. Freeze code by what actually runs.
9. Run the whole pipeline on practice items.
10. Date "new" data by its earliest public appearance, not its issue date.
