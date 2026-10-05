# Head-to-head: is the corpus worth it? (pre-registration, v2)

Owner question: does a research assistant with this corpus give better answers than a normal LLM, in particular the
realistic alternative **an LLM with web search**? Results may be used in marketing material, so the design, the
analysis and the chart are fixed here **before any arm runs**, and every result is reported (§9).

v1 adopted every must-fix of review 1; **v2 adopts every must-fix of review 2** (summary in §10). Written by the lead session, which also
orchestrates; every step after the freeze commit is scripted (`src/fusion_corpus/h2h/`).

## 1. Arms

Same model for every arm (Claude Opus 5.5 subagents, fresh `general-purpose` agents, never `fork`), same brief,
same answer format. Only one block of the brief — the tool sentence — differs (`eval/h2h/tools_{A,B,C,D}.md`);
prompt parity is checked by script and the prompts are hashed.

| Arm | Access | Role |
|---|---|---|
| A | own knowledge only | floor ("a normal LLM") |
| B | own knowledge + corpus CLI | the product |
| C | own knowledge + web search/fetch | the realistic alternative |
| D | own knowledge + corpus + web | added value of the corpus on top of web |

Every arm may use its own knowledge and may abstain. Tool calls ≤ 15 per question (B, C, D).

## 2. Question sets (140 questions, opaque ids, shuffled with seed 20261004, 28 batches of 5)

| Set | n | Source | Reference (key) |
|---|---|---|---|
| **P** (primary) | 60 | researcher-style questions written blind by two independent agents that saw no corpus, no other set and no repo file (`eval/h2h/P_questions.jsonl`, `P_questions_2.jsonl`); 20 quantitative, 16 state-of-evidence, 12 recent, 12 "has anyone…" | two independent keys (§4) |
| S1-A | 19 | blind seed v1 Part A in scope (`eval/seed_v1.jsonl`) | two independent keys (§4) |
| S1-B | 25 | blind seed v1 Part B — **"known-paper lookup"** (written from catalogued abstracts; favours B by construction) | the source abstract's claims |
| S1-OOC | 6 | blind seed v1 out-of-corpus | "abstain / say unknown or nonexistent" |
| S2 | 30 | exact facts from the 10 frozen-gold 2026 papers; questions written by an independent agent from the gold item only; 44/54 usable, filled 3 per paper then up to 4 (theory paper aea7b4 has none usable) — **favours B and D by construction (recency)** | the gold value (2 % tolerance) |

**Deviation (declared):** the review asked for questions written by the owner; the owner delegated, so P is written
by an independent agent. Owner questions, if added later, are reported as a separate set.

## 3. Leakage controls

- Arms get a fixed one-line pointer to their own prompt file (`data/h2h/prompts/<arm>/bNN.txt`), which they may
  Read exactly once; the prompt holds only `[opaque id] question` lines. Set, notes, sources, keys and gold live
  in `data/h2h/private/` (never given).
- Corpus arms use a wrapper (`data/h2h/bin/corpus`) on a **frozen read-only snapshot** (1,815 full-text papers;
  sha256 `b6f9939d4ae00b10…`), which refuses to run if the file is missing or its hash differs; eval mode strips
  local paths and logs queries outside the snapshot (`data/h2h/query_log.jsonl`).
- The session's working directory has no MCP config for the corpus; no MCP tool is allowed to any arm.
- **Scripted transcript audit** (`h2h/audit.py`), use-based: one Read of the agent's own prompt file; exactly one
  Write to its own answer file; Bash only as `<wrapper> (search|passage|works|paper|count) …` (no shell
  metacharacters; a trailing `2>&1` allowed); WebSearch/WebFetch only for C/D and only to public http(s) URLs;
  ToolSearch/TodoWrite harmless; anything else (Read of another file, Glob, Grep, Agent, MCP) is a violation.
  The first user message must be the fixed pointer.
- **Voids:** a violating batch is re-run once from a fresh agent (`bNN.rerun1`); a second void scores every
  question of that batch as no answer (incorrect), reported. Malformed answer files are treated the same way;
  a missing question is scored incorrect and listed.
- Arms run interleaved (batch 1 of A, B, C, D, then batch 2 …).

## 4. Keys (built before any arm runs, then frozen)

- P and S1-A: **two independent keys.** Key 1: Gemini 3.8 Flash searches the web (Google Search grounding,
  retried until ≥ 1 source; ungrounded keys flagged), then Gemini 3.1 Pro writes the key from those notes.
  Key 2: Claude agents with web search **and** the corpus write their own key, without seeing key 1. A Gemini
  step marks a question **contested** when the keys conflict on an essential fact. Judges see both keys; where
  they conflict, an answer consistent with either key is acceptable. Rationale: each tool arm has one "friendly"
  key (key 1 web-only, key 2 web + corpus), so neither is privileged.
- S1-B: the source abstract's claims (Gemini 3.8 Flash); keys with no abstract are flagged weak.
- S2: the frozen gold value with its quote. S1-OOC: fixed "should abstain" key.
- `eval/h2h/FREEZE.json` (committed before any arm runs) holds the sha256 of the questions, batches, keys,
  snapshot, arm prompts and Gemini prompts, and the label seeds; normalisation, judging and aggregation refuse
  to run if any of them changed.

## 5. Normalisation and blind judging

- Every answer is rewritten by a fixed Gemini 3.8 Flash step into neutral atomic claims (`normalise.md`). A
  scripted check re-runs it once when the claims still mention sources/tools/web/DOIs or lose the answer's
  value; remaining leaks are counted per arm and reported. Answer length is capped at 200 words by the brief;
  word counts per arm are reported.
- **Two blind judges** with **independent label shuffles** (Gemini seed 20261005, Claude seed 20261006):
  Gemini 3.1 Pro (`judge.md`, one question per call) and Claude Opus subagents (`judge_claude.md`, packets of 10
  questions). Every question must receive exactly one valid grade per label from each judge (validated; retried).
- Correctness = matches a key; "can't verify" is never "incorrect". For keys with answerable = false, an
  abstention counts as correct (applied in code, not only by the judges).

## 6. Citations

Every cited work of every arm is checked the same way (`h2h/score.py`): DOI resolved through doi.org handles (any
registrar: Crossref, DataCite/arXiv, OSTI…), title checked against Crossref; without a DOI, title + year searched
in Crossref and OpenAlex (strict order-insensitive title match). Classes: ok, miscited, fabricated, incomplete,
unverifiable_nonscholarly (a URL that isn't a scholarly record). Reported per answer and per cited work.
Secondary only: it favours B by construction (B copies DOIs from its tool).

## 7. Endpoints and decision rule

- **Primary endpoint (single):** on set P (n = 60), proportion of questions answered **correctly by strict
  consensus** (both judges "correct"; partial = incorrect), **B vs C**, exact two-sided McNemar; the paired
  difference is reported with a bootstrap 95 % CI.
  - **Corpus better than web**: B − C ≥ +10 points and p < 0.05.
  - **Corpus worse than web**: C − B ≥ +10 points and p < 0.05.
  - **Inconclusive** otherwise. Inconclusive is **not** evidence that the two are equivalent and must never be
    presented as "as good as web". With n = 60, p < 0.05 needs roughly ≥ 6–0, 8–1, 10–2 or 13–4 discordant splits.
- Secondary (reported, no decision): D vs C (added value), B vs A, C vs A, D vs A, D vs B, B vs C on every set
  (P, P-uncontested, S1-A, S1-B, S1-OOC, pooled S1, S2); abstention; contradiction rate; fabricated
  citations; tokens and wall time per arm; sensitivity analyses (partial = correct; each judge alone).
- Judge agreement: Cohen's κ and the number of disagreements; disagreements listed for the owner.

## 8. Snapshot

Corpus snapshot at freeze: papers indexed, chunks, catalog sha256 (recorded in the freeze amendment). The arms see
only this snapshot.

## 9. Reporting (fixed now; applies to whatever the results are)

- `data/h2h/results.json` (all metrics) and `eval/h2h/RESULTS.md` (committed) report **every arm on every set**,
  including sets where the corpus loses, with 95 % Wilson intervals and the tests above.
- **The chart is fixed now** (three panels, same axes, 95 % Wilson intervals, n under every bar):
  1. **Primary — set P:** strict-consensus accuracy for A, B, C, D, with the verdict sentence as the panel title.
  2. **Other sets — S1-A, S1-OOC:** same bars.
  3. **"Favours the corpus by construction" — S1-B, S2, and the fabricated-citation rate:** hatched bars with
     the label printed on the panel.
  Any public use shows panel 1 with the verdict sentence.
- No arm, question or set is re-run, dropped or re-graded to change a result. Re-runs happen only for audit
  violations or infrastructure failures, are listed, and the original outputs are kept.

## 10. Review 2 must-fixes and how they were met

Read-only hash-checked snapshot (C1, C2) · use-based audit with fixed pointer, prompt-file read, `2>&1`,
TodoWrite/ToolSearch allowed, public-URL WebFetch only (C3, C4) · pointer prompts, re-run files, second-void
rule, no corpus MCP in the session (C5) · citation checker on doi.org handles + OpenAlex + strict title match +
URL class (H1) · two independent keys + contested flag (H2) · committed FREEZE.json checked by later stages (H3)
· answer/judge validation, Claude-judge merge, explicit no-answer records (H4, H5) · every pre-registered
analysis and the verdict in code (H6) · abstain on unanswerable = correct in code (H7) · three-panel chart (H8)
· inconclusive ≠ equivalence, P raised to 60 (M1) · normalisation leak check (M2) · word counts (M3) ·
independent judge shuffles (M4) · runbook in code (M5) · coverage assertions (M6) · S2 value assertion (M7) ·
pilot of every arm on 5 practice questions outside the test set before the freeze.

## 11. Pilot (before the freeze; practice questions x001–x005, not in the test set)

| Arm | Tool calls | Audit | Note |
|---|---|---|---|
| A | Read 1, Write 1 | ok | abstained on the fictional 2031 question |
| B (1st) | Read 1, Bash 13 | **void** | wrote its answer file with a shell heredoc followed by a JSON-validation one-liner |
| B (2nd) | Read 1, Bash 10, Write 1 | ok | after the brief added "with the Write tool (not with a shell command)" |
| C | Read 1, WebSearch 2, WebFetch 1, Write 1 | ok | |
| D | Read 1, Bash 13, Write 1 | ok | |
Change made after the pilot (applies to every arm identically): the output instruction in `brief_core.md`. The audit
also accepts a pure `cat > <own answer file> <<'EOF' … EOF` command as the write; anything chained to it remains
a violation. Pilot transcripts: `data/provenance/h2h_pilot/`.

## 12. Product change before the freeze (applies to arms B and D)

The second-key agents reported that the corpus search "returned nothing for longer queries": passage and title
search required every term to match (FTS5 implicit AND), so natural-language queries usually failed. Before the
freeze, search was changed to run the all-terms query first and, when it returns fewer than the requested number
of hits, top up with an any-term query (stop-words removed), BM25-ranked; each hit is labelled `match: all|any`.
This is a generic retrieval fix, not tuned to any test question; it is part of the system under test.

## 13. Execution change before the freeze: headless processes instead of subagents

The second-key subagents exhausted the **session-wide web-search budget** (200 searches per Claude Code session,
shared by every subagent); a check confirmed WebSearch was refused afterwards. Running the arms as subagents would
have left C and D without web search. Therefore every second-key batch and every arm batch runs as its **own
headless process** (`claude -p`, `src/fusion_corpus/h2h/headless.py`):
- own web budget; empty working directory (`/private/tmp/h2h-empty`); `--strict-mcp-config` with no servers;
- tools enforced natively: `--tools` lists the arm's tools; `--allowedTools` permits Read of its own prompt file,
  Write of its own answer file, Bash only as the corpus wrapper (`Bash(<wrapper>:*)`), and WebSearch/WebFetch for
  C/D; anything else is denied by Claude Code (denials are recorded) and still audited from the stream-json
  transcript;
- the user-level Claude Code settings, memory file and hooks still load (no API key for `--bare`); they load
  identically for every arm and are declared here;
- each run's duration, token usage and notional cost come from the CLI's result record.
The 8 second-key batches written by subagents are void (3 completed, 2 of them hit the search limit; 5 stopped);
their transcripts and outputs are archived (`data/provenance/key2_session_void/`,
`data/h2h/private/keys2_void_session/`) and all 8 are re-run headless. Headless pilots of arms A and B passed the
audit (tools `[Read, Write]` and `[Bash, Read, Write]`; 14/14 Bash calls were corpus commands).

## 14. Correction to §13: tool restriction was not exclusive (found before any arm ran)

The second-key audit flagged non-wrapper Bash commands in batches k02 (a malformed `cd`-less path chained before a
corpus search) and k03 (`sed -i` on its own output file, then a `python3` read of it) with **no permission
denial recorded**. Probe (Haiku, empty dir, 2026-10-02): with the owner's user settings loaded (`defaultMode:
auto`, allow rules, a command-rewriting hook), `--allowedTools` is not exclusive — `ls /` ran, and `cat` was
rewritten by the hook. With `--setting-sources project --permission-mode dontAsk --disable-slash-commands`,
`echo` (whitelisted) ran and `ls /`, `cat /etc/hosts`, `touch` were all denied, and the model saw no user
CLAUDE.md content. The §13 sentence "anything else is denied by Claude Code" was therefore false for key 2 and
the pilots; the audit (use-based, from transcripts) was the only enforcement and caught both cases.

Changes (before the freeze, before any arm batch): every arm and judge process runs with those three flags, so
the whitelist is enforced natively, no user hooks or memory load, and the audit remains the second line.
Key 2 decisions: k02 and k03 are **kept** — neither command reached information outside the allowed sources
(a failing path; edits/reads of the agent's own output file) — and flagged `audit_note` in the run log. Key 2
was produced under the owner's user settings (declared); it is a reference key, not a measured arm.
A hardened pilot of arms B and D (practice questions) is run before the freeze; transcripts in
`data/provenance/h2h_pilot_hardened/`.

## 15. Full dry run of the pipeline and the changes it caused (before the freeze)

The owner asked for a full, actively monitored runner test with no bias. The whole pipeline (keys 1 and 2,
contest, freeze, 4 arms × 2 batches, validation, normalisation, both judges, citations, aggregation) is run on 6
practice questions (x001–x005 researcher-style, x006 out-of-corpus) in a separate directory
(`FUSION_H2H_ROOT=data/h2h_dryrun`; own freeze file; Gemini calls tagged `study_root`). No test question is used.
Changes made from it, all before the freeze and identical for every arm:
1. **File-write permission.** Under `dontAsk`, writes are governed by `Edit(//path)` rules, not `Write(...)`: the
   hardened pilot's B and D could not write their answers. Rule changed; probe confirmed only that one file is
   writable.
2. **Audit: rewrites of the own answer file are allowed** (the B pilot rewrote its file once to fix it; no
   information is gained by that). A write anywhere else is still a violation; zero writes voids the batch.
   Calls natively denied by Claude Code are excluded from the use-based audit (never executed) and kept in the
   transcript.
3. **Word cap enforced uniformly:** every answer is cut to 200 words (the brief's limit) before normalisation; the
   original length and a `truncated` flag are recorded and reported per arm. Prevents a length advantage with the
   judges.
4. **Judges see only `abstained`, `value`, `claims`.** Before, the claims record (including `leak_problems`, which
   can contain e.g. "leak: 'web'") was passed to the judges — an arm-identifying leak. Fixed for both judges.
5. **Leak scan extended** to "et al", arXiv, journal abbreviations (Nucl. Fusion, PPCF, Phys. Plasmas) and URLs.
6. **Claude judge implemented** as headless processes (tools Read + Write only, isolated settings): its packet is
   inlined in its single prompt file (`judge_claude.md` reworded accordingly), output in
   `private/judge_claude/jNN.json`, audited like an arm with no tools.
7. **Validation reads the counted attempt** (a passing re-run), not always the first file; aggregation includes the
   headless run log.
8. **Effort pinned to `xhigh`** (`--effort xhigh`) for arms, key 2 and the Claude judge. Isolation (§14) had also
   dropped the owner's `effortLevel: xhigh`, which key 2 and the earlier pilots had inherited; the first dry-run
   arms then ran at the default effort and used about a third of the tool calls (B: 4–5 corpus calls per 3
   questions vs 14–16 per 5). Those dry-run outputs are archived in `data/h2h_dryrun/void_default_effort/`.
9. **Corpus-command audit judged on shell tokens, not characters** (bias against B and D, found in the dry run):
   the old character blacklist voided a B batch twice because a quoted query contained `Q>1`; Claude Code had
   (correctly) allowed it. Now: `shlex` tokens; rejected are unquoted operators, `$`/backticks anywhere, newlines,
   other executables (12 attack forms tested, `tests/test_h2h_audit.py`). `headless reaudit` re-applies the
   current audit to stored transcripts, keeping the previous verdict in the record.
10. **Freeze hashes the original batch prompts only.** Re-run prompts (`bNN.rerunK.txt`) were included, so any
    re-run would have broken `verify_freeze` and blocked every later stage.
11. **Judge labels normalised.** Gemini echoed the heading as the label ("Answer W") on 3 of 6 questions; labels
    are mapped to the letter (raw label kept), each judgement must cover exactly the expected labels (one re-call,
    then `label_problem` flagged); the coverage assertion in aggregation stopped scoring until fixed.
12. **Title matching is punctuation-blind.** Curly vs straight quotes put a real paper (cited by all four arms) at
    87.6 < 90 → "miscited". Fixed with a test; citations re-checked (v1 kept as `citations_v1_quote_bug.json`).
Dry-run outcome after the fixes: all stages ran; 8/8 arm batches pass the audit; 0 normalisation leaks; both judges
complete; kappa 0.45 on 24 gradings; verdict "inconclusive" as expected at n = 5. Practice results carry no
evidential weight and are not reported as findings.
Arms within a batch run concurrently (A, B, C, D interleaved), same model, same limits; void rates are reported per
arm. All arms and both judges are Claude Opus 5.5 / Gemini, so a judge's self-preference applies to every arm
equally; the second judge is from another vendor.

## 16. Adversarial code review of §14–15 and the fixes (before the freeze)

An independent reviewer (no edit rights, no model calls) checked the staged changes for arm bias, identity leaks,
forbidden access, silent data loss and real-run breakage. No critical issue; fixed:
- **HIGH: a twice-voided batch could be graded correct** (the empty "no answer" record went to Gemini; "abstained"
  is correct on out-of-corpus questions). Now no-answer records bypass normalisation with a fixed record and are
  scored incorrect by rule whatever a judge says; counted per arm (`no_answer`).
- **HIGH: citation checker asymmetries.** A doi.org URL is now treated as a DOI; a real title with a DOI that
  resolves nowhere is `miscited` (was `ok` via title search); every title hit is considered (a first hit with an
  off year no longer hides a matching one); an unmatched scholarly title is `fabricated` with or without a URL —
  only a non-DOI web page is `unverifiable_nonscholarly`. Same rule for every arm; tests with a fake resolver.
- **MEDIUM:** `shlex` comment stripping (a mid-word `#` could hide `; cmd`) disabled; heredoc writes must use a
  quoted tag and a single body; more private URL ranges blocked; a timeout is a counted failed attempt (it used to
  crash the runner and allow unlimited retries); the answer file is deleted before each attempt (no stale pass);
  arms and the Claude judge call `verify_freeze` before starting; the freeze also hashes the h2h code, the brief
  and the tool blocks, and `verify_freeze` now compares the union of keys (a probe edit to `score.py` was
  refused); `reaudit` keeps the full verdict history; truncated answers are counted per arm.
- **LOW:** label checks no longer crash on a null label.
Checked and sound per the reviewer: wrapper path / `2>&1` handling, `Read(//…)`/`Edit(//…)` rules (private keys
unreachable), isolation flags, judge view, label regex and shuffles, freeze glob, punctuation-blind titles,
first-passing-attempt selection (no best-of-two).

## 17. Post-freeze amendment 1 (2026-10-03, after the arms and judges ran, before any result was computed)

Two infrastructure bugs surfaced in stages after the freeze; both are fixed in frozen code, so the freeze was
re-written. Verified before re-freezing: the **only** changed key is `code_sha256`, and only for `audit.py` and
`score.py` (questions, batches, keys, snapshot, prompts, briefs, seeds unchanged). No answer, key, claim or grade
was changed or re-generated.
1. **Claude judge audit.** The real packets (10 items × 4 claim lists + both keys) exceed one Read page, so every
   judge read its prompt file once and then once more with an offset to get the rest; the audit counted two reads
   and flagged all 14 judge runs. Transcripts verified: per run exactly Read(own prompt) + Read(own prompt,
   offset) + Write(own output), no other tool, no denial. The audit now treats an offset read of the same prompt
   file as part of the one read (test added); re-audit of the 14 judges: all pass
   (`data/h2h/judge_reaudit.json`, previous verdicts kept). The grades used are the ones written by those runs.
   Arms were unaffected (their prompts fit one page; 0 voids).
2. **Citation check crash.** OpenAlex returns HTTP 400 for a search string containing `?`; the frozen checker
   (which never skips a citation silently) stopped at 116/560 answers. The OpenAlex query now drops punctuation
   (the comparison itself is unchanged and already punctuation-blind). Same rule for every arm; the 116 answers
   already checked had no failing query, so they are not re-checked.
Also recorded: the Gemini judge hit the project's daily request quota for `gemini-3.1-pro` (250/day) at 134/140
questions; the frozen judge model is not substituted — the remaining 6 are graded after the quota resets.
Earlier in the normalisation stage two Gemini responses were unusable (a degenerate repetition to the token
limit; an empty response); the resumable stage was re-run, which re-issued exactly those calls (no record had
been written). Every resume is in `data/h2h/score_chain.log`.
3. **Amendment 2 (same day): OpenAlex credentials in the citation check.** The checker sent no contact email or
   key, so OpenAlex counted it against the free per-IP daily budget, which ran out at 120/560 answers (HTTP 429).
   It now sends the contact email and, when configured, the project's `OPENALEX_API_KEY`. A lookup credential;
   it cannot change any outcome. Freeze re-written; only `code_sha256` of `score.py` changed (verified). The stage
   resumes where it stopped.
