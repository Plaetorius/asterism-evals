# Corpus, web, or both? A pre-registered, audited head-to-head of literature tools for an LLM research assistant in fusion science

*Draft with final results. The Q&A results (§7) are filled from `data/h2h/results.json` and `eval/h2h/RESULTS.md`
(commit `c21209a`); §8–10 report the follow-up studies. The recent-facts study (§7.10) is still running and its
results are now filled from `eval/facts/RESULTS.md`.*

Authors: <author name(s): to be confirmed by the owner> (with Claude Opus 5.5 as research engineer) · Date: 2026-10-03 · Repository commit of the freeze: `b31ea9a` (2026-10-02)

---

## Abstract

Retrieval-augmented generation (RAG) is often presented as the way to make a large language model (LLM) a
trustworthy research assistant. The claim is usually tested against the same model without retrieval, rarely
against what a researcher would actually use instead: the same model with web search. We built a home-made
literature corpus for fusion science (metadata for 24,101 *Nuclear Fusion* and *Plasma Physics and Controlled
Fusion* works; full text of 1,815 openly licensed papers in the frozen snapshot used here), exposed it through a
small BM25 command-line tool, and compared four arms of the same model (Claude Opus 5.5): no tools (A), corpus
(B), web search and fetch (C), and corpus plus web (D). The primary endpoint, fixed before any arm ran, is
strict-consensus accuracy of B versus C on 60 researcher-style questions written blind, tested with an exact
McNemar test; four secondary question sets (140 questions in total) are reported in full, including two that
favour the corpus by construction. Answers were cut to a common length, rewritten into neutral claims by a
different-vendor model and graded by two blind judges from two vendors against two independently built answer
keys. The main contribution of this paper is the execution harness: every agent ran as an isolated headless
process with a native tool whitelist, every transcript was audited by script against a use-based policy, and
every input, prompt and line of harness code was hashed into a committed freeze file that later stages check
before they run. A full dry run and two adversarial reviews found biases that would have decided the outcome on
their own — among them an audit rule that voided corpus queries containing a quoted `Q>1`, arm-identifying leak
flags passed to the judges, and a loss of the reasoning-effort setting that cut tool use to about a third. All
112 arm batches passed the audit (0 voids; 560 answers, none missing). The primary result is **inconclusive**:
B 28/60 (47 %) against C 31/60 (52 %), B − C = −5.0 points (95 % CI −20.0 to +10.0, exact McNemar p = 0.664). This
is not evidence that the corpus is as good as web search. Every tool arm beat the bare model on P (A 27 %; B vs A p = 0.0169,
C vs A p = 0.00149). Corpus plus web (D) scored highest on P (62 %) and contradicted the key least (5/60 answers,
against 12–14 for the other arms), but its lead over web search alone (+10.0 points, 95 % CI −5.0 to +25.0, p =
0.263) is not significant. On two secondary sets written from corpus papers, the corpus won decisively (S2 exact
facts 100 % vs 23 %; known-paper lookup 92 % vs 60 %); these favour the corpus **by construction**. A fair,
pre-registered version on a random sample of 100 recent NF/PPCF papers (188 items, code-graded) **found the corpus
better than web search on exact facts: 93 % vs 74 %, +19.4 points, paper-clustered p = 1 × 10⁻⁵** (corpus + web
94 %, +20.2, p = 5 × 10⁻⁶); the gain comes from facts inside the paper body (+28.6 points) and paper lookup (+15.3),
with no difference on facts stated in abstracts, and the web arm's transcripts show it often could not reach the
full text (27 % of its publisher fetches hit a bot challenge). A second pre-registered study on values printed in tables (69 items from 69 random recent papers) widened the gap: **corpus 99–100 % vs web 57 %, +42 points, p = 3.7 × 10⁻⁹**, also when the web arm reached the paper (+50 on 16 items); a reconstructed table layer (3,893 tables) did not add accuracy, because the text passages already carry the table text (both at ceiling). Two follow-up studies used the same harness. In an evidence-gathering
(meta-analysis) study on 6 tasks, both co-primary comparisons were inconclusive (verified data points: corpus 63,
web 84, corpus + web 75); the corpus arm's points were more often verifiable (median precision 0.97 against 0.77),
but agreement between the two verifiers (κ 0.55) fell below the pre-registered 0.6 flag. Re-running all 112 Q&A
batches with identical prompts, correctness flipped on 22.1 % of questions for the web arm against 12.1 % for the
corpus and corpus + web arms, about twice as often (an exploratory, not pre-registered, comparison). An offline
programme to extract data from the corpus's figures reached precision 1.000 on synthetic plots but is still below
its pre-registered precision bar on real figures, and no figure values are used. We report every arm on every set
and describe the harness in enough detail to be reused.

---

## 1. Introduction

A research assistant built on an LLM can answer from what the model memorised, from documents a retrieval tool
hands it, or from the open web. Builders of domain corpora — and we are one — have an incentive to show that
their corpus helps. The usual demonstration compares the model with retrieval against the model alone. That
comparison answers a question few users ask. A researcher deciding whether to use a curated corpus is not
choosing between the corpus and nothing; they are choosing between the corpus and the web search that is already
built into their assistant. A corpus that beats the bare model may still lose to, tie with, or add nothing to web
search.

Three further problems make such comparisons easy to get wrong.

**Contamination and construction.** Questions written from the corpus favour the corpus; questions written from
textbook knowledge favour the bare model. In our first protocol draft, one of the two primary sets was written
from catalogued abstracts that the corpus tool returns almost verbatim, and the other from textbook topics the
model knows well; an adversarial reviewer predicted that the result was "mostly fixed before anything runs"
(review 1, C1). Recency adds a second axis: facts published after the model's training data favour any arm that
can retrieve them, corpus or web.

**Measurement harnesses for agents are themselves software.** An agentic arm's answer depends on its tool
permissions, environment, reasoning settings and the policy that decides whether a run counts. Each can differ
between arms without anyone intending it, and in this study several did (§6.2).

**Marketing pressure.** The owner of this project stated at the outset that results may be used in marketing
material. That is a reason to fix the design, the analysis and the chart before any arm runs, to report every
arm on every set including those where the corpus loses, and to forbid, in writing, presenting an inconclusive
result as "as good as the web".

This paper describes a head-to-head built under those constraints:

1. a corpus and retrieval tool that are the system under test (§3);
2. a pre-registered benchmark: four arms, five question sets with stated biases, dual answer keys, normalised
   answers, two blind judges from two vendors, a single primary endpoint with a fixed decision rule and a fixed
   chart (§4);
3. an execution harness for agentic arms — isolated headless processes, native and audited tool whitelists,
   hashed freezes, a full dry run, and complete provenance logging (§5), which we consider the main
   contribution;
4. a catalogue of threats to validity, most of them found by the dry run and by two adversarial reviews, with
   how each was handled (§6).

Results of the Q&A head-to-head are in §7 and are reported whatever they are. Three follow-up studies reuse the
harness: evidence gathering for meta-analysis (§8), stability across identical runs (§9) and a programme to extract
data from figures and tables (§10); a fair test of recent exact facts is in progress (§7.10). §11 discusses the
results; the discussion paragraph for the inconclusive case was written before the outcome was known and is the
one kept.

---

## 2. Related work

**Retrieval-augmented generation.** Conditioning generation on retrieved passages was popularised by Lewis et
al. (2020) and builds on dense passage retrieval (Karpukhin et al., 2020) and on lexical ranking functions such
as BM25 (Robertson and Zaragoza, 2009). Surveys describe the many variants since (Gao et al., 2023). Our system
is deliberately simple — lexical BM25 over passages, called by an agent through a shell command — and is the
system under test, not a proposed method.

**Literature assistants for science.** Several systems pair LLMs with scientific literature retrieval, for
example PaperQA (Lála et al., 2023) and OpenScholar (Asai et al., 2024). Web-browsing assistants (Nakano et al.,
2021, WebGPT) are the other branch. Evaluations in this line usually compare a system against baselines chosen by
its authors; head-to-head comparisons of a domain corpus against the same model with general web search, under
identical prompts and audited tool use, are less common. `[CITATION NEEDED: a published corpus-vs-web comparison
for scientific QA, if one exists]`

**LLMs as judges.** Using LLMs to grade answers is now standard and has known biases: position bias, verbosity
bias and self-preference (Zheng et al., 2023); models can recognise and favour their own generations
(Panickssery et al., 2024). Our design responds with claim-level normalisation by a different-vendor model, a
hard length cap, independent label shuffles per judge, two judges from two vendors, and strict consensus.

**Hallucinated citations and contamination.** LLMs are known to produce plausible but non-existent references
(Walters and Wilder, 2023; see Ji et al., 2023, for a survey of hallucination). Benchmark contamination —
test items present in training data — is a recognised threat to evaluation validity (Sainz et al., 2023).

**Agent evaluation and pre-registration.** Agent benchmarks are sensitive to harness details and cost, and
should report both (Kapoor et al., 2024). Pre-registration — committing hypotheses and analyses before seeing
data — is established practice in empirical science (Nosek et al., 2018). The statistics we use are standard:
McNemar's test for paired proportions (McNemar, 1947), Wilson score intervals (Wilson, 1927), Cohen's κ (Cohen,
1960) and the percentile bootstrap (Efron and Tibshirani, 1993).

---

## 3. System under test

### 3.1 Corpus

The corpus is a standalone project whose purpose is grounded question answering and literature mapping for
magnetic and inertial confinement fusion (PLAN §1). It has two layers:

- **Metadata layer (M):** 24,101 works from *Nuclear Fusion* (NF) and *Plasma Physics and Controlled Fusion*
  (PPCF), 1960–2026, harvested from Crossref and OpenAlex: title, authors, date, venue, abstract where available,
  references and licence per content version.
- **Full-text layer (F):** papers whose **version of record** carries a Creative Commons licence, fetched from
  the publisher, validated and indexed. The census found 2,549 NF works with a CC licence on the version of
  record, all from 2010 onwards and mostly from the 2020s.

The snapshot used by the arms holds **1,815 full-text papers** (plus the full M layer), opened read-only and pinned
by sha256 (`b6f9939d4ae00b1094b52aa19d3edfac41f24d60b9ac082f4fe338fb70bfc2ab`). Chunk count at freeze:
86,587.

**Licence rule.** A full text is shareable only if a CC licence applies to *the same content version* as the
stored copy (Crossref licences differ between the version of record and the accepted manuscript; NF has 127 works
with a CC licence on the accepted manuscript only), the licence start date has passed, and no conflicting licence
applies (DECISIONS D5). Open-access copies without a reuse licence ("bronze") are excluded. Abstracts carry their
own flag. The rule is unit-tested.

**Acquisition.** Full texts were downloaded from the publisher's website one at a time, with a 3–6 s jitter,
for CC-licensed items only, under an access arrangement the owner reports the publisher agreed to; the
publisher's bulk XML has been requested as the replacement route (D24, D27, D32). The fetch harness classifies
13 failure types (paywall HTML, bot challenge, truncation, wrong document and others), checks a canary file every
50 fetches and quarantines failures. The publisher stamps each downloaded PDF's cover page with the downloader's IP
address and time; this first broke the canary hash and is the reason for the privacy rule in §15.

### 3.2 Text layer

Text is extracted with PyMuPDF in a sandboxed subprocess (timeout, memory cap, no network) and cleaned: invisible
spans and private-use glyphs are dropped, running heads and footers repeated on at least half the pages are
removed, hyphenation is resolved against the corpus vocabulary, the reference section is split off and not
indexed, and sub- and superscripts are marked. Local PDFs are stored as 150 dpi renditions kept only if every
page's extracted text is identical to the original's (D28, D29). Text is chunked into section-aware passages of
about 400 tokens. Quote anchors (copy, text-layer version, character span, quote hash) are re-located whenever a
text layer changes.

### 3.3 Retrieval

Retrieval is SQLite FTS5 with BM25 ranking over **symbol-normalised** text: Unicode NFKC, Greek letters spelled
out, sub/superscript markers fused, so that `τ_E`, `tau_E` and `tauE` match one another (`index/normalize.py`).
A query first runs as an all-terms match. **Before the freeze**, the independent key-writing agents reported that
"long queries return nothing", because FTS5's implicit AND requires every term. Search was therefore changed to
top up an all-terms result that returns fewer than the requested number of hits with an any-term query (stop
words removed), BM25-ranked, each hit labelled `match: all|any` (HEAD2HEAD §12). This is a generic retrieval fix,
not tuned to any test question, and it is part of the system under test. No dense retrieval or reranker is used.

### 3.4 Access for the arms

Agents reach the corpus through a shell wrapper (`data/h2h/bin/corpus`) with five subcommands: `search`
(full-text passages), `passage` (a passage with context), `works` (titles and abstracts of all catalogued works),
`paper` (metadata and abstract) and `count`. The wrapper:

- refuses to run if the snapshot is missing or its sha256 differs from the frozen value (an earlier version would
  have silently created an empty database and returned no hits — review 2, C2);
- opens the database **read-only** (an earlier version wrote a query-log row on every call, which would either
  have failed against a read-only file or changed the snapshot's hash — review 2, C1);
- runs in evaluation mode, which strips local file paths from results (they invited the agent to read PDFs
  directly) and writes a server-side query log outside the snapshot, used as a cross-check against transcripts;
- reports errors as one JSON line without tracebacks or paths.

Paper text in results is labelled as data, not instructions.

---

## 4. Benchmark design

The protocol is `eval/HEAD2HEAD.md`, version 2, with dated sections 10–16 recording every change made before
the freeze and why. It went through two adversarial reviews (`eval/h2h/reviews/`, verbatim): review 1 of v0
("not a fair head-to-head as drafted") and review 2 of v1 ("NOT READY", five critical and eight high issues).
Every must-fix of both reviews was adopted.

### 4.1 Arms

All four arms use the same model (Claude Opus 5.5, `claude-opus-5-5`, effort `xhigh`), the same brief and the
same answer format. Only one block of the brief differs: the tool sentence (Appendix B).

| Arm | Access | Role |
|---|---|---|
| A | own knowledge only | floor ("a normal LLM") |
| B | own knowledge + corpus tool | the product |
| C | own knowledge + web search and web fetch | the realistic alternative |
| D | own knowledge + corpus + web | the corpus's added value on top of the web |

Every arm may use its own knowledge and may abstain (review 1, H6: a corpus arm told to answer only from the
corpus would over-abstain). Tool calls are capped at 15 per question. Prompt parity is checked by script: after
replacing each arm's tool block and answer path with a placeholder, the 4 prompts of every batch must be
byte-identical; the prompts are hashed.

### 4.2 Question sets

140 questions, assigned opaque ids after a seeded shuffle (seed 20261004), in 28 batches of 5.

| Set | n | Source | Reference | Bias |
|---|---|---|---|---|
| **P** (primary) | 60 | researcher-style questions written blind by two independent agents (30 each) that saw no corpus, no other set and no repository file; 20 quantitative, 16 state-of-evidence, 12 recent, 12 "has anyone…" | two independent keys | none designed in |
| S1-A | 19 | blind seed set v1, part A, in scope; written from ITER Physics Basis chapter topics, a textbook and review scope | two independent keys | likely favours A (textbook knowledge) |
| S1-B | 25 | blind seed set v1, part B: one question per randomly sampled NF abstract, stratified by decade (16/25 pre-2000) | the abstract's claims | **favours B by construction** ("known-paper lookup") |
| S1-OOC | 6 | out of corpus: future events, a fictional machine, two non-fusion questions | "abstain / unknown / nonexistent" | small; tests abstention |
| S2 | 30 | exact facts from 10 papers of NF 66(12) (2026) with frozen gold from the extraction benchmark; written by an independent agent from the gold item only | the gold value (2 % tolerance) | **favours B and D by construction** (recency, in-corpus) |

**Why these sets.** Review 1 found that the original primary set decided the outcome (half catalogue lookups
favouring the corpus, half textbook questions favouring the bare model). P, written by agents with no corpus
access, became the only primary set; the earlier retrieval seed set (`eval/SEED_V1.md`) and S2 are secondary,
with biases labelled in every table and in the chart. The review asked for owner-written questions; the owner
delegated, and the deviation is declared. P was raised from 30 to 60 after review 2 showed that n = 30 detects
only differences of about 20 points.

**Contamination.** The "recent" questions in P (12) and all of S2 concern results that may post-date the model's
training data. This is intended: it is where retrieval of any kind should help most. It also means that A's
performance on those items measures recall of possibly unseen facts and should be read accordingly.

### 4.3 Answer keys

Keys were built and frozen before any arm ran.

- **P and S1-A: two independent keys.**
  - *Key 1 (web, Gemini):* Gemini 3.8 Flash searches the web with Google Search grounding (re-tried until at
    least one source is returned; ungrounded keys are flagged), then Gemini 3.1 Pro writes the key from those
    notes. The two-stage design exists because, in a probe, Gemini Pro at high thinking often skipped the search
    altogether; this was caught by a `grounded` check.
  - *Key 2 (web + corpus, Claude):* Claude processes with web search and the corpus write their own key without
    seeing key 1. Key 2 was merged for all 79 P and S1-A questions.
  - *Contest step:* Gemini 3.8 Flash compares the two keys and marks a question **contested** if they disagree on
    an essential fact (a different yes/no, a value beyond rounding, a different device, or answerable vs not).
    **11 questions** were contested.
  - Judges see both keys; where they conflict, an answer consistent with either is acceptable.
  - *Rationale:* each tool arm has one "friendly" key — key 1 used only the web, key 2 used web and corpus — so
    neither B nor C is privileged by the reference (review 2, H2: a web-only key can miss corpus-only literature
    and grade a correct B answer as contradicting the key).
- **S1-B:** the source abstract's specific claims (Gemini 3.8 Flash), flagged weak when no abstract exists.
- **S2:** the frozen gold value with its quote and page.
- **S1-OOC:** a fixed key: abstaining or stating that the thing is unknown or nonexistent is correct.

For "has anyone…" questions, "no published report found" is a definite answer (answerable = true); only future
events, nonexistent things or out-of-scope questions are unanswerable. The protocol does not include a check of
the keys by a human domain expert; §13 discusses this.

### 4.4 Normalisation and length

LLM judges reward length and recognise style. Every answer is therefore:

1. **cut to 200 words** (the brief's limit) before anything else, for every arm; the original length and a
   `truncated` flag are recorded;
2. **rewritten** by a fixed Gemini 3.8 Flash step into a neutral list of atomic claims with numbers, units and
   meaning-changing hedges kept, and citations, author names, titles, DOIs, references to sources, tools,
   searches or "the literature/corpus" removed (`normalise.md`);
3. **leak-checked** by regular expression for words that could identify an arm (corpus, library, chunk,
   database, search, web, internet, Google, retrieval, "according to the paper", "et al", arXiv, journal
   abbreviations, URLs, DOI patterns) and for loss of the answer's main value; on a hit the step is re-run once
   with a stricter instruction; remaining flags are counted per arm and reported.

A question that an arm failed to answer (a batch voided twice, a malformed file or a missing question) bypasses
normalisation with a fixed "no answer" record and is scored incorrect by rule, whatever a judge says.

### 4.5 Two blind judges

- **Gemini 3.1 Pro** (`judge.md`), one question per call, thinking high, structured output.
- **Claude Opus 5.5** (`judge_claude.md`), packets of 10 questions, each packet one isolated headless process
  with no tools except reading its prompt and writing its output.

Both see only the question, the key (both keys and the conflicts for P and S1-A) and, for each anonymous label
W/X/Y/Z, the answer's `abstained` flag, `value` and `claims` — nothing else (§6.3). Labels are shuffled
independently per judge (seeds 20261005 and 20261006), so position bias is not shared. Grades are `correct`,
`partial`, `incorrect` or `abstained`; "can't verify" is never "incorrect"; claims beyond the key are fine unless
they contradict it. Every question must receive exactly one valid grade per label from each judge; the Gemini
judge is re-called once if not, judge labels echoed as "Answer W" are mapped to the letter (raw label kept), and
aggregation refuses to run without full coverage. For an unanswerable key, an abstention is mapped to correct in
code, not left to the judges.

**Correctness** for the primary analysis is **strict consensus**: both judges say `correct`; `partial` counts as
incorrect. Sensitivity analyses: partial = correct, and each judge alone. Judge agreement is reported as Cohen's κ
over all grade pairs with the list of disagreements.

All arms and the Claude judge are the same model family. The Claude judge's self-preference, if any, applies to
every arm's answer equally (all four are Claude answers, normalised by Gemini); the second judge is from another
vendor, and strict consensus requires both.

### 4.6 Citations

Every cited work of every arm is checked the same way (`h2h/score.py`):

- a DOI (or a doi.org URL, whichever arm wrote it) is resolved through the doi.org handle API, which covers every
  registrar (Crossref, DataCite/arXiv, OSTI…); the title is compared with Crossref's;
- without a DOI, title and year are searched in Crossref and OpenAlex (10 hits each), with a strict,
  order-insensitive, punctuation-blind title match (token-sort ratio ≥ 90 after NFKC, tag stripping and Greek
  mapping);
- outcomes: `ok`; `miscited` (real work, wrong DOI or year off by more than one); `fabricated` (no matching
  record, URL or not); `incomplete` (no title and no DOI); `unverifiable_nonscholarly` (a non-DOI web page).

Citations are **secondary only** and are charted in the "favours the corpus by construction" panel: arm B copies
DOIs from tool output.

### 4.7 Primary endpoint, decision rule and statistics

- **Primary endpoint (single):** on P (n = 60), the proportion of questions answered correctly by strict
  consensus, **B versus C**, exact two-sided McNemar test on the paired outcomes; the paired difference is reported
  with a percentile bootstrap 95 % CI (10,000 resamples of questions, seed 20261007).
- **Decision rule:**
  - *corpus better than web:* B − C ≥ +10 points and p < 0.05;
  - *corpus worse than web:* C − B ≥ +10 points and p < 0.05;
  - *inconclusive* otherwise. Inconclusive is **not** evidence of equivalence and must never be presented as "as
    good as web".
- **Power, stated in advance:** with n = 60, p < 0.05 needs discordant splits of roughly 6–0, 8–1, 10–2 or 13–4.
  The +10-point condition (6 net questions) therefore almost coincides with significance; only large effects are
  detectable.
- **Secondary (reported, no decision):** D vs C (added value), B vs A, C vs A, D vs A, D vs B, and B vs C on every
  set (P, P-uncontested, S1-A, S1-B, S1-OOC, pooled S1, S2); abstention; contradiction of the key; fabricated
  citations; tool use, wall time and cost per arm; the sensitivity analyses above.

All pre-registered analyses and the verdict are implemented in code (`score.aggregate`) and were frozen with it,
so no analysis is chosen after seeing data (review 2, H6).

### 4.8 Reporting and the chart, fixed in advance

Every arm is reported on every set, with 95 % Wilson intervals and the tests above. The chart has three panels on
the same axes, with n under every bar:

1. **Primary — set P:** strict-consensus accuracy for A, B, C, D, with the verdict sentence as the panel title.
2. **Other sets:** S1-A and S1-OOC.
3. **"Favours the corpus by construction":** S1-B, S2 and the fabricated-citation rate, hatched, with the label
   printed on the panel.

Any public use shows panel 1 with the verdict sentence. No arm, question or set is re-run, dropped or re-graded to
change a result; re-runs happen only for audit violations or infrastructure failures, are listed, and the
originals are kept.

---

## 5. The agentic execution harness

This section is the paper's main contribution. The harness turns "run four agents on 140 questions and grade
them" into a procedure where each run is isolated, every tool use is either refused or audited, every input is
pinned by hash, and every step leaves a record. It is about 1,500 lines of Python in `src/fusion_corpus/h2h/`
(`questions`, `run`, `headless`, `audit`, `gemini_steps`, `stages`, `score`) plus tests.

### 5.1 From subagents to isolated headless processes

The first design ran each batch as a fresh subagent of the orchestrating session (never a `fork`, which would
inherit the orchestrator's context and therefore the question sets). Two problems ended that design before any
arm ran.

1. **A shared web budget.** The key-2 subagents exhausted the session-wide web-search budget (200 searches per
   session, shared by all subagents); a check confirmed that web search was refused afterwards. Arms C and D would
   have run without web search, an infrastructure failure that looks like a result. The 8 key-2 batches written
   by subagents were declared void, archived, and re-run.
2. **An inherited environment.** Subagents inherit the parent's environment: its MCP servers (the repository
   registered the live corpus server, so arm A could have reached the corpus), the user's global instructions and
   hooks (one rewrote shell commands), and the working directory (review 2, C5).

Every arm batch, key-2 batch and Claude-judge packet therefore runs as its **own headless process**
(`claude -p`, `h2h/headless.py`):

- the user message is a fixed one-line **pointer** to the batch's prompt file; the agent may read that file once;
  the prompt holds only `[opaque id] question` lines, the brief and the output path;
- working directory: an empty directory; `--strict-mcp-config` with an empty server list, so no MCP tool exists;
- **isolation flags:** `--setting-sources project --permission-mode dontAsk --disable-slash-commands`, so no
  user settings, memory file or hooks load, and any tool call not explicitly allowed is refused rather than
  prompted for or auto-approved;
- **effort pinned:** `--effort xhigh` (§6.2);
- `--tools` lists the arm's tools (A: Read, Write; B: + Bash; C: + WebSearch, WebFetch; D: all five);
- `--allowedTools` permits exactly: `Read` of the agent's own prompt file, `Edit` of its own answer file (under
  `dontAsk`, file writes are governed by `Edit` rules — a dry-run finding), `Bash(<wrapper>:*)` for corpus arms,
  and WebSearch/WebFetch for web arms;
- the answer file is deleted before each attempt, so a stale file can never pass for this attempt's answer;
- a timeout (1 hour) is a counted failed attempt, not a crash;
- the stream-json transcript is written to disk and is the record that the audit reads;
- duration, token usage, turn count, notional cost and permission denials are taken from the CLI's result record
  and appended to a run log.

Each process has its own web budget. Batches run four at a time, in the order A, B, C, D for batch 1, then batch
2, and so on, so that drift in API latency or model behaviour over the day does not line up with one arm.

### 5.2 Two lines of enforcement: native whitelist and use-based audit

A tool whitelist is only as good as the mechanism that enforces it. We found this out directly. The first
headless key-2 run used `--allowedTools` under the owner's normal settings; the transcript audit then flagged
non-wrapper shell commands in two batches (a `cd`-less path chained before a corpus search; `sed -i` on the
agent's own output file followed by a `python3` read of it) **with no permission denial recorded**. A probe with a
small model in an empty directory showed why: with the user's settings loaded (auto permission mode, allow
rules, a command-rewriting hook), the allow list was not exclusive — `ls /` ran, and `cat` was rewritten by the
hook. With the three isolation flags, the whitelisted command ran and `ls /`, `cat /etc/hosts` and `touch` were
all denied, and the model saw no user instructions (HEAD2HEAD §14). The protocol sentence "anything else is
denied" had been false for key 2 and the early pilots; the audit had been the only enforcement, and it had caught
both cases. (Neither command reached information outside the allowed sources; both batches were kept and flagged.
Key 2 is a reference key, not a measured arm, and its production under user settings is declared.)

The harness therefore enforces the policy twice:

1. **natively**, by the isolation flags and the per-file, per-command allow rules; refused calls are recorded as
   permission denials and never execute;
2. **by a scripted audit of every transcript** (`h2h/audit.py`), which never relies on the first line holding.

The audit is **use-based**: only the *use* of a tool can leak, so it checks what each executed call did rather
than scanning text for forbidden words. Rules, identical for every arm except for which tools are allowed:

- exactly one `Read`, of the agent's own prompt file;
- at least one write to the agent's own answer file and none elsewhere; rewriting the own answer file is allowed
  (no information is gained by it); a pure `cat > <own answer file> <<'TAG' … TAG` heredoc with a quoted tag and a
  single body counts as the write, anything chained to it does not;
- `Bash` only for B and D, and only as one call of the corpus wrapper with an allowed subcommand. The command is
  judged on **shell tokens**, not characters (`shlex` with POSIX rules and comment stripping disabled): any
  unquoted operator (`; & | < > ( )`), any `$` or backtick anywhere (expansion happens inside double quotes),
  newlines, or a different executable is rejected; a trailing `2>&1` is allowed. Twelve attack forms are covered
  by tests (`tests/test_h2h_audit.py`);
- `WebSearch` and `WebFetch` only for C and D; `WebFetch` only to public http(s) URLs (localhost, private address
  ranges, link-local and `.local` hosts are rejected, as is any non-http scheme such as `file://`);
- `ToolSearch` and `TodoWrite` are harmless (they read nothing);
- anything else — another `Read`, `Glob`, `Grep`, `Agent`, any MCP tool — is a violation;
- more than 15 tool calls per question (75 per batch, excluding the prompt read, writes and harmless tools) is a
  violation;
- natively denied calls are excluded from the use-based check (they never executed) but kept in the transcript
  and counted.

An earlier audit scanned for forbidden path fragments such as `gold`, which voided legitimate searches for the
Goldston heuristic drift model, a core scrape-off-layer topic (review 2, C3); and because voids trigger re-runs,
any rule that voids normal tool use gives tool arms extra draws the no-tool arm does not get (C4).

**Voids and re-runs.** A batch that fails the audit, or whose answer file is malformed, is re-run once from a fresh
process, with a re-run prompt that differs only in its output path (`bNN.rerun1`); the original output is kept. A
second void scores every question of the batch as no answer (incorrect), reported per arm. The counted attempt is
the **first** attempt that passed — never the better of two. `headless reaudit` re-applies the current audit to
stored transcripts and keeps the full verdict history in each record, so any audit change made before the freeze
is visible.

### 5.3 Freeze with hashes, checked by every later stage

The protocol's "keys frozen (sha256)" originally existed only as a sentence (review 2, H3). The harness now writes
`eval/h2h/FREEZE.json`, committed before any arm ran (commit `b31ea9a`), containing the sha256 of:

- the question file (private) and the public batch file;
- the merged keys (private);
- the corpus snapshot;
- the 112 original arm prompts (re-run prompts are generated later from the same template and are excluded —
  including them would have broken verification at the first re-run, a dry-run finding);
- every Gemini and Claude-judge prompt;
- the arm brief and the four tool blocks;
- **every source file of the harness code** (`h2h/*.py`);
- the label seeds and the question seed.

`verify_freeze()` recomputes the payload and compares the **union** of keys, so a newly added or removed entry
also fails. It is called before the arms start, before the Claude judge starts, and inside normalisation, Gemini
judging and aggregation; any difference stops the stage with the names of the changed entries. A probe edit to
`score.py` after the freeze was refused. Because the private files are git-ignored (§15), the committed hashes are
what tie a published result to the exact questions, keys, snapshot, prompts and code that produced it.

### 5.4 Pilots and a full dry run

Before the freeze, every arm was piloted on five practice questions outside the test set (HEAD2HEAD §11): the
first corpus-arm pilot was voided because it wrote its answer with a shell heredoc followed by a JSON-validation
one-liner; the brief was changed for every arm ("with the Write tool (not with a shell command)") and the re-run
passed. A hardened pilot of B and D followed the isolation change.

The owner then asked for a full, actively monitored runner test with no bias. The **entire pipeline** — keys 1 and
2, contest, freeze, four arms × two batches, validation, normalisation, both judges, citations and aggregation — ran
on six practice questions in a separate study root (`FUSION_H2H_ROOT=data/h2h_dryrun`) with its own freeze file,
and with every Gemini call tagged with the study root. No test question was used. The dry run produced twelve
changes, all made before the freeze and identical for every arm (HEAD2HEAD §15); the most consequential are
discussed in §6.2. Its practice results (κ 0.45 on 24 gradings; verdict "inconclusive" at n = 5) carry no
evidential weight.

An independent code reviewer, with no edit rights and no model calls, then checked the staged changes for arm
bias, identity leaks, forbidden access, silent data loss and breakage (HEAD2HEAD §16). It found no critical issue
and two high ones (§6.2), all fixed with tests before the freeze.

### 5.5 Failure handling

The guiding rule is that a failure is either retried by a fixed rule or recorded as a failure — never silently
absorbed into a denominator.

- **Agent runs:** audit voids and malformed files follow the re-run rule in §5.2; timeouts count as failed
  attempts; each attempt's audit record is stored separately.
- **Gemini steps** (keys, contest, normalisation, judging) retry transient errors with exponential backoff up to
  four attempts, are resumable (completed records are skipped on restart) and write each record as soon as it is
  produced. JSON that fails to parse is repaired by a schema-constrained call (keys) or re-requested.
- **Validation** (`stages validate`) checks that each counted answer file parses, is a list, and holds exactly its
  batch's question ids once; missing or extra ids are listed; word counts and over-cap counts are reported per
  arm.
- **Coverage assertions:** aggregation refuses to score unless every (arm, question) pair has a claims record, a
  Gemini grade, a Claude grade and a citation record.
- **Citations:** HTTP failures other than 404 raise, so a citation is never silently skipped.

### 5.6 Provenance

Everything is kept (METHODS §5):

- every headless run: full stream-json transcript, exact command line, exit code, duration, usage, notional cost,
  permission denials and audit verdict with history;
- every Gemini call of the study, verbatim: prompt, settings, raw response, usage, and for grounded calls the
  search queries and source URLs, tagged with the study root;
- the server-side corpus query log;
- the exact prompt text each agent read and its answer file;
- every subagent transcript from earlier phases (question writers, reviewers, annotators, the voided key-2 runs,
  pilots);
- the extraction benchmark's records, including a voided batch run.

### 5.7 Operational record of the main run

All facts in this subsection are operational measurements, not outcomes.

| Measure | A | B | C | D |
|---|---|---|---|---|
| Batches (5 questions) | 28 | 28 | 28 | 28 |
| Audit voids | 0 | 0 | 0 | 0 |
| Answers missing or malformed | 0 / 140 | 0 / 140 | 0 / 140 | 0 / 140 |
| Mean tool calls per batch | none | 33.6 corpus | 22.2 WebSearch + 21.2 WebFetch | 26.6 corpus + 2.6 WebSearch + 3.4 WebFetch |
| Mean wall time per batch | 93 s | 220 s | 437 s | 260 s |
| Notional cost (CLI-reported) | $8 | $42 | $60 | $41 |
| Answers over 200 words before truncation | 7 | 23 | 22 | 24 |
| Normalisation leak flags remaining | 1 | 6 | 6 | 3 |

All 112 arm batches passed the audit; 560 answers were collected with none missing or malformed. In the whole run
there was **one** native permission denial (a mistyped wrapper path), which never executed. Two Gemini
normalisation calls failed — one degenerate repetition output and one empty response — and were handled by
resuming the resumable stage; no record had been written for either. Gemini spend for keys, normalisation and
judging: not recorded per call (token usage is logged); Claude-judge notional cost: $9.17 (14 packets, CLI-reported).

Per question, mean tool use was about 6.7 calls for B, 8.7 for C and 6.5 for D (mostly corpus). These describe
behaviour, not quality.

---

## 6. Threats to validity and how each was handled

### 6.1 Overview

The threats below come from the incidents table (METHODS §4), the two adversarial reviews and the protocol
amendments (HEAD2HEAD §11–16). "Arm bias" means a mechanism that would change one arm's score relative to
another for reasons unrelated to answer quality.

| # | Threat | Direction | Found by | Handling | Residual |
|---|---|---|---|---|---|
| 1 | Question sets decide the outcome (catalogue lookups favour B; textbook items favour A) | both | review 1 (C1) | blind primary set P; biased sets secondary and labelled; hatched chart panel | P written by LLM agents, not researchers |
| 2 | No web arm: "corpus vs nothing" is a straw man | for B | review 1 (C2) | arms C and D; B vs C primary | — |
| 3 | Answers in question `notes`; set names in ids | leak | review 1 (C3) | arms see only opaque id + question | — |
| 4 | Corpus MCP server visible to every arm; user instructions, hooks inherited | leak, for B | reviews 1–2 | empty working dir, no MCP servers, isolated settings | — |
| 5 | Bash prefix check bypassable (`cli search x; cat gold`) | leak | review 1 (C3) | token-level whitelist + native rule | tested on 12 attack forms |
| 6 | Local PDF paths in search results invite reading | leak, for B | review 1 (C3) | stripped in evaluation mode | — |
| 7 | Snapshot written on every call / empty if missing | against B/D | review 2 (C1, C2) | read-only open, hash-checked wrapper, external query log | — |
| 8 | Forbidden-word scan voids "Goldston" searches | against B/D | review 2 (C3) | use-based audit | — |
| 9 | Normal agent behaviour (`2>&1`, TodoWrite) voids tool arms, giving them extra draws | mixed | review 2 (C4) | allowed; second void = no answer | — |
| 10 | Session-wide web budget exhausted by subagents | against C/D | key-2 agent note, verified | one headless process per batch | — |
| 11 | Tool whitelist not exclusive under user settings | leak | key-2 audit (violations without denial), probe | isolation flags; audit as second line | key 2 produced under user settings (declared) |
| 12 | Isolation dropped the effort setting; arms used ~⅓ of the tool calls | against B/C/D | dry run (tool counts) | `--effort xhigh` pinned for all | — |
| 13 | Audit voided a quoted `Q>1` query | against B/D | dry run | shell-token audit + tests | — |
| 14 | Judges sent `leak_problems` (could contain "web") | identity leak | dry-run code review | judges see `abstained`, `value`, `claims` only | — |
| 15 | Longer answers look more complete to judges | for longer arms | review 2 (M3), dry run | 200-word cut for all; counts reported | cut may remove content (§11) |
| 16 | Style and source mentions identify arms | identity leak | review 1 (H1) | Gemini claim normalisation + leak scan + re-run | flags remain: A 1, B 6, C 6, D 3 |
| 17 | Judge self-preference / same-family errors | for agreement with Claude | review 1 (C4) | second judge from another vendor; strict consensus; normaliser from another vendor | both judges are LLMs |
| 18 | Shared position bias between judges | mixed | review 2 (M4) | independent label shuffles | — |
| 19 | Web-only key misses corpus-only literature | against B | review 2 (H2) | dual keys + contested flag + P-uncontested | no human key check |
| 20 | Keys not actually frozen | any | review 2 (H3) | committed FREEZE.json + `verify_freeze` everywhere | — |
| 21 | Re-run prompts in the freeze block every later stage | infrastructure | dry run | freeze original prompts only | — |
| 22 | Missing or mislabelled judge grades silently "incorrect" | mixed | review 2 (H4), dry run | exact-coverage validation, label normalisation, re-call | — |
| 23 | Voided batch graded correct (empty record judged "abstained") | for voided arm | code review (§16, HIGH) | no-answer records bypass judges, scored incorrect | — |
| 24 | Citation checker misses arXiv/OSTI/web sources | against C/D | review 2 (H1) | doi.org handles, OpenAlex, URL class | — |
| 25 | Citation rules differed by URL presence; first-hit year check | mixed | code review (§16, HIGH) | single rule for all arms, all hits considered, tests | — |
| 26 | Curly vs straight quotes → real paper "miscited" | all arms | dry-run spot check | punctuation-blind title match + test | — |
| 27 | S1-OOC abstention depends on judge obedience | mixed | review 2 (H7) | mapping in code | — |
| 28 | Unplanned analyses chosen after data | any | review 2 (H6) | all analyses + verdict in frozen code | — |
| 29 | Inconclusive presented as equivalence | for B in marketing | review 2 (M1) | written prohibition; verdict wording in code | — |
| 30 | Chart mixes biased and unbiased sets | for B | review 2 (H8) | three fixed panels | — |
| 31 | Comment stripping in `shlex` could hide `; cmd` after `#` | leak | code review (§16) | comment stripping disabled | — |
| 32 | Timeout crashed the runner, allowing unlimited retries | mixed | code review (§16) | timeout = counted failed attempt | — |
| 33 | Stale answer file passes for a new attempt | mixed | code review (§16) | file deleted before each attempt | — |
| 34 | Drift over the run day | mixed | review 1 (L2) | interleaved A, B, C, D per batch | — |
| 35 | Contamination: model may have seen P/S1-A facts; may not have seen recent ones | for A on old, against A on new | design | "recent" kind in P; S2 labelled; A reported as floor | not measurable directly |
| 36 | Claude-judge audit counted an offset re-read of its own prompt as a second read and flagged all 14 judge runs | infrastructure | post-freeze audit | amendment 1: offset read of the same file is one read; re-audit, verdicts kept | grades unchanged |
| 37 | Citation check stopped by OpenAlex errors (HTTP 400 on `?`, then the free daily budget) | infrastructure | crash, HTTP 429 | amendments 1–2: punctuation dropped from the query; contact email/API key sent | same rule for every arm |
| 38 | Claude usage limit (HTTP 429) made attempts fail instantly with no answer | against whichever arms ran then | instant $0 voids in the run log (facts study) | records archived as infrastructure failures, not attempts; resumed after reset | — |
| 39 | Fixed paths in shared `/tmp` for the empty working directory and MCP config (config injection by another local account) | leak | security review after the runs | per-process private directory (`mkdtemp`, 0700); past runs verified unaffected | — |

### 6.2 The bias bugs caught by the dry run and the reviews

Five of these deserve detail, because each would have moved the primary comparison without any visible error.

**An audit that punished the corpus arms for quoting.** The first audit rejected corpus commands containing any
shell metacharacter, character by character. In the dry run a corpus-arm batch was voided twice — all five
questions scored as no answer — because a quoted query contained `Q>1` (a fusion-gain threshold), which Claude
Code had correctly allowed. Only B and D could be hit (web queries are not shell commands), and physics queries
routinely contain `>`, `<`, `|` or parentheses. The fix is the token-level check of §5.2; `reaudit` re-applied it
to stored transcripts, keeping the old verdict.

**Leak flags shown to the judges.** The normalisation record included `leak_problems` (e.g. `leak: 'web'`), and
both judges were sent the whole record — telling them which arm wrote the answer and defeating normalisation. A
code review during the dry run caught it; judges now see only `abstained`, `value` and `claims` (`JUDGE_VIEW`).

**Effort lost by isolation.** The isolation flags also stopped the owner's `effortLevel: xhigh` from loading,
which key 2 and the early pilots had inherited. The first dry-run arms ran at default effort; the symptom was in
the tool counts (the corpus arm used 4–5 commands for three questions, against 14–16 for five earlier). Lower
effort is not neutral: it cuts tool use, which is what distinguishes B, C and D from A. Effort is now pinned for
arms, key 2 and the Claude judge; the low-effort outputs are archived.

**Self-preference.** All four arms and one judge are Claude Opus 5.5, so the Claude judge might favour answers
matching Claude's own priors — plausibly the bare model's. We cannot remove this, only bound it: judges see
Gemini-normalised claim lists, not Claude prose; the second judge is from another vendor; the primary score needs
both; label orders are independent; and each judge alone is reported as a sensitivity analysis.

**Length.** The brief limits answers to 200 words, but tool arms wrote more: 23, 22 and 24 answers over the
limit for B, C and D, against 7 for A. A longer answer is more likely to "contain all essential facts", so every
answer is cut to 200 words in code before normalisation and the number cut is reported. The cost of the cut
(a late correct fact removed) falls mostly on the tool arms (§11).

The code review found two more issues invisible in aggregate numbers: a twice-voided batch's empty record was
still sent to the Gemini judge, where on an out-of-corpus question it would look like a correct abstention; and
the citation checker classified a scholarly-looking title differently depending on whether a URL was attached,
which web arms do. Both were fixed with tests before the freeze.

### 6.3 What the arms and judges can and cannot see

| Who | Sees | Does not see |
|---|---|---|
| Arms | the brief, its tool block, five `[opaque id] question` lines, its own output path | set names, sources, notes, keys, gold, other arms' answers, any repository file |
| Normaliser (Gemini 3.8 Flash) | the question and one answer (text, value, abstain flag), cut to 200 words | arm, citations as structured data, other answers |
| Judges | question, key(s) and conflicts, per label: `abstained`, `value`, `claims` | arm identity, label maps, leak flags, word counts, truncation flags, usage |

### 6.4 After the freeze

Three kinds of event after the freeze are recorded here (HEAD2HEAD §17; METHODS §4 and §8).

**Post-freeze amendments (code only).** After the arms and judges had run, and before any result was computed, two
infrastructure bugs surfaced. The Claude-judge packets exceeded one read page, so every judge read its prompt
once more with an offset; the audit counted two reads and flagged all 14 judge runs. The transcripts showed exactly
two reads of the judge's own prompt and one write, and the audit now treats an offset read of the same file as one
read; the re-audit passed all 14, and the grades used are those the runs wrote. Separately, the citation checker
stopped when OpenAlex rejected a query containing `?` (amendment 1: punctuation dropped from the query) and again
when the free daily OpenAlex budget ran out (amendment 2: contact email and API key sent). Each time the freeze was
re-written after verifying that the only changed entry was the code hash of `audit.py` or `score.py`; no answer,
key, claim or grade was changed. The Gemini judge also reached its daily quota at 134/140 questions; the frozen
judge model was not substituted, and the remaining 6 were graded after the quota reset.

**Usage-limit interruption.** In the recent-facts study (§7.10), the Claude session limit returned HTTP 429 on every
call from about 17:05, so 95 attempts ended in less than a second with no tool call and no answer file. The chain
was stopped; those records were identified from their transcripts, archived as infrastructure failures rather than
counted as attempts, and the run resumed after the reset with the frozen materials unchanged (`verify_freeze`
passed), running only the batches without a passing attempt.

**Security fix.** A security review after the runs found that the headless runners used fixed paths in the shared
`/tmp` for the empty working directory and the empty MCP-server configuration; another local account could have
pre-created them with a configuration that the isolated runs would load. For every past run, the directories and
the configuration were owned by the owner's account, empty, the configuration exactly `{"mcpServers": {}}`, and
every transcript's initial record lists no MCP server, so no run was affected. All three runners now create a fresh
private directory per process. This changes the code hashes of `h2h/headless.py` and `meta2h/run.py` after those
studies were completed and scored; their recorded results are unaffected.

---

## 7. Results

*Every number in §7.1–7.9 comes from `data/h2h/results.json` and `eval/h2h/RESULTS.md` (commit `c21209a`), except
where a recomputation from the stored per-question grades is stated. Cell convention for accuracy: `k/n = p %
[Wilson 95 % CI]`.*

### 7.1 Run summary

The run completed with 112/112 arm batches passing the audit and 560/560 answers collected (§5.7). No question
was scored as no answer (`no_answer` = 0 for every arm). Key 2 covered all 79 P and S1-A questions; 11 were
contested (10 in P, 1 in S1-A), leaving 50 uncontested questions in P. Ungrounded key-1 keys: 1 (h005).
Weak S1-B keys (no abstract): 0.

### 7.2 Primary endpoint: B vs C on P

| | C correct | C not correct |
|---|---|---|
| **B correct** | 19 | 9 |
| **B not correct** | 12 | 20 |

- Accuracy, strict consensus: B 28/60 = 47 % [35–59 %]; C 31/60 = 52 % [39–64 %].
- Paired difference B − C: −5.0 points, bootstrap 95 % CI −20.0 to +10.0.
- Exact two-sided McNemar: p = 0.664 (discordant 9–12).
- **Verdict (pre-registered rule): inconclusive (not evidence of equivalence).**

Verdict sentence for panel 1 of the chart: "Primary (set P): inconclusive (not evidence of equivalence)"

### 7.3 Accuracy by arm and set (strict consensus)

| Set | n | A | B | C | D |
|---|---|---|---|---|---|
| **P (primary)** | 60 | 16/60 = 27 % [17–39 %] | 28/60 = 47 % [35–59 %] | 31/60 = 52 % [39–64 %] | 37/60 = 62 % [49–73 %] |
| P-uncontested | 50 | 14/50 = 28 % [17–42 %] | 24/50 = 48 % [35–61 %] | 26/50 = 52 % [39–65 %] | 35/50 = 70 % [56–81 %] |
| S1-A | 19 | 15/19 = 79 % [57–91 %] | 15/19 = 79 % [57–91 %] | 16/19 = 84 % [62–94 %] | 19/19 = 100 % [83–100 %] |
| S1-OOC | 6 | 4/6 = 67 % [30–90 %] | 4/6 = 67 % [30–90 %] | 4/6 = 67 % [30–90 %] | 4/6 = 67 % [30–90 %] |
| S1 pooled | 50 | 22/50 = 44 % [31–58 %] | 42/50 = 84 % [71–92 %] | 35/50 = 70 % [56–81 %] | 46/50 = 92 % [81–97 %] |
| *S1-B (favours corpus by construction)* | 25 | 3/25 = 12 % [4–30 %] | 23/25 = 92 % [75–98 %] | 15/25 = 60 % [41–77 %] | 23/25 = 92 % [75–98 %] |
| *S2 (favours corpus by construction)* | 30 | 0/30 = 0 % [0–11 %] | 30/30 = 100 % [89–100 %] | 7/30 = 23 % [12–41 %] | 30/30 = 100 % [89–100 %] |

Figure 1 (fixed in advance, §4.8): `eval/h2h/chart.png`.

### 7.4 Secondary contrasts

Paired, strict consensus; `x only`/`y only` are the discordant counts; difference in points with bootstrap
95 % CI; exact McNemar p. No decision attaches to these.

| Contrast | Set | x only | y only | x − y (pp) [95 % CI] | p |
|---|---|---|---|---|---|
| D vs C (added value of corpus on top of web) | P | 13 | 7 | +10.0 [−5.0, +25.0] | 0.263 |
| B vs A | P | 17 | 5 | +20.0 [+5.0, +35.0] | 0.0169 |
| C vs A | P | 18 | 3 | +25.0 [+11.7, +38.3] | 0.00149 |
| D vs A | P | 26 | 5 | +35.0 [+18.3, +50.0] | 1.92 × 10⁻⁴ |
| D vs B | P | 11 | 2 | +15.0 [+3.3, +26.7] | 0.0225 |
| B vs C | P-uncontested | 9 | 11 | −4.0 [−22.0, +14.0] | 0.824 |
| B vs C | S1-A | 2 | 3 | −5.3 [−26.3, +15.8] | 1.000 |
| B vs C | S1-OOC | 0 | 0 | 0.0 [0.0, 0.0] | 1.000 |
| B vs C | S1 pooled | 10 | 3 | +14.0 [+0.0, +28.0] | 0.0923 |
| B vs C | S1-B | 8 | 0 | +32.0 [+16.0, +52.0] | 0.00781 |
| B vs C | S2 | 23 | 0 | +76.7 [+60.0, +90.0] | 2.38 × 10⁻⁷ |
| D vs C | S2 | 23 | 0 | +76.7 [+60.0, +90.0] | 2.38 × 10⁻⁷ |

The full grid (six contrasts × seven sets) is in `results.json` under `tests`; all of them are also printed in `eval/h2h/RESULTS_tables.md`.

### 7.5 Sensitivity analyses on P

| Scoring | A | B | C | D | B − C (pp) | p |
|---|---|---|---|---|---|---|
| Strict consensus (primary) | 16/60 = 27 % [17–39 %] | 28/60 = 47 % [35–59 %] | 31/60 = 52 % [39–64 %] | 37/60 = 62 % [49–73 %] | −5.0 | 0.664 |
| Partial = correct (both judges) | 43/60 (72 %) | 47/60 (78 %) | 51/60 (85 %) | 58/60 (97 %) | −6.7 | 0.424 |
| Gemini judge alone | 19/60 (32 %) | 28/60 (47 %) | 34/60 (57 %) | 39/60 (65 %) | −10.0 | 0.286 |
| Claude judge alone | 20/60 (33 %) | 34/60 (57 %) | 35/60 (58 %) | 42/60 (70 %) | −1.7 | 1.000 |

Per-arm counts in the three sensitivity rows are from `results.json`. The paired B − C tests for these rows are not
stored there; they were recomputed from the stored per-question grades (`data/h2h/private/judge_*.json` and the label
maps) with the frozen scoring rules, a recomputation that reproduces the stored primary test exactly (9–12, p =
0.664). Discordant B-only–C-only counts: partial = correct 5–9, Gemini alone 8–14, Claude alone 11–12. In every
scoring, C is at or above B and D is highest.

### 7.6 Abstention and contradiction on P

| | A | B | C | D |
|---|---|---|---|---|
| Abstained (both judges) | 4/60 | 4/60 | 1/60 | 2/60 |
| Contradicts the key (either judge) | 12/60 | 12/60 | 14/60 | 5/60 |

### 7.7 Judge agreement

- Cohen's κ over all 560 (arm, question) grade pairs (expected 560): 0.77.
- Disagreements: 74; by arm: A 30, B 17,
  C 19, D 8.
- Gemini label problems after the re-call: 0.

### 7.8 Citations (secondary; favours the corpus by construction)

| | A | B | C | D |
|---|---|---|---|---|
| Answers with ≥ 1 citation | 87/140 | 135/140 | 124/140 | 136/140 |
| Cited works | 208 | 368 | 407 | 390 |
| ok | 195 | 364 | 357 | 368 |
| miscited | 7 | 3 | 7 | 2 |
| fabricated | 4 | 1 | 1 | 1 |
| incomplete | 0 | 0 | 0 | 0 |
| unverifiable_nonscholarly | 2 | 0 | 42 | 19 |
| Answers with ≥ 1 fabricated citation (all 140) | 4/140 | 1/140 | 1/140 | 1/140 |

### 7.9 Cost, time and length

See §5.7 for tool use, wall time, notional cost, over-length answers and remaining leak flags per arm. Mean
answer length before truncation: A 124.3, B 157.7, C 156.1, D 155.2 words.

### 7.10 Fair test of recent exact facts

S2 and S1-B (§7.3) favour the corpus by construction: their questions were written from papers the corpus holds.
A separate pre-registered study (`eval/FACTS.md`) asks the same question without that bias, on facts from a random
sample of recent papers in the corpus's stated scope, including papers the corpus holds only as abstracts.

- **Sample.** 100 papers drawn at random (seed 20261013) from the *Nuclear Fusion* and PPCF research articles
  published 2025-01-01 to 2026-10-01 in the catalogue (a frame of 1,873 at the draw), without regard to full-text
  status; 88 of the 100 have full text.
- **Items.** For each paper a headless Claude process with no tools except writing one file, given only that
  paper's text, wrote an ABS item (an exact fact stated in the abstract), a BODY item (an exact fact stated only in
  the body; full-text papers only) and, for a seeded 60 papers, a LOOKUP item ("which recent paper reported …? Give
  its DOI"). Items are validated automatically (quote present in the source, key value next to a compatible unit,
  no title, author surname, DOI or key value in the question, paraphrase enforced). Final set before the freeze:
  **188 items from 97 papers** (45 ABS, 84 BODY, 59 LOOKUP; 20 dropped and listed); 174 items from full-text papers,
  14 from abstract-only papers.
- **Arms and grading.** The same four arms, model, brief, isolation and audit as the Q&A study, against the
  full-corpus snapshot of §8 (3,494 full texts; abstracts of all 24,101 catalogued works). Grading is code, with no
  judge model: a value is correct within ±2 % (or the key's printed rounding) after unit conversion; a LOOKUP is
  correct if the key DOI is cited.
- **Pre-registered analysis (v1, after an adversarial review that blocked the first freeze).** Co-primary: B vs C
  and D vs C on ABS + BODY items, paper-clustered sign-flip permutation test, Holm correction; a verdict needs Holm
  significance and a difference of at least 10 points. LOOKUP and the abstract-only slice are reported separately;
  BODY is labelled "full-text search", because BODY items exist only for papers the corpus holds.
- **Status.** Arm runs started on 2026-10-03 and were interrupted by a usage limit (§6.4); they resumed with the
  frozen materials unchanged; 152/152 runs passed the audit. Full results: `eval/facts/RESULTS.md`.
- **Result.** Corpus better than web on exact facts (+19.4 points, p = 1 × 10⁻⁵), corpus + web better than web
  (+20.2, p = 5 × 10⁻⁶). By stratum: ABS 93 / 91 / 96 % (B / C / D; no difference), **BODY 93 / 64 / 93 % (B − C
  +28.6, p = 5 × 10⁻⁶; 28 web abstentions)**, LOOKUP 98 / 83 / 100 % (+15.3, p = 0.012); facts published after the
  model's training cutoff: B 96 %, C 57 %. Mechanism: the web arm fetched iopscience.iop.org 171 times and 46 fetches
  (27 %) hit IOP's bot challenge; it fell back on arXiv and OSTI preprints. Grader checks: 74/75 random gradings
  agree with a hand check; among answers containing the key number, 19 grader false negatives were found, mostly
  against the corpus arms (B 9, C 3, D 7), so the frozen grade understates the corpus lead (≈ +24 points
  hand-corrected).

| | A | B | C | D |
|---|---|---|---|---|
| ABS + BODY, correct | 3/129 (2 %) | 120/129 (93 %) | 95/129 (74 %) | 121/129 (94 %) |

Co-primary B vs C: +19.4 points, p = 1.0 × 10⁻⁵ (paper-clustered permutation); D vs C: +20.2 points,
p = 5.0 × 10⁻⁶; verdict: corpus better than web; corpus + web better than web (both Holm-significant, ≥ 10 points). LOOKUP: B 58/59 (98 %), C 49/59 (83 %), D 59/59 (100 %); B vs C +15.3 points, p = 0.012. Abstract-only slice:
B 14/14, C 13/14, D 14/14 (no difference; n = 14).

### 7.11 Facts printed in tables

**Design.** The pre-registered study is `eval/TABLE_FACTS.md`, run on 2026-10-04. It used a table layer reconstructed from
the PDF layout: 3,893 of the 4,429 captioned tables (88 %), searchable with `tables` and `table`. A writer with no tools
saw the rendered image of one random table from each of 100 random 2025–26 NF/PPCF papers, none of them in the §7.10
sample. It wrote one question per table, keyed to a printed cell value, and each key was validated in the PDF text of
the table region. 69 items survived validation. A hand check of 25 items found every question mapped to a unique cell
and the correct value. There were five arms: model alone (A), text corpus (B), corpus + tables (E), web (C), and
corpus + tables + web (F). The co-primaries were E vs B and E vs C, tested with exact McNemar and Holm correction.

**Result.**

| | A | B | E | C | F |
|---|---|---|---|---|---|
| correct | 2/69 (3 %) | 69/69 (100 %) | 68/69 (99 %) | 39/69 (57 %) | 68/69 (99 %) |

- **E vs C: +42.0 points, 29 vs 0 discordant, p = 3.7 × 10⁻⁹. Corpus better.**
- **E vs B: −1.4 points, p = 1.0. Inconclusive at ceiling.** The key occurs in the text passages for all 69 items, so
  structured access had nothing left to add, although E used the table commands in 13 of 14 batches.
- **Web reached the paper.** On the 16 items where the web arm fetched the paper without a bot challenge, it scored
  8/16 against the corpus's 16/16 (+50, p = 0.008). Reaching the page did not reliably give it the table value.
- **Grader checks.** A random hand check of 28 grades agreed with the grader on all 28. Of the 6 wrong non-abstaining
  answers, 4 were grader false negatives ("vol%" and superscript exponents, on E, F and C), 1 was a strict first-reading
  miss, and 1 was a true error. Hand-corrected figures are E 69 and C 40 (or 41); the verdicts are unchanged.
- Full results: `eval/tabfacts/RESULTS.md`.

### 7.12 Papers public only after the model's training cutoff

**Design.** The pre-registered study is `eval/POSTCUT.md`. Its frame is NF/PPCF papers whose earliest public date
is on or after 2026-06-01. That date is the minimum of Crossref DOI registration (i.e. accepted manuscript online),
online, print and catalog dates, and the first version of a matching arXiv preprint. 105 of the 475 issue-dated
candidates (22 %) were public earlier and were excluded. The sample is 100 random eligible papers, coverage-inclusive
(31 without full text), giving 185 items. Arms A, B (corpus with tables), C and D. Primary tests: B vs C and D vs C
on fact items, paper-clustered permutation, Holm.

**Result.** Facts: A 3/124, B 121/124 (98 %), C 60/124 (48 %), D 118/124.
- **B vs C +49.2 points, p = 5 × 10⁻⁶. D vs C +46.8, p = 5 × 10⁻⁶.** Both verdicts: better than web.
- By kind: body +66, table +75, abstract +14, lookup +23.
- Where the web agent fetched the paper and the value was in the fetched text (17 items), it matched the corpus
  (17/17).
- Grader checks: 36/37 random grades agreed with a human reading. 7 grader false negatives were found; verdicts
  unchanged.
- Full results: `eval/POSTCUT_RESULTS.md`.

---

## 8. Evidence gathering for meta-analysis

The Q&A study measures answering. A literature meta-analysis needs something else: finding the published
measurements of a quantity, with sources, and synthesising them. This study (`eval/META2HEAD.md`; results
`eval/meta2h/RESULTS.md`, `data/meta2h/results.json`) asks whether the corpus is a better tool than the web for that.

### 8.1 Design

- **Tasks chosen blind to the corpus.** A headless Claude process with no tools except writing one file proposed 10
  candidate tasks under written criteria (a quantity with units, at least 8 published data points from at least 3
  devices or groups, a published synthesis as reference, at most 2 tasks per topic). A mechanical rule (id order,
  reference confidence high or medium, at most 2 per topic) selected 6 test tasks and 1 practice task. No candidate
  was checked against the corpus before selection.
- **Arms and execution.** The same four arms, model, effort, isolation and use-based audit as the Q&A study, against
  a new read-only, sha256-pinned snapshot of the full corpus (3,494 full-text papers, 165,953 passages). Each arm ran
  each task twice: 6 × 4 × 2 = 48 runs, each returning at most 40 data points with a verbatim quote per point, and a
  synthesis (at most 150 tool calls, 60 minutes).
- **Verification, blind to arm.** All points of a task were pooled, deduplicated (DOIs anywhere, arXiv ids, device
  aliases, 2 significant figures), grouped by source work and shuffled. A Claude verifier with both the corpus and
  the web saw only value, unit, device, conditions, quote and bibliographic fields, and graded each point
  `supported`, `partial`, `unsupported`, `source_not_found` or `inaccessible`, plus whether it met the task's
  inclusion criteria. An independent second verifier re-checked a random 25 %. Every step used Claude because the
  Gemini quota was limited (owner constraint).
- **Pre-registered analysis.** The metric V is the number of distinct relevant supported points. The unit of
  analysis is the task (the two replicates share a task's difficulty): V summed over the replicates gives 6 paired
  tasks. Co-primary: B vs C and D vs C, exact two-sided Wilcoxon signed-rank tests with Holm correction at α = 0.05;
  the smallest attainable p with 6 tasks is 0.031, so a significant result needs every task to agree.
- **Review and pilot.** An adversarial review before the freeze (3 critical, 5 high findings: failed verifier
  batches still scored, arm identity visible to the verifier, pseudo-replicated pairs, among others) and a pilot on
  the practice task led to changes, all made before the freeze (commit `fc74e85`).

### 8.2 Results

All 48 runs passed the audit on the first attempt; 35 verifier jobs (first pass and second verifier), one of which
needed its pre-registered re-run; no point was left unverified. No arm cited a work that does not exist
(`source_not_found` = 0 in every arm).

| Arm | Runs | Mean time / run | Notional cost | Claimed points | Verified relevant (V) | Precision (median) | Distinct verified sources | Inaccessible to verifier |
|---|---|---|---|---|---|---|---|---|
| A model only | 12 | 2.1 min | $5 | 30 | 6 | 0.29 | 6 | 16 |
| B + corpus | 12 | 12.9 min | $59 | 121 | 63 | 0.97 | 46 | 1 |
| C + web | 12 | 21.3 min | $72 | 141 | 84 | 0.77 | 53 | 28 |
| D + corpus + web | 12 | 15.3 min | $70 | 116 | 75 | 0.82 | 47 | 1 |

Verifier jobs cost $43 (notional).

| Co-primary | Per-task differences in V (t01…t06) | Totals | Exact Wilcoxon p | Verdict (pre-registered rule) |
|---|---|---|---|---|
| B vs C | −2, −3, +1, +5, −10, −12 | 63 vs 84 | 0.31 | inconclusive (not evidence of equivalence) |
| D vs C | −5, +6, −1, +9, −5, −13 | 75 vs 84 | 0.81 | inconclusive (not evidence of equivalence) |

Both co-primary comparisons are **inconclusive**. Secondary results: every tool arm beat the bare model on every
task (B, C and D vs A: p = 0.031, the minimum attainable); D vs B +12 verified points in total (p = 0.44); distinct verified
sources B 46, C 53, D 47 (B vs C p = 0.50). The accessibility sensitivity analysis, which counts `inaccessible`
relevant points as verified, gives B 64 vs C 106 (p = 0.16): web points were more often behind paywalls (28
inaccessible points for C against 1 for B), so counting them widens C's lead without changing the verdict.

**Precision.** The corpus arm's points were the most often verifiable: median precision 0.97 for B, against 0.77
for C and 0.82 for D. A plausible reason, not tested here, is that a corpus point comes with the passage it was read from,
while a web point may come from a page the verifier cannot open.

**Per task.** V per replicate (run 1 / run 2):

| Task | A | B | C | D | Union of verified points |
|---|---|---|---|---|---|
| t01 L-H threshold vs Martin | 0/0 | 4/7 | 3/10 | 5/3 | 30 |
| t02 isotope mass exponent | 1/1 | 9/7 | 9/10 | 13/12 | 51 |
| t03 pedestal width coefficient | 1/0 | 8/8 | 8/7 | 8/6 | 26 |
| t04 λq vs B_pol (Eich) | 0/0 | 7/3 | 3/2 | 4/10 | 22 |
| t05 current-quench time | 1/0 | 2/2 | 10/4 | 7/2 | 20 |
| t06 runaway threshold field | 1/1 | 5/1 | 9/9 | 3/2 | 22 |

The web arm's lead comes from the two disruption and runaway-electron tasks (t05, t06), where much of the
literature is published outside the corpus's two journals; the corpus arms lead on t04 and tie on t03.

**Verifier agreement.** The second verifier re-checked 82 points: exact agreement 0.70, Cohen's κ 0.55, **below
the pre-registered 0.6 flag**; agreement on whether a point counts (supported and relevant) was 0.85. The primary
uses the first pass, as pre-registered; the lower agreement is a limitation of every number in this section.

**Estimates against the references (descriptive).** The references come from the task generator's memory and are
not ground truth, so this comparison is descriptive only. t04 (reference exponent −1.19): B −1.15, D −1.24, C −0.89
(run 1). t06 (reference E/E_CH ≈ 2–5): B ≈ 5, C ≈ 7.8, D 3–5 (run 1). t03 (reference C = 0.076): all tool arms
≈ 0.09 from device means, each noting 0.076 as the canonical value. t05 (reference ≈ 1.7 ms m⁻²): every arm,
including A, ≈ 1.67–1.7.

Agents in both the corpus and the web arms wrote, unprompted, that much of the data they needed is shown only in
figures; §10 follows this up.

---

## 9. Stability across identical runs

The Q&A study ran each arm once. This study (`eval/DRIFT.md`, written before the re-runs; results
`eval/DRIFT_RESULTS.md`, `data/h2h_drift/drift_results.json`) asks how often an arm's answer to the same question
changes between identical runs.

**Design.** All 112 Q&A batches were re-run once with byte-identical prompts (only the output path differs), after
the Q&A and meta-analysis studies had finished, with the same isolation, model, effort and audit; 112/112 audits
passed. The replicate answers went through the same Gemini normaliser and the same frozen Claude judge prompt. The
Gemini judge was not used (rate-limited), so correctness here is the Claude judge's grade alone (`correct`, or
`abstained` on an unanswerable question) and is not comparable with the strict-consensus figures of §7.

**Judge-noise control.** A flip can come from the answer or from the judge. The original answers were therefore
judged a second time by the Claude judge with byte-identical packets; the share of questions whose correctness
differs between these two judgings of the *same* answer is the judge-noise flip rate.

| Arm | Questions whose correctness flipped between runs | Judge-noise flips (same answer judged twice) | Net drift | Correct, run 1 → run 2 (Claude judge, of 140) |
|---|---|---|---|---|
| A model only | 15 / 140 (10.7 %) | 9 | +4.3 pts | 46 → 49 |
| B + corpus | 17 / 140 (12.1 %) | 5 | +8.6 pts | 109 → 112 |
| C + web | 31 / 140 (22.1 %) | 2 | +20.7 pts | 81 → 86 |
| D + corpus + web | 17 / 140 (12.1 %) | 5 | +8.6 pts | 119 → 116 |

No arm's accuracy changed significantly between runs (paired exact McNemar, p 0.47–0.63): the drift is churn in
*which* questions are answered correctly, not a change of level. On P (60 questions) the flips were A 8, B 14,
C 19, D 13. On S2 the corpus arms never flipped (B and D 0/30) and the web arm flipped on 4/30. The web arm's churn
is not explained by judge noise, which was lowest for C (2 flips).

**Exploratory tests (not pre-registered).** The protocol pre-registered per-arm reporting only. Paired exact
McNemar tests on the flip indicator give: web flips more than corpus (25 vs 11
discordant questions, p = 0.029), more than corpus + web (24 vs 10, p = 0.024) and more than the model alone (28 vs
12, p = 0.017); corpus against the model alone shows no difference (p = 0.84). These p-values are exploratory.

**Reading.** In this run, the web-search arm gave a different right-or-wrong answer to the same question on about
one question in five, roughly twice as often as the corpus arm or the corpus + web arm. Limitations: one replicate
per arm; one judge; the comparison between arms is exploratory.

---

## 10. Toward evidence the web cannot see: figure and table data

The meta-analysis agents' remark that much of the data is only in figures points to a source that neither text
search of the corpus nor web search reads. A separate programme (`eval/FIGURES.md`) is designed to extract such data
once, offline, and cache it. It is reported here as an **interim result, negative so far**.

**Design.** Each value gets exactly one method label, cheapest exact source first: attached data files (M1), tables
(M2), vector plots read from the PDF's drawing paths (M3), captions and text mentions (M4) and, only for raster
plots a task actually needs, a vision model that reports pixel positions while code does the calibration (M5). The
acceptance thresholds were fixed before any extraction: a method may write values for a figure type only if, on the
**real** benchmark set, point precision is at least 0.95 and recall at least 0.80, the median error is at most 1 %
of the axis range (90th percentile at most 3 %), and axis calibration is right on at least 95 % of figures.

**Census (free, all 3,494 papers, 0 errors).** 43,932 figures (15,052 vector, 26,409 raster, 2,244 mixed, 227
without a graphic found), of which 20,765 are plot candidates (8,097 vector, 11,402 raster, 1,266 mixed); 4,429
captioned tables; 954 papers with a data-availability statement, 472 "on request" and 155 with an open link, so
attached data is a real but small source. Only 22 % of vector plots carry their tick labels as text; in 78 % the
labels are outlined glyphs, so reading them needs OCR.

**Synthetic benchmark.** 300 journal-style plots with 6,770 known points, each as a vector PDF and as rasters. The
vector extractor (M3) reached precision 1.000, recall 0.994, median error 0.21 % of the axis range (90th percentile
0.34 %), with the axis scale right on 99 %; 3 plots were not calibrated and were flagged rather than guessed. The
benchmark caught four bugs before any real figure was touched.

**Real figures (interim).** On a seeded sample of 300 real plot candidates, successive versions of M3 raised the
number of figures yielding points from 1/300 to 23/300; the current version yields points for 18/300. In a review of
14 overlays, about half were clean, about a third contained false points (single outlined letters or labels) and
the rest were unclear at review resolution. Real-figure precision is **below the pre-registered bar**, and coverage
is limited mainly by axes detection (no axes box found in 239/300 figures). **No real figure values are written to
the corpus or used in any result in this paper.** The next steps are a real-figure truth set, so that every change
is scored rather than inspected, and better axes detection; a fifth arm on the frozen meta-analysis tasks would
follow only if the real benchmark passes.

---

## 11. Discussion

**The primary comparison is inconclusive.** The primary comparison did not meet the pre-registered rule: B − C =
−5.0 points (95 % CI −20.0 to +10.0, p = 0.664). This is **not** evidence that the corpus is as good as web search,
and must not be presented as such; the confidence interval includes differences of practical size in both
directions, from 20 points in favour of web search to 10 points in favour of the corpus. With 60 paired questions,
only differences of roughly 10–20 points could be detected, so a modest advantage either way remains possible. The
operational record still matters for a user: the corpus arm reached its answers with fewer calls and about half the
wall time of the web arm (§5.7). The secondary sets show where each tool helps, with their construction biases kept
in view: every tool arm beat the bare model on P (B, C and D vs A: +20.0, +25.0 and +35.0 points, p ≤ 0.0169);
corpus plus web scored highest on P (62 %) and contradicted the key least (5/60), but its +10.0-point lead over web
alone is not significant (p = 0.263; on the 50 uncontested questions, a sensitivity analysis, +18.0 points, p =
0.049); on S1-A all arms were close (79–100 %); and on the two sets written from corpus papers the corpus arms led
web search by 32 points (S1-B) and 76.7 points (S2), by construction. A larger blind set, ideally of questions
written by working researchers, is the obvious next step.

**The follow-up studies point the same way.** In evidence gathering (§8), both co-primary comparisons were also
inconclusive; the web arm found more verified points in total (84 against 63), mostly on topics published outside
the corpus's two journals, while the corpus arm's points were more often verifiable (median precision 0.97 against
0.77). Across identical runs (§9), web answers churned about twice as often as corpus answers, a difference found
by exploratory tests. None of this establishes that the corpus is better or worse than web search; it describes
how the two tools behave differently.

**Cautions.** Three cautions apply to any reading. First, the keys are LLM-built from web and corpus
research and were not checked by a human expert; the dual-key design limits the risk that a key favours one tool,
but both keys can be wrong in the same way. The P-uncontested row (§7.3) shows whether the result depends on the
10 contested questions in P. Second, the 200-word cut removes text from more tool-arm answers (22–24 per arm) than
from the bare model (7); if late content was often correct, the cut penalises the tool arms slightly. Third, all
answers come from one model family and one of the two judges is from the same family; the per-judge sensitivity
rows (§7.5) show how much the result depends on either judge.

---

## 12. Lessons for building LLM-evaluation harnesses

These are the generalisable lessons from building and running this harness. Each one is tied to something that
happened here.

1. **Compare against the alternative users actually have.** For a retrieval tool, that is usually the same model
   with web search, not the bare model. Include both: the bare model is the floor, the web arm is the competitor,
   and "both" measures added value.

2. **Assume every arm-specific mechanism is biased until proven otherwise.** Audit rules, timeouts, retries,
   length limits and citation checkers all interact with tool use, and tool use differs by arm. In this study,
   four separate mechanisms (the character-level audit, the forbidden-word scan, the effort setting and the
   citation rules) each disadvantaged particular arms, and none produced an error message.

3. **Enforce permissions twice, and test the first line.** A declared tool whitelist is a claim about the
   runtime, not a fact. Probe it with a forbidden command in the exact configuration you will run, and keep a
   scripted transcript audit as the second line. Our first line was silently non-exclusive under user settings;
   the audit caught it.

4. **Audit uses, not words.** Scanning transcripts for forbidden strings voids legitimate work ("Goldston"
   contains "gold"). Check what each executed call did: which file it read, which file it wrote, which
   executable it ran, which host it fetched. Parse shell commands as shell tokens.

5. **Voids are draws.** If a void triggers a re-run, a rule that voids normal behaviour gives some arms extra
   attempts. Make the policy explicit before running: how many re-runs, which attempt counts (the first that
   passes, never the best), and what happens after the last void.

6. **Isolation has side effects; pin every setting that matters.** Removing user settings to stop leakage also
   removed a reasoning-effort setting that changed tool use by a factor of three. After isolating, compare
   behavioural statistics (tool calls, turns, tokens) with an earlier run and pin every parameter explicitly.

7. **Shared budgets couple runs.** A per-session web-search budget shared by subagents would have made the web
   arms' results depend on run order. One process per run gives each its own budget and environment.

8. **Control exactly what the judge sees, by allow-list.** Pass judges an explicit allow-list of fields, not a
   record minus some fields. Our judges briefly received a diagnostic field whose content named an arm's tool.

9. **Normalise before judging, and cap length mechanically.** A different-vendor rewrite into atomic claims,
   followed by a leak scan with one re-run and reported residuals, removes most style cues. A length limit in
   the prompt is a request; cut the text in code and report how many answers were cut.

10. **Freeze code, not only data.** Hash the questions, keys, prompts, briefs, snapshot and the harness source
    files into a committed file, and make every later stage verify it — comparing the union of keys, so additions
    count. Exclude files that are legitimately created later (re-run prompts), or the freeze will block the run.

11. **Run the whole pipeline on practice questions before the freeze.** Not one stage, the whole pipeline, in a
    separate study root with its own freeze and tagged logs. Our dry run found twelve changes, at least four of
    them arm-biased. Practice results are not findings.

12. **Get adversarial review of the protocol and of the code, separately.** The protocol reviews found design
    problems (question sets that decide the outcome, no web arm, single key). The code review found problems no
    protocol review could see (a voided batch graded correct, an asymmetric citation rule, a comment character
    that could hide a command).

13. **Turn every "never silently" into code.** Missing grades, missing answers, unchecked citations and
    malformed files must stop aggregation or be scored by an explicit rule and listed. Coverage assertions are
    cheap.

14. **Log everything an outsider would need** — transcripts, command lines, every model call verbatim,
    denials, audit histories. Most bugs above were found by reading these records, not scores.

15. **Write the reporting rules, including the chart and the verdict wording, before the run.** Especially if
    results may be used in marketing. "Inconclusive is not equivalence" has to be in the protocol and in the code
    that prints the verdict.

16. **Amend after the freeze only in code, and prove what changed.** Our post-freeze fixes (a judge audit that
    miscounted an offset read, a citation checker stopped by an external API) were recorded as numbered amendments,
    and each re-written freeze was checked to differ from the old one only in the hash of the file fixed.

17. **Treat usage limits as infrastructure failures that look like results.** A usage limit turned 95 attempts
    into instant failures with no answer. They showed up as zero-cost, sub-second records in the run log; they were
    archived as infrastructure failures, not counted as attempts, and the run resumed unchanged.

18. **Isolation includes the file system.** Fixed paths in a shared `/tmp` for an "empty" working directory and
    configuration could have been pre-filled by another local account. Create a private directory per process, and
    check past runs' records (directory owner, configuration content, the servers a transcript lists) rather than
    assume.

---

## 13. Limitations

- **Small primary set.** n = 60 detects only large differences (about 10–20 points). An inconclusive result is
  likely a priori and is not evidence of equivalence.
- **Questions written by LLM agents.** P was written blind by two independent agents, not by working
  researchers. The owner delegated this; real researcher questions may differ in difficulty and in how much they
  depend on recent or obscure literature.
- **Keys not checked by a human expert.** Both keys are LLM-built from web and corpus research. Dual keys and
  the contested flag reduce the risk that a key favours one arm, not the risk that both are wrong.
- **One model, one family of judges.** All arms are Claude Opus 5.5 at one effort setting; results may not
  transfer to other models or settings. One judge shares the arms' family.
- **One corpus, one retriever.** The corpus covers two journals and mostly 2014–2026 open-access full text;
  retrieval is lexical BM25 with a top-up. A dense retriever, a reranker or a larger corpus could change the
  result; this study does not test them.
- **Agentic search depends on the day.** Web search results change; the web arm's results are not exactly
  reproducible, although its queries and fetched URLs are logged.
- **Normalisation and truncation are imperfect.** Leak flags remain on 1–6 answers per arm; normalisation may
  drop nuance a human would credit; the equal 200-word cut affects more tool-arm answers.
- **Contamination is not measured.** We cannot tell which P or S1-A facts the model saw in training.
- **Acquisition route.** The full texts are CC-licensed, but were downloaded from the publisher's website under an
  arrangement the owner reports the publisher agreed to, with manual clearance of its access check; a replacement
  through the publisher's bulk XML has been requested. This does not affect the measurement but bears on how the
  corpus could be rebuilt by others.
- **Single study, single operator.** One lead session designed, orchestrated and wrote up the study; every step
  after the freeze is scripted, but there is no independent replication.
- **Follow-up studies are small or exploratory.** The meta-analysis study has 6 tasks (the smallest attainable p is
  0.031) and a second-verifier κ of 0.55, below its pre-registered 0.6 flag; its reference values come from a
  model's memory. The drift study has one replicate per arm and one judge, and its comparison between arms is
  exploratory. The figure programme has not passed its real-figure benchmark. The recent-facts study is still
  running.
- **Post-freeze changes.** Two code-only amendments were made to the Q&A pipeline after the arms ran and before any
  result was computed, and a security fix changed two runners' code hashes after their studies were scored (§6.4).
  None changed an answer, key, claim or grade.

---

## 14. Conclusion

On 60 researcher-style questions written blind, the same model with a 1,815-paper corpus and with web search
scored 47 % and 52 % (B − C = −5.0 points, 95 % CI −20.0 to +10.0, p = 0.664): the primary result is inconclusive,
and it is not evidence that the corpus is as good as web search. The meta-analysis study's two co-primary
comparisons are also inconclusive. On a fair, pre-registered test with randomly sampled recent papers, however,
the corpus was clearly better than web search on exact facts (93 % vs 74 %, +19.4 points, p = 1 × 10⁻⁵), and the
gain came from facts inside the paper body and from paper lookup — not from facts any search engine can see in an
abstract, where the two were equal. What the studies also show: every tool arm beat the bare model; the corpus plus web
was the best setup on P (62 %) and contradicted the key least, though not significantly better than web search
alone; the corpus arm's meta-analysis data points were more often verifiable (median precision 0.97 against 0.77),
while the web arm found more of them; and, in exploratory tests, web answers changed between identical runs about
twice as often as corpus answers. The fair test confirms the direction of the by-construction wins on exact facts and lookup, with a smaller
but decisive margin, and points to the mechanism: reliable access to full text that web agents often cannot read. Figure and table data, which neither tool reads today,
remain a possible advantage for an offline corpus, but the extraction does not yet meet its own quality bar. The
harness — isolated processes, native and audited tool whitelists, hashed freezes, dry runs and full provenance — is
the part of this work we expect to transfer most directly.

---

## 15. Reproducibility and data availability

**In version control** (public repository `https://github.com/<owner>/fusion-corpus`; sanitised copy of the private research repository):

- protocols and amendments: `eval/HEAD2HEAD.md`, `eval/METHODS.md`, `eval/SEED_V1.md`, `bench/PREREGISTRATION.md`;
- the two adversarial reviews, verbatim: `eval/h2h/reviews/`;
- the arm brief, tool blocks and all model prompts: `eval/h2h/brief_core.md`, `eval/h2h/tools_*.md`,
  `eval/h2h/prompts/*.md`;
- the P questions and the seed set: `eval/h2h/P_questions*.jsonl`, `eval/seed_v1.jsonl`;
- the freeze file with all hashes: `eval/h2h/FREEZE.json` (commit `b31ea9a`);
- all code (`src/fusion_corpus/`) and tests (`tests/test_h2h_audit.py`, `tests/test_h2h_score.py`);
- the results summary `eval/h2h/RESULTS.md` (commit `c21209a`), full tables `eval/h2h/RESULTS_tables.md` and the
  chart `eval/h2h/chart.png`;
- follow-up studies: `eval/META2HEAD.md`, `eval/meta2h/` (freeze, brief, tool blocks, `RESULTS.md`; frozen in commit
  `fc74e85`); `eval/DRIFT.md` and `eval/DRIFT_RESULTS.md`; `eval/FIGURES.md`; `eval/FACTS.md`.

**Not in version control** (`data/` is git-ignored for size and licence reasons), tied to the committed record by
the hashes in `FREEZE.json`:

- the private question file (sets, sources), merged keys, claims, grades, label maps and citation outcomes
  (`data/h2h/private/`);
- the arm prompts as read and the answer files (`data/h2h/prompts/`, `data/h2h/answers/`);
- headless transcripts, run logs and audit records (`data/provenance/h2h_transcripts/`,
  `data/h2h/headless_runlog.jsonl`, `data/h2h/audit/`);
- the verbatim Gemini call log (`data/provenance/gemini_calls.jsonl`);
- the corpus snapshot (`data/h2h/snapshot/catalog.sqlite`, sha256 `b6f9939d…c2ab`) and the server-side query log;
- the dry-run study root and archived void runs (`data/h2h_dryrun/`, `data/provenance/key2_session_void/`);
- the follow-up studies' data: `data/meta2h/` (snapshot, runs, verifier records, `results.json`),
  `data/h2h_drift/` and `data/h2h_rejudge/` (replicate answers, re-judgings, `drift_results.json`), and
  `data/facts/` (including the usage-limit voids in `data/facts/void_usage_limit/`).

**Hashes.** The snapshot, questions, batches, keys, the 112 arm prompts, the brief and tool blocks, every model
prompt and every harness source file are hashed in `FREEZE.json` (Appendix A). A reader with the data directory
can recompute every hash; `verify_freeze()` does this.

**What can be shared.** The snapshot contains full texts under CC licences, which may be redistributed under
their terms, and abstracts of all catalogued works, which carry their own licence flags and may not all be
redistributable; any release goes through the licence flags (§3.1). Transcripts and answers contain passages
quoted from papers and are subject to the same rule.

**Privacy rule (D30).** The publisher stamps each downloaded PDF's cover page with the downloader's IP address and
the download time. The text layer and index drop the cover page, and the canary fingerprint skips it. **Any export
or sharing of PDFs must strip page 1 when it is such a cover page.** Before any release of logs, transcripts or
derived files, they are to be checked for IP addresses and the owner's local paths. This paper contains no IP
address.

---

## References

Only works the authors are confident exist are listed; claims needing further support are marked
`[CITATION NEEDED]` in the text.

- Asai, A., et al. (2024). OpenScholar: Synthesizing scientific literature with retrieval-augmented LMs. arXiv.
- Cohen, J. (1960). A coefficient of agreement for nominal scales. *Educational and Psychological Measurement*, 20(1).
- Efron, B., and Tibshirani, R. J. (1993). *An Introduction to the Bootstrap*. Chapman and Hall.
- Gao, Y., et al. (2023). Retrieval-augmented generation for large language models: A survey. arXiv.
- Ji, Z., et al. (2023). Survey of hallucination in natural language generation. *ACM Computing Surveys*.
- Kapoor, S., Stroebl, B., Siegel, Z. S., Nadgir, N., and Narayanan, A. (2024). AI agents that matter. arXiv.
- Karpukhin, V., et al. (2020). Dense passage retrieval for open-domain question answering. *EMNLP*.
- Lála, J., et al. (2023). PaperQA: Retrieval-augmented generative agent for scientific research. arXiv.
- Lewis, P., et al. (2020). Retrieval-augmented generation for knowledge-intensive NLP tasks. *NeurIPS*.
- McNemar, Q. (1947). Note on the sampling error of the difference between correlated proportions or
  percentages. *Psychometrika*, 12(2).
- Nakano, R., et al. (2021). WebGPT: Browser-assisted question-answering with human feedback. arXiv.
- Nosek, B. A., Ebersole, C. R., DeHaven, A. C., and Mellor, D. T. (2018). The preregistration revolution. *PNAS*.
- Panickssery, A., Bowman, S. R., and Feng, S. (2024). LLM evaluators recognize and favor their own generations.
  *NeurIPS*.
- Robertson, S., and Zaragoza, H. (2009). The probabilistic relevance framework: BM25 and beyond. *Foundations and
  Trends in Information Retrieval*.
- Sainz, O., et al. (2023). NLP evaluation in trouble: On the need to measure LLM data contamination for each
  benchmark. *Findings of EMNLP*.
- Walters, W. H., and Wilder, E. I. (2023). Fabrication and errors in the bibliographic citations generated by
  ChatGPT. *Scientific Reports*.
- Wilson, E. B. (1927). Probable inference, the law of succession, and statistical inference. *JASA*, 22(158).
- Zheng, L., et al. (2023). Judging LLM-as-a-judge with MT-Bench and Chatbot Arena. *NeurIPS Datasets and
  Benchmarks*.

---

## Appendix A. Frozen prompts and hashes

From `eval/h2h/FREEZE.json` (commit `b31ea9a`). Prompt hashes in the freeze file are the first 16 hex digits of
the sha256; the full digests of the committed files are given for convenience and match those prefixes.

| Prompt | Used by | sha256 |
|---|---|---|
| `key_research.md` | key 1, research step (Gemini 3.8 Flash, Google Search) | `0a951230d63bfe17ea7a9b06a6e3bec1eef5927f8082f826e688c32c72e3c194` |
| `key_web.md` | key 1, key writing (Gemini 3.1 Pro) | `f9548a00f1be15b9826f77cc528af09b08a352f4bf346e9e1184277374a32be8` |
| `key2_claude.md` | key 2 (Claude, web + corpus) | `30c5300fa196b06e5ed99c543355a26fc519ddc83ef92719c84f2b3af18e7bfb` |
| `key_abstract.md` | S1-B keys (Gemini 3.8 Flash) | `73cf1c4581c281e2cf3636cc0ad0044abb49f591b72abdb24de451f3a5079861` |
| `key_compare.md` | contest step (Gemini 3.8 Flash) | `b23deefc3cb0ddbc1c5c9ef83a57fb32ec772fde5a3441e8a5154fffcf3a4319` |
| `normalise.md` | answer normalisation (Gemini 3.8 Flash) | `baf0c7dab02ebf51b4dcfe65ece0a0a1f388a9bae7eff75e000dd04be51cb3a3` |
| `judge.md` | Gemini judge (Gemini 3.1 Pro) | `8d6d3b18f1379be19997efb6732e967a256b7530d4e6b8447467fd1b785da20f` |
| `judge_claude.md` | Claude judge (Claude Opus 5.5) | `7060cba41052d6ebc20c6015d699628b29802eccc8586dabdc4c77a2bbc5c440` |

Other frozen hashes:

| Item | sha256 |
|---|---|
| questions (private) | `7f4dd61cbde3d299dcc9c53402101f8f5cd4b13ce909eb8239622f92df4a148a` |
| batches (public) | `b35bc0c1db46140f3fd1ca360a7a7311f4a25fe55424164e15527cf65e262271` |
| keys (private, keys 1 + 2 merged, contest flags) | `abce8e54cc585a4186d8462934dd21d36fb1252cd67ef287bfe4bf8b3ad4fb85` |
| corpus snapshot | `b6f9939d4ae00b1094b52aa19d3edfac41f24d60b9ac082f4fe338fb70bfc2ab` |
| 112 original arm prompts (concatenated) | `72f35ef2e47e37acf1c1e8b4cd6c7fe4d1a2ee92a61d09d3f0713e33d8ae368a` |
| arm brief + 4 tool blocks (concatenated) | `de991f306df67ae45726e83a8978d73b41aa4eceb08a6218ffc63f53098637b1` |
| `h2h/audit.py` | `ff409db153d5992dd980f228e55e1844964937f60d2d84dc7dc8649f7ff7b4f5` |
| `h2h/gemini_steps.py` | `fb244eb63974788458f59adffd22dc63336b4749010e642c9dbe9929412728f1` |
| `h2h/headless.py` | `4adb31c5d432bc0c38a668f1e3c3b9615dd57242e76f48220d3dd2ef5c0e7c7b` |
| `h2h/questions.py` | `94b32f830fb3dbfffd27ddebad3d3ff32e2222763cfaa08fc988a5783510dfdc` |
| `h2h/run.py` | `1de271e486c72213fcdfef2b46e1b44b5f01860d381f758959c275203b1ba1f7` |
| `h2h/score.py` | `a30baf0f4c49bcaccb644f63db4864a583e436d9bcfdbd4505e25ac4d8c14901` |
| `h2h/stages.py` | `dce4a32fb1c699a40a28f2ed06f28ead685d7458345f806b6c676bea9cecbbe4` |

Seeds: questions 20261004; Gemini-judge labels 20261005; Claude-judge labels 20261006; bootstrap 20261007.

The fixed pointer message given to every arm process (`audit.POINTER`):

> Read exactly one file, {PROMPT_FILE}. It contains your full instructions; follow them exactly (they tell you
> which tools you may use and where to write your single output file). Do not read any other file. When done,
> reply DONE.

## Appendix B. The arm brief, verbatim

`eval/h2h/brief_core.md`. `{TOOLS}` is replaced by the arm's tool block, `{QUESTIONS}` by five
`- [hNNN] question` lines, `{OUTFILE}` by the arm's answer path; `{{`/`}}` become single braces.

```text
You are a research assistant for fusion science. Answer each question below as well as you can.

{TOOLS}

Rules for every question:
- Give a direct answer first, then brief support, in at most 200 words per question.
- For numbers, give the value with its unit and the conditions it applies to.
- Cite your sources for factual claims when you can: authors or collaboration, title, year, and the DOI or URL if
  you know it. Only cite works you are confident exist; never invent a citation, DOI, number or result.
- If you don't know, or the evidence isn't available to you, say so plainly and set "abstain" to true. A wrong
  answer is worse than an honest abstention, and an abstention is worse than a correct answer.
- Answer each question independently.

Questions (answer all of them):
{QUESTIONS}

Output: write exactly one file, {OUTFILE}, with the Write tool (not with a shell command), containing a JSON list
with one object per question:
{{"qid": "...", "answer": "<your answer text>", "value": "<number with unit, if the question asks for a
quantity, else null>", "citations": [{{"authors": "...", "title": "...", "year": 0, "doi": "... or null", "url": "... or null"}}],
"abstain": false, "confidence": 0.0}}
Do not read any other file. After writing the file, reply with only the word DONE.
```

Tool blocks (`eval/h2h/tools_{A,B,C,D}.md`), verbatim; `{CORPUS}` is replaced by the absolute path of the
wrapper.

**A**
```text
Tools: none. Answer from your own knowledge only. Do not use any tool except the single file write required at the end.
```

**B**
```text
Tools: you may search a local library of fusion papers (open-access Nuclear Fusion and Plasma Physics and Controlled Fusion full texts, mostly 2014–2026, plus titles and abstracts of Nuclear Fusion and PPCF papers since 1960) with this shell command and nothing else:
  {CORPUS} search "<query>" [--years Y1 Y2] [--limit N]   # full-text passages
  {CORPUS} passage <chunk_id> [--context 1]               # read a passage with context
  {CORPUS} works "<query>" [--years Y1 Y2]                # titles/abstracts of all catalogued papers
  {CORPUS} paper <work_id>                                # metadata and abstract
Run each command on its own (no pipes, no ';', no redirection). Use at most 15 commands per question. You may also use your own knowledge. Do not use web search or any other tool except the single file write required at the end.
```

**C**
```text
Tools: you may use web search and web fetch (at most 15 calls per question). You may also use your own knowledge. Do not use any other tool except the single file write required at the end.
```

**D**
```text
Tools: you may search a local library of fusion papers (open-access Nuclear Fusion and Plasma Physics and Controlled Fusion full texts, mostly 2014–2026, plus titles and abstracts of Nuclear Fusion and PPCF papers since 1960) with this shell command, and you may use web search and web fetch:
  {CORPUS} search "<query>" [--years Y1 Y2] [--limit N]   # full-text passages
  {CORPUS} passage <chunk_id> [--context 1]               # read a passage with context
  {CORPUS} works "<query>" [--years Y1 Y2]                # titles/abstracts of all catalogued papers
  {CORPUS} paper <work_id>                                # metadata and abstract
Run each corpus command on its own (no pipes, no ';', no redirection). Use at most 15 tool calls per question in total. You may also use your own knowledge. Do not use any other tool except the single file write required at the end.
```
