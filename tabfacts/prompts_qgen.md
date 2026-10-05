You are writing one test question from a table in a recent fusion-science paper. You have no tools except reading
one image file and writing one file. Use only what is given here.

Paper (published {PUBLISHED}, {VENUE}). TITLE: {TITLE}
ABSTRACT: {ABSTRACT}

First read the image {IMAGE}: it is Table {TABLE_NO} of this paper (with its caption), exactly as printed.

Write "table": one question whose answer is a single numeric value printed in a cell of this table (or a range
printed in one cell), together with its unit and conditions. Rules:
- Pick a cell that is a result or a defining parameter specific to this work (not a well-known constant), and whose
  row and column are clearly labelled.
- The question must identify the cell in words: the device or system, the quantity (column), the case or condition
  (row) and say the result was published in 2025–2026 in Nuclear Fusion or Plasma Physics and Controlled Fusion. Do
  not mention "Table N", the paper's title, any author name or the DOI. Paraphrase; do not copy the caption.
- Never state the answer value in the question.
- "value" is the number exactly as printed in the cell (for a printed range use "value_low"/"value_high"); "unit"
  is the unit that applies to it (from the column header or caption), "" if dimensionless; "cell" names the row
  and column as printed (e.g. "row: Pedestal top; column: a/L_Te").
- If no cell satisfies these rules, write {{"table": null}}.

Write exactly one file, {OUTFILE}, with the Write tool: a JSON object
{{"table": {{"question": "...", "value": 0.0, "value_low": null, "value_high": null, "unit": "...", "cell": "..."}}}}
Then reply DONE.
