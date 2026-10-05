You are checking data points extracted from the fusion literature for a meta-analysis. You did not extract them
and you don't know who did; do not try to work out where they came from.

Task the data points are meant for: {QUESTION}
Inclusion criteria: {INCLUSION}

Tools: to look up each source you may use web search and web fetch, and this local paper library (run each
command on its own, no pipes, no ';', no redirection):
  {CORPUS} works "<title or DOI words>"     # find a paper by title
  {CORPUS} search "<query>" [--limit N]       # full-text passages
  {CORPUS} passage <chunk_id> [--context 1]   # read a passage
  {CORPUS} paper <work_id>                    # metadata
Use either or both for every point, whichever reaches the source text. Budget: about 6 tool calls per point.

Points from the same work are given together; check that work once for all of them. Some quantities are derived
(a ratio to a scaling law, an exponent, a coefficient): for those, "supported" means the source states the inputs
(or the derived value itself) and the stated value follows from them within normal rounding.

For each point below, find the cited source and decide:
- "verdict": "supported" (the source states this value — or range — with this unit for this device, within normal
  rounding), "partial" (the value is in the source but the device, conditions or unit differ), "unsupported" (the
  source exists and you could read the relevant part, but it does not state this value), "source_not_found" (no
  such work exists as cited), or "inaccessible" (the work exists but you could not read the relevant text — try the publisher page, the
  DOI, open versions such as arXiv or OSTI, and the local library before concluding this);
- "relevant": true if the point meets the inclusion criteria, or is a comparison value the task explicitly asks
  for (for example reference values from the same devices under other conditions); else false;
- "evidence": one sentence (what you found, where).

Points (JSON):
{POINTS}

Write exactly one file, {OUTFILE}, with the Write tool: a JSON object {{"<point_id>": {{"verdict": "...",
"relevant": true, "evidence": "..."}}, ...}} covering every point. Do not read any other file. Then reply DONE.
