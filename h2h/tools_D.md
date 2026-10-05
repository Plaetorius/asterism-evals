Tools: you may search a local library of fusion papers (open-access Nuclear Fusion and Plasma Physics and Controlled Fusion full texts, mostly 2014–2026, plus titles and abstracts of Nuclear Fusion and PPCF papers since 1960) with this shell command, and you may use web search and web fetch:
  {CORPUS} search "<query>" [--years Y1 Y2] [--limit N]   # full-text passages
  {CORPUS} passage <chunk_id> [--context 1]               # read a passage with context
  {CORPUS} works "<query>" [--years Y1 Y2]                # titles/abstracts of all catalogued papers
  {CORPUS} paper <work_id>                                # metadata and abstract
Run each corpus command on its own (no pipes, no ';', no redirection). Use at most 15 tool calls per question in total. You may also use your own knowledge. Do not use any other tool except the single file write required at the end.
