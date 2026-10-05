# Asterism evaluations

The evidence behind the results claimed for [Asterism](https://github.com/Plaetorius/asterism), the open fusion
literature corpus with an MCP server: does an LLM research assistant do better with the corpus, with web search, or
with both? Every study has a protocol that was written and frozen **before** any run, a freeze file of hashes of every
input, and a results file. Answers were graded by code against a key, not by a model.

Same model in every setup (Claude Opus 5.5). "Web search" means the assistant's built-in web search and fetch.

| Claim | Result | Protocol | Results |
|---|---|---|---|
| Exact facts from recent papers | corpus 93 % vs web 74 % (129 questions) | [FACTS.md](FACTS.md) | [facts/RESULTS.md](facts/RESULTS.md) |
| Values printed in tables | corpus with table layer 99 % vs web 57 % (69 questions) | [TABLE_FACTS.md](TABLE_FACTS.md) | [tabfacts/RESULTS.md](tabfacts/RESULTS.md) |
| Papers public after the training cutoff | corpus 98 % vs web 48 % (124 fact questions) | [POSTCUT.md](POSTCUT.md) | [POSTCUT_RESULTS.md](POSTCUT_RESULTS.md) |
| Open-ended questions | 47 % vs 52 %, p = 0.66: inconclusive | [HEAD2HEAD.md](HEAD2HEAD.md) | [h2h/RESULTS.md](h2h/RESULTS.md) |
| Broad evidence gathering | inconclusive (6 tasks) | [META2HEAD.md](META2HEAD.md) | [meta2h/RESULTS.md](meta2h/RESULTS.md) |

Inconclusive means the studies were not large enough to tell; it does not mean "as good as".

## How to check a claim

1. Read the protocol: questions, grading rule and decision thresholds were fixed in advance.
2. Check the freeze file (`<study>/FREEZE.json`): the sha256 of every prompt, brief and tool list used.
3. Read the results file: counts, intervals, paired tests and the audit outcome. Every number on the Asterism site
   and in the announcement comes from these files.
4. [METHODS.md](METHODS.md) explains how all studies were run; [paper/ARTICLE.md](paper/ARTICLE.md) is a short
   summary and [paper/PAPER.md](paper/PAPER.md) the long draft.

## What is not here yet

The raw per-question answers, grades and transcripts (the benchmark data bundle) and the analysis code are not
published yet, so the scoring cannot yet be re-run from this repository. Protocols, freezes and results can be
read and cross-checked now. The protocols refer to internal files that are not part of this repository. A
replication with a second model family (Gemini) is still running and is not reported.

## Limits

Two journals; one model; question writers and judges were AI agents, not practising researchers; web results depend
on the day of the run. The project's owner stated at the outset that results may be used in marketing; this is
disclosed in the paper and every verdict follows a rule fixed before the run.

Corpus, server and data: https://github.com/Plaetorius/asterism. Licence: [LICENSE.md](LICENSE.md).
