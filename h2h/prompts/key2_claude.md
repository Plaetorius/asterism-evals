You are preparing an independent ANSWER KEY for an evaluation of research assistants in fusion science. For each
question below, find primary sources and write the key. You may use web search and web fetch, and this command to
search a local library of fusion papers (open-access Nuclear Fusion and PPCF full texts, mostly 2014–2026, plus
titles and abstracts of Nuclear Fusion and PPCF papers since 1960):
  {CORPUS} search "<query>" [--years Y1 Y2] [--limit N]
  {CORPUS} passage <chunk_id> [--context 1]
  {CORPUS} works "<query>" [--years Y1 Y2]
  {CORPUS} paper <work_id>
Run each command on its own (no pipes, no ';', no redirection). Do not read any file and do not use any other tool
except the single file write at the end. Use both the web and the library; base the key on sources, not memory.

For each question write: "answerable" (true if the published literature allows a definite answer; for "has
anyone…" questions "no published report found" counts as a definite answer, answerable = true; false only for
future events, nonexistent things or non-fusion questions); "key_facts" (the specific facts a correct answer must
contain, at most 6, each one short sentence with values, units, conditions, devices; for yes/no questions the
first fact states yes or no); "acceptable_variants"; "notes" (caveats, disagreements, essential vs supporting);
"sources" (title, year, DOI or URL of what you used).

Questions:
{QUESTIONS}

Write exactly one file, {OUTFILE}: a JSON object {"<qid>": {"answerable": ..., "key_facts": [...],
"acceptable_variants": [...], "notes": "...", "sources": [...]}, ...} covering every question. Then reply DONE.
