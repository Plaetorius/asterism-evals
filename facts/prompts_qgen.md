You are writing test questions from one recent fusion-science paper. You have no tools except writing one file and
no other information than the paper text below. Write questions that a researcher could ask without knowing which
paper reports the answer.

Paper (published {PUBLISHED}, {VENUE}):
TITLE: {TITLE}
ABSTRACT:
{ABSTRACT}
{BODY_BLOCK}

Write:
1. "abs": one exact quantitative result stated in the ABSTRACT — a number specific to this paper's finding (a
   measured or computed value, not a well-known constant or a generic input), with its unit and conditions.
2. "body": {BODY_INSTRUCTION}
3. "lookup": {LOOKUP_INSTRUCTION}

Rules for every question:
- Self-contained: name the device or system, the quantity and the conditions, and say the result was published in
  2025–2026 in Nuclear Fusion or Plasma Physics and Controlled Fusion. Ask for a single number (or a range if the paper
  gives a range).
- Write the question in your own words, the way a researcher would ask it: paraphrase, do not copy phrases from
  the paper (no run of more than 4 consecutive words from the paper, other than names of devices, codes and
  quantities).
- Never state the answer value (or any value from the quote) in the question.
- Do not include the paper's title or a paraphrase of it, any author name, the DOI, or ≥ 5 consecutive words of the title.
- "quote" must be copied verbatim from the paper text above (the sentence that states the value) and must contain
  the number.
- "value" is the number as written in the quote (use "value_low"/"value_high" for a range), "unit" as written.

Write exactly one file, {OUTFILE}, with the Write tool: a JSON object
{{"abs": {{"question": "...", "value": 0.0, "value_low": null, "value_high": null, "unit": "...", "quote": "..."}},
"body": {{...same fields...}} or null,
"lookup": {{"question": "...", "quote": "..."}} or null}}
Then reply DONE.
