# Meta-analysis head-to-head (pre-registration, v0)

Owner question (2026-10-03): is the corpus a better tool than the web for **literature meta-analysis** — finding
the published measurements of a quantity and synthesising them? The Q&A head-to-head (`HEAD2HEAD.md`) measured
answering; this measures evidence gathering. Owner constraint: Gemini is rate-limited, so **every step uses Claude**
(Opus 5.5) and the scoring is designed to be mostly objective (is each data point in its cited source?) rather than
judged. Written before any task is run; changes after the freeze are numbered amendments.

## 1. Arms (same as HEAD2HEAD §1)

A model only · B + corpus · C + web (WebSearch, WebFetch) · D + corpus + web. Same model (`claude-opus-5-5`,
`--effort xhigh`), same brief except the tools block, same isolation as the Q&A study (headless `claude -p`, empty
working directory, `--setting-sources project --permission-mode dontAsk`, per-file Read/Edit rules, corpus wrapper
only, use-based transcript audit). Corpus: a new read-only, sha256-pinned snapshot of the full corpus at this date
(3,494 full-text papers, 165,953 passages; metadata for all catalogued works): `data/meta2h/snapshot`, wrapper
`data/meta2h/bin/corpus`.

## 2. Tasks — chosen blind to the corpus

- A separate headless Claude process with **no tools except writing one file** (it cannot see the corpus or the
  web) proposed 10 candidate meta-analysis tasks under written criteria (quantity with units, inclusion criteria,
  ≥ 8 published data points from ≥ 3 devices/groups, numbers normally in text/tables, a published synthesis as
  reference, topic spread ≤ 2 per topic, no publisher/source optimisation). Prompt and transcript:
  `data/meta2h/private/taskgen_*`.
- **Mechanical selection rule (fixed now):** in id order, keep candidates whose `reference_confidence` is high or
  medium, respecting ≤ 2 per topic, until **6 test tasks**; the next eligible candidate is the **practice task**
  (pilot only). No candidate is inspected against the corpus before selection; corpus coverage per task is
  measured afterwards and reported.

## 3. What each arm produces

Per run, one JSON file: `data_points` (≤ 40), each {`value` or `value_low`/`value_high`, `unit`, `device`,
`conditions`, `source` {`authors`, `title`, `year`, `doi`, `url`}, `quote` (verbatim sentence or table cell that
states the value), `location` (page/table/section)}; and `synthesis` {`method`, `estimate` (pooled value or trend
parameters), `uncertainty`, `n_points_used`, `notes`}. Budget per run: ≤ 150 tool calls, ≤ 60 min. Each
arm × task is run **twice** (independent replicates) → 6 × 4 × 2 = **48 runs**, interleaved A, B, C, D per task.

## 4. Verification (Claude, blind to arm)

- Per task, all data points of the 8 runs are pooled, exact duplicates (same DOI/URL + value + device) merged with
  provenance kept, DOI links normalised to bare DOIs, order shuffled (seed 20261008); the verifier never sees
  which run or arm a point came from.
- Verifier: headless Claude with the corpus wrapper and web access (both, so any source can be looked up the same
  way), in batches of ≤ 25 points. For each point: `supported` (the source states this value, unit and device,
  within rounding), `partial` (value found but device/conditions/units differ), `unsupported` (source exists but
  does not state it), `source_not_found` (no such work), `inaccessible` (work exists but its text could not be
  read); and `relevant` (meets the task's inclusion criteria: yes/no).
- **Second verifier:** an independent verifier run re-checks a random 25 % of points (seed 20261009); agreement
  (Cohen's kappa) is reported.

## 5. Metrics

Per run: **V = distinct relevant supported data points** (distinct by source work + device + value); claimed
points; precision = supported ÷ claimed; distinct source works; fabricated-source rate (`source_not_found`);
inaccessible rate; tool calls, wall time, cost.
Per task: the **union** of distinct relevant supported points over all runs = pooled evidence; each run's
**recall** = V ÷ union. Estimate check: the run's pooled estimate vs the reference value/range when the task's
reference confidence is high/medium (inside range or within ± 20 % of a point value).

## 6. Pre-registered analysis (v1, after review 1)

- **Unit = task** (the two replicates of a task share its difficulty, so the 12 runs are not independent): each
  arm's V is summed over the two replicates of a task → 6 paired tasks.
- **Co-primary:** B vs C (corpus vs web) and D vs C (corpus + web vs web, the product question), each an exact
  two-sided Wilcoxon signed-rank test over the 6 tasks, **Holm** correction at α = 0.05. Verdict per comparison:
  better / worse when Holm-significant with the median task difference in that direction, else "inconclusive (not
  evidence of equivalence)". With 6 tasks the smallest attainable p is 0.031: a significant result needs every
  task to agree. Power is low; effect sizes (totals, ratio, per-task differences) are reported in full.
- **Secondary:** run-level pairs (12, pseudo-replicated, descriptive); all other arm pairs; **distinct verified
  source works** per arm (quality, not volume); precision; recall vs the union; fabricated-source and inaccessible
  rates per arm; tool calls, time, cost; the pooled estimate vs the reference (descriptive — the references come
  from the task generator's memory and are not ground truth).
- **Sensitivity (accessibility):** V recomputed counting `inaccessible` relevant points as verified (an upper
  bound that removes any advantage of open-access corpus sources), with the same tests.
- Second verifier: if kappa < 0.6 it is flagged in the results next to the primary.
- Every run, task and point is reported; nothing is re-run or dropped to change a result.

## 7. Freeze and pilot

Pilot on the practice task (all 4 arms × 1, verifier included) before the freeze; fixes recorded here. Then
`eval/meta2h/FREEZE.json` (tasks, prompts, briefs, code, snapshot hashes) committed before any test run.

## 8. Review 1 (adversarial, before the freeze) and changes

No arm had run. Findings and fixes:
- **Critical:** failed verifier batches were never retried and still scored → only a batch whose audit passed
  counts; up to 3 attempts; scoring refuses to run while any point lacks a valid first-pass verdict.
- **Critical:** verifier easier on open-access corpus sources than on paywalled web sources, and one work's points
  scattered over batches → points grouped by source work into the same batch (blind to arm); verifier told to try
  publisher, DOI, arXiv/OSTI and the library before `inaccessible`; accessibility sensitivity analysis (§6).
- **Critical:** 12 pairs pseudo-replicated → task-level primary, D vs C co-primary with Holm (§6).
- **High:** derived quantities (ratios, exponents, coefficients) → arms quote the inputs and state the calculation;
  "supported" defined for derived values.
- **High:** dedupe missed DOIs in publisher URLs, `www.doi.org`, arXiv ids, device aliases, rounding → DOI regex
  anywhere, arXiv ids, device alias map, 2 significant figures; tests.
- **High:** arm identity visible to the verifier (URL of the page used, `location` with chunk ids) → the verifier
  sees only value, unit, device, conditions, quote and authors/title/year/DOI; id patterns stripped.
- **High:** arm failures turned into V = 0 after 2 attempts → up to 3 attempts for any failure; audit call cap
  200 (brief asks ≤ 150); per-arm void rates reported. "One point per device and condition per paper" in the brief.
- **Medium:** freeze now also hashes this protocol, the shared audit module, the wrapper and the snapshot file
  itself (recomputed); scoring checks the freeze; template braces unescaped before inserting points/questions.

## 9. Pilot (practice task x01, fuel retention; before the freeze)

4 arms × 1 run, all audits passed (A: no tools, 1 point; B: 35 corpus calls, 10 points; C: 37 searches + 67
fetches, 9 points; D: 45 corpus calls and no web, 9 points). Blind verification: 28 distinct points from 11 works in
2 batches + second-verifier sample; 0 unverified; kappa 0.77 (n = 7). Precision 0.9–1.0 for B, C, D. Change made:
the verifier marked the task's explicitly requested comparison values (carbon-wall references) relevant for one
device and not for others → verifier rule now counts explicitly requested comparison values as relevant (same rule
for every point). Pilot numbers are not results.
