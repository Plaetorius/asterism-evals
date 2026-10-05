Rewrite an assistant's answer into a neutral list of atomic factual claims, for blind grading.

Question: {QUESTION}
Answer (JSON): {ANSWER}

Rules: one claim per item, each a short declarative sentence; keep every number with its unit and conditions; keep
hedges that change meaning ("about", "up to"); drop citations, author names, paper titles, DOIs, references to
sources, tools, searches or "the literature/corpus", and all style, formatting and filler. Do not add, correct or
judge anything. "abstained": true if the answer declines to answer or says it doesn't know (even if it adds
generic background). "value": the answer's main number with unit if the question asks for a quantity, else "".
Return only JSON.
