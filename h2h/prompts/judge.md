You are grading anonymous answers to a fusion-science question against an answer key. Each answer is a list of
atomic claims (rewritten neutrally; you can't tell which system produced it, and you must not try).

Question: {QUESTION}
Answer key: {KEY}

{ANSWERS}

Grade every answer label independently:
- "correct": contains the key's essential facts (or an acceptable variant) and nothing that contradicts the key;
  for numbers, the value must agree with the key within normal rounding or with a stated acceptable variant;
- "partial": some essential facts right, others missing or vague, nothing central contradicted;
- "incorrect": misses the essential facts, or states something central that contradicts the key, or gives an
  answer when the key says the question is not answerable;
- "abstained": declines or says it doesn't know (for a key with answerable=false, an abstention or a statement
  that it's unknown / doesn't exist counts as "correct" instead).
When the key contains two independently written keys (key_1 and key_2), grade against both: where they agree, use them as one key; where they conflict on a fact (see "conflicts"), an answer consistent with either key on that fact is acceptable.
Claims beyond the key are fine unless they contradict it. Set "contradicts_key" when any claim contradicts the key.
Give a one-sentence reason per label. Return only JSON.
