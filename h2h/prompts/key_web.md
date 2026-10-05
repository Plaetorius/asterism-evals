You are preparing the ANSWER KEY for an evaluation of research assistants in fusion science. Base the key on the
web research notes below (gathered with Google Search from primary sources) and your expertise; where the notes and
your memory disagree, prefer the sources and say so in "notes".

Research notes:
{NOTES}

Question: {QUESTION}

Write the key as JSON with:
- "answerable": true if the published literature allows a definite answer, false only if it can't (the event
  hasn't happened, the thing doesn't exist, or the question is outside fusion science). For "has anyone…" questions,
  "no published report found" is a definite answer: answerable = true, first key fact = "No …" with its basis;
- "key_facts": the specific facts a correct answer must contain (values with units and conditions, devices,
  findings), each as one short sentence, at most 6; for a yes/no question, the first fact states yes or no;
- "acceptable_variants": other values or framings that should also count as correct (different papers, ranges,
  definitions), with their basis;
- "notes": caveats, disagreements in the literature, and which facts are essential vs supporting.
Be strict about what is established and honest about uncertainty. Return only the JSON object.
