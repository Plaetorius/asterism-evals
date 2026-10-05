# Methods record (for the paper)

Working title: *Does a curated literature corpus make LLM research assistants better? A pre-registered, audited
head-to-head in fusion science.*

This file records **how everything was done and why**, in order, with pointers to the committed protocol,
amendments, code and raw data. It is written as the study runs; nothing is edited silently (git history is the
audit trail). Results go in `eval/h2h/RESULTS.md` and `bench/RESULTS_bench-v2.md`.

## 1. System under test

- **Corpus** (`src/fusion_corpus/`): Nuclear Fusion + PPCF. Metadata for 24,101 works (Crossref + OpenAlex, 1960–2026;
  licence per content version). Full text of CC-on-version-of-record papers fetched with a validated harness
  (13 failure classes, canary, health checks), stored as 150 dpi renditions with identical text (D28/D29).
- **Text layer:** PyMuPDF in a sandboxed child process; cleaning (invisible text, running heads, vocabulary-aware
  dehyphenation, reference split, sub/superscript marking). Chunks ≈ 400 tokens, section-aware.
- **Retrieval:** SQLite FTS5/BM25 over symbol-normalised text (τ_E ≡ tau_E ≡ tauE); all-terms query topped up by an
  any-term query (§13 of HEAD2HEAD). Quote anchors re-located across text-layer versions.
- **Access:** MCP server / CLI (`search`, `passage`, `paper`, `works`, `count`), DOI and local-PDF links.

## 2. Extraction benchmark (which LLM to use for structured extraction)

Protocol: `bench/PREREGISTRATION.md` (+ amendments 1–4), result `bench/RESULTS_bench-v2.md`.
1. **Gold:** 10 papers (NF 66(12), stratified by paper type); 2 independent Claude annotators per paper following
   `bench/ANNOTATION_GUIDE.md`; inter-annotator F1 (devices ≈ 1.0, quantities 0.77–0.93, open problems 0.53–1.0);
   disagreements turned into guide v1.1 rulings; adjudication by a third session; 30-item spot check (delegated to
   the lead session by the owner — recorded deviation); freeze by sha256 before any model call.
2. **Arms:** 5 Gemini models × {text, native PDF}.
3. **Amendment 2:** single-call extraction under-extracts (recall 0.07–0.28, precision 0.85–1.0) → 3-page windows.
4. **Amendment 3:** the Batch API silently ignored `response_json_schema` (probe-confirmed; `response_schema`
   works) → 90 void records kept, Stage B re-run.
5. **Amendment 4:** run-health gates (preflight, between-sample circuit breaker, per-arm health in scores).
6. **Result:** `gemini-3.8-flash`, text input, 3-page windows: quantity F1 0.56, open-problem F1 0.71, quote
   validity 0.98; spend $10.25.

## 3. Head-to-head (is the corpus worth it?)

Protocol: `eval/HEAD2HEAD.md` (v0 → v1 → v2, sections 10–13 record every change and why).
- **Design evolution:** v0 → adversarial review 1 (questions decided the outcome; no web arm; leaks; judge bias)
  → v1 → adversarial review 2 (snapshot written to; "gold" substring voided Goldston searches; audit too brittle;
  citation checker biased against web; keys not frozen) → v2. Both reviews: `eval/h2h/reviews/` (verbatim, see §5).
- **Arms:** A model only · B + corpus · C + web · D + corpus + web; same model (Claude Opus 5.5), same brief except
  one tools block (parity checked, hashed).
- **Questions (140):** P = 60 blind researcher-style questions (primary) · S1 = 50 pre-registered blind seed
  questions (A in-scope 19, B lookup 25, out-of-corpus 6) · S2 = 30 exact facts from the frozen 2026 gold.
- **Keys:** key 1 — Gemini 3.8 Flash web research (grounding required) → Gemini 3.1 Pro writes the key; key 2 —
  independent Claude processes with web + corpus; contested flag when they conflict on an essential fact.
- **Grading:** Gemini normaliser → neutral claim lists (leak-checked) → two blind judges (Gemini 3.1 Pro, Claude
  Opus) with independent label shuffles → strict consensus. Citations checked on doi.org/Crossref/OpenAlex.
- **Primary endpoint:** B vs C on P, strict-consensus accuracy, exact McNemar; verdict better / worse /
  inconclusive (inconclusive ≠ equivalence). Chart fixed before the run.
- **Execution:** arms and key 2 run as separate headless `claude -p` processes (own web budget, native tool
  restrictions, empty working directory) after the session-wide search budget was exhausted (§13).
- **Pilot** on 5 practice questions outside the test set: A, C, D clean; B wrote via shell heredoc → brief fixed;
  re-run clean (§11).

## 4. Things that went wrong and were caught (for the "lessons" section)

| When | What | How caught | Consequence |
|---|---|---|---|
| download | IOP stamps each PDF with the downloader's IP and time → canary hash and then text fingerprint mismatched | canary stop | fingerprint excludes the cover page; IP-stripping rule for exports |
| indexing | MuPDF messages on stdout broke the JSON channel of the sandboxed extractor | crash | messages silenced; failures counted |
| benchmark | one call per paper → models summarise (recall ≤ 0.28) | pilot | page windows (amendment 2) |
| benchmark | Batch API ignored `response_json_schema` | collapsed scores, probe job | `response_schema`, void run kept ($3.39) |
| benchmark | offset units (°C) crashed the scorer | crash | fixed with test |
| head-to-head | the protocol's own question sets would have decided the outcome | review 1 | blind P set, web arm, dual keys/judges |
| head-to-head | "gold" in a forbidden-path regex would void Goldston searches; snapshot not read-only | review 2 | use-based audit; read-only hash-checked snapshot |
| head-to-head | Gemini Pro skipped web search at high thinking | `grounded` check | two-stage keys (Flash research → Pro key) |
| head-to-head | corpus search returned nothing for long queries | key-2 agents' notes | any-term top-up (§12) |
| head-to-head | session-wide 200 web-search budget exhausted by subagents | key-2 agent note, verified | headless processes (§13) |
| head-to-head | headless tool whitelist not exclusive under the owner's user settings (auto mode, hooks) | key-2 audit: violations with no denial; probe | `--setting-sources project --permission-mode dontAsk` (§14) |
| head-to-head | corpus arm wrote its answer through a shell heredoc | pilot audit | brief requires the Write tool |
| head-to-head | headless runs had dropped the owner's effort setting → arms used ~⅓ of the tool calls | dry run: tool counts | `--effort xhigh` pinned (§15) |
| head-to-head | audit voided a corpus query containing a quoted `Q>1` (bias against B/D) | dry run | shell-token audit + tests (§15) |
| head-to-head | re-run prompts inside the freeze hash would block all later stages | dry run | freeze original prompts only (§15) |
| head-to-head | judges were sent `leak_problems` (could reveal "web") | dry-run code review | judges see claims only (§15) |
| head-to-head | Gemini judge labels "Answer W" → grades unmatched | coverage assertion | label normalisation + exact-coverage check (§15) |
| head-to-head | curly vs straight quotes → real paper "miscited" in all arms | dry-run spot check | punctuation-blind title match (§15) |
| head-to-head | void batches could score as correct; citation rules differed by URL; audit holes (#, heredoc); freeze ignored new keys | adversarial code review (§16) | fixed with tests |
| head-to-head (post-freeze) | judge prompt read in two pages → all judges flagged; OpenAlex 400 on "?" stopped the citation check; Gemini Pro daily quota (250) hit at 134/140; two unusable normaliser responses | audit, crash, 429 | amendment 1 (§17): code-only, freeze re-written, only code hash changed |
| meta-analysis | verifier could see arm-identifying fields; failed verifier batches would have been scored; 12 pseudo-replicated pairs | adversarial review | blinded fields, audited retries, task-level analysis (META2HEAD §8) |
| meta-analysis | verifier inconsistent on comparison values the task requested | pilot | relevance rule (META2HEAD §9) |
| meta-analysis | second-verifier kappa 0.55 < pre-registered 0.6 flag | scoring | reported as a limitation next to the primary |
| data | download loop re-selected 8 licence-refused papers forever (blocked the index rebuild) | status check | loop skips `refused_licence.txt` |
| facts study | Claude session limit (HTTP 429) failed 95 attempts in < 1 s mid-run | instant $0 voids in the run log | chain stopped; limit-caused records archived as infrastructure failures; resumed after reset (FACTS §9) |

## 5. Data and provenance inventory (everything kept)

| What | Where |
|---|---|
| Protocols, amendments, prompts, briefs, guides | `eval/`, `bench/` (git) |
| Questions (private: sets, sources), keys, claims, grades, citations | `data/h2h/private/` |
| Arm prompts (exact text each agent read) and answers | `data/h2h/prompts/`, `data/h2h/answers/` |
| Every Gemini call: prompt, settings, raw response, usage, search queries and sources | `data/provenance/gemini_calls.jsonl` |
| Every headless agent run: full stream-json transcript, command line, duration, usage, cost, denials, audit | `data/provenance/h2h_transcripts/`, `data/h2h/headless_runlog.jsonl`, `data/h2h/audit/` |
| Every subagent transcript (annotators, adjudicators, reviewers, question writers, voided key-2 runs, pilots) | `data/provenance/agent_transcripts/` (+ `INDEX.json`), `data/provenance/key2_session_void/`, `data/provenance/h2h_pilot/` |
| Benchmark records (every Gemini extraction call with response text and usage) | `data/bench/bench-v1/`, `bench-v2/`, `bench-v2-void-batch/` |
| Corpus snapshot used by the arms (read-only, sha256 recorded) | `data/h2h/snapshot/catalog.sqlite` |
| Server-side query log of the corpus arms | `data/h2h/query_log.jsonl` |
`data/` is git-ignored (size, licences); the freeze file and protocols commit the hashes that tie results to it.

## 6. Meta-analysis head-to-head (evidence gathering)

Protocol `eval/META2HEAD.md`; results `eval/meta2h/RESULTS.md`.
- **Task selection blind to the corpus:** a headless Claude process with no tools except writing one file proposed
  10 candidate meta-analysis tasks under written criteria; a mechanical rule (id order, reference confidence high or
  medium, ≤ 2 per topic) selected 6 test tasks and 1 practice task. Prompt and transcript kept.
- **Arms and execution:** as the Q&A study (A–D, Claude Opus 5.5, xhigh effort, isolated headless processes,
  use-based audit), against a new read-only snapshot of the full corpus (3,494 papers). 6 tasks × 4 arms × 2
  replicates = 48 runs; each run returns ≤ 40 data points with verbatim quotes and a synthesis.
- **Scoring without a judge model from another vendor (owner constraint: Gemini rate-limited):** all points of a task
  pooled, deduplicated (DOI found anywhere, arXiv ids, device aliases, 2 significant figures), grouped by source
  work and shuffled; the verifier sees only value, unit, device, conditions, quote and authors/title/year/DOI, and
  has both the corpus and the web to read sources. 25 % re-verified by an independent verifier.
- **Analysis:** task level (6 pairs; replicates are not independent), exact Wilcoxon, B vs C and D vs C co-primary
  with Holm; secondary and sensitivity analyses pre-registered.
- **Review and pilot:** adversarial review before the freeze (3 critical, 5 high findings — all fixed, §8 of the
  protocol); pilot on a practice task led to one verifier-rule change (requested comparison values count as
  relevant).

## 7. Drift study and figure-extraction programme

- Drift (`eval/DRIFT.md`): the 112 Q&A batches re-run with byte-identical prompts; Claude judge on the replicates
  and a second judging of the originals to separate answer drift from judge noise. Result (`eval/DRIFT_RESULTS.md`):
  correctness flipped between identical runs on 22.1 % of questions for the web arm vs 12.1 % for corpus and for
  corpus + web and 10.7 % for the model alone (judge noise 1–6 %); exploratory paired tests: web vs corpus p = 0.029,
  vs corpus + web p = 0.024.
- Figures/tables (`eval/FIGURES.md`): offline extraction by a cost cascade (data files → tables → vector plots →
  captions → vision on raster crops, model reports pixels only), synthetic and real quality benchmarks with
  pre-registered acceptance thresholds, cost controls (dry runs, caps, preflight probes, circuit breakers,
  canaries, content-hash cache), then a fifth arm on the frozen meta-analysis tasks. Progress is logged in §9 of
  that protocol: free census of every figure/table; a 300-plot synthetic benchmark with exact truth on which the
  free vector extractor reached precision 1.000, recall 0.994, median error 0.21 % of the axis range, with four
  bugs caught by the benchmark before any real figure was touched. On real figures (full census: 43,932 figures,
  20,765 plot candidates, 78 % of vector plots with outlined tick labels) the free route currently yields points
  for 18/300 sampled figures with precision below the pre-registered bar — recorded as a negative interim result;
  no real values are used until the real-figure benchmark passes.

## 8. Security fix after the runs (2026-10-03)

A commit security review flagged that the headless runners used fixed paths in the shared `/tmp` for the empty
working directory and the empty MCP-server config. Another local account could have pre-created them with an MCP
config or project settings that the isolated runs would load (config injection). Verified for every past run: the
directories and the config were owned by the owner account, empty, the config content exactly `{"mcpServers": {}}`,
and every run transcript's init record lists no MCP servers — no run was affected. All three runners (h2h, meta2h,
facts) now create a fresh private directory per process (`mkdtemp`, mode 0700) for both. This changes the code
hashes of `h2h/headless.py` and `meta2h/run.py` after those studies were completed and scored; their recorded
results are unaffected (the change touches only where the empty directory and config live).

## 9. Recent-facts study (fair test of the by-construction advantage)

Protocol `eval/FACTS.md`, results `eval/facts/RESULTS.md`. Random sample of 2025–26 NF/PPCF papers (not chosen from
the corpus), tool-less blind question writer with paraphrase rules, automatic item validation (quote in source,
value next to a compatible unit, self-grading, leak and copy checks), code grader (no judge model), paper-clustered
primary test. Adversarial review blocked the first version (DOI parsing biased against web citations, clustering,
lenient grading, copied wording) — all fixed before the freeze; real-item validation exposed a dozen parser bugs,
fixed with tests before the freeze. **Result: corpus +19.4 points over web on exact facts (p = 1 × 10⁻⁵), driven by
facts in the paper body (+29) and paper lookup (+15); no difference on abstract facts.** Mechanism measured in the
web arm's transcripts: 27 % of its publisher fetches hit a bot challenge. Grader checks: 74/75 random gradings
agree; 19 false negatives found among answers containing the key number, mostly against the corpus arms.


## 10. Table facts (eval/TABLE_FACTS.md, results in eval/tabfacts/RESULTS.md)
- Table layer: 3,893/4,429 captioned tables reconstructed from the PDF layout (rules, white channels, script-aware).
- Study: a blind writer keyed each question from the rendered table *image*; the key was validated inside the PDF table
  region. 69 items, 5 arms; adversarial review before the freeze (4 high findings fixed, including a grader false
  positive on dropped divisors).
- Result: text corpus 100 %, corpus + tables 99 %, web 57 %, model alone 3 %. E vs C +42 pts (p = 3.7e-9); E vs B
  at the ceiling (inconclusive).
- The web gap persists on items where the web fetched the paper (+50 on 16).
- Grader false negatives: 4/6 wrong answers, verdicts unchanged.


## 11. Post-cutoff study (eval/POSTCUT.md, results in eval/POSTCUT_RESULTS.md)
- **Frame and sample.** Papers whose earliest public date (Crossref DOI registration, online and print dates, and
  the first version of an arXiv preprint) is on or after 2026-06-01. 22 % of the issue-dated "post-cutoff" papers
  had been public earlier and were excluded. The sample was 100 papers, 185 items, coverage-inclusive.
- **Review.** An adversarial review before the freeze found 1 critical issue (the frame dates) and 5 high, all
  fixed. The freeze hashes exactly the code the study and its wrapper process load.
- **Result.** Corpus 98 % against web 48 % on facts (+49.2, p = 5 × 10⁻⁶); body +66, table +75, abstract +14,
  lookup +23. When the web reached the value, it was equal to the corpus (17/17).
- **Grader checks.** 36/37 random grades agreed with a human reading. 7 grader false negatives were found ("R² = x"
  read as 2, among others), spread across arms; verdicts unchanged.
