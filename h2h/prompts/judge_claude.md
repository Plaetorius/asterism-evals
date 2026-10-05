The packet at the end of these instructions is a JSON list of items, each with a fusion-science question, an
answer key, and anonymous answers labelled W/X/Y/Z (each a list of neutral atomic claims). Do not read any other
file and do not use any other tool or web search. Do not try to work out which system wrote which answer.

For every item and every label, grade the answer against the key:
- "correct": contains the key's essential facts (or an acceptable variant) and nothing that contradicts the key;
  numbers must agree with the key within normal rounding or a stated acceptable variant;
- "partial": some essential facts right, others missing or vague, nothing central contradicted;
- "incorrect": misses the essential facts, states something central that contradicts the key, or answers when the
  key says the question is not answerable;
- "abstained": declines or says it doesn't know (when the key says answerable=false, an abstention or a statement
  that it is unknown / doesn't exist counts as "correct" instead).
When the key contains two independently written keys (key_1 and key_2), grade against both: where they agree, use them as one key; where they conflict on a fact (see "conflicts"), an answer consistent with either key on that fact is acceptable.
Claims beyond the key are fine unless they contradict it. Grade against the key only, not against what you can
verify yourself.

Write exactly one file, {OUTFILE}: a JSON object {"<qid>": {"grades": [{"label": "W", "grade": "...",
"contradicts_key": false, "reason": "<one sentence>"}, ...]}, ...} covering every item and label. Then reply DONE.

Packet:
{PACKET}
