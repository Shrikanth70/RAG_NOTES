# Note 15: Hallucination Prevention (Strict Grounding)

## 1. Concept Overview
**Hallucination Prevention** is the design of strategies to prevent an LLM from generating inaccurate, fabricated, or unverified claims. In a RAG system, this is achieved through **Strict Grounding**, which confines the LLM's answers entirely to the provided reference context.

To guarantee grounding, the system uses three layers of defense:
1. **Low Temperature**: Configuring the LLM's generation temperature to `0.0`.
2. **Relevance Thresholding**: Blocking retrieval of low-similarity chunks before they reach the prompt.
3. **Negative Prompt Constraints**: Instructing the model to output a strict fallback string if the context is insufficient.

---

## 2. Why It Exists
LLMs are designed to predict the most likely next word, making them naturally creative and eager to please. When asked a question for which they lack information, they will combine fragments of their training data to make up an answer. 

In enterprise environments (like medicine, finance, or customer service), a hallucination can be disastrous. The chatbot must be honest about what it does not know.

---

## 3. Real-World Analogy
Imagine a witness on a stand in a courtroom:
- **Hallucinating Witness**: When asked, *"Was the suspect at the bank?"*, the witness thinks, *"Well, the suspect is a shady person, so probably."* They answer, *"Yes, he was."* This is a lie.
- **Strictly Grounded Witness**: The witness is told, *"Answer ONLY based on what you saw with your own eyes. If you did not see it, say 'I do not know.'"* The witness answers, *"I did not see the suspect at the bank."* This is honest and accurate.

---

## 4. Technical Explanation
To ensure grounding, we implement a multi-layered verification strategy:

### Layer 1: Temperature = 0.0
Setting the temperature to `0.0` forces the LLM to choose the highest-probability tokens (greedy decoding), making its responses deterministic and suppressing creative improvisation.

### Layer 2: Distance Score Thresholding
If the best chunk returned from the database has a similarity score below our threshold (e.g. `0.65`), we intercept the query before it reaches the LLM and return the fallback message immediately.

```python
def query_rag_system(query: str, threshold: float = 0.65) -> str:
    # 1. Retrieve chunks
    chunks = vector_db.similarity_search(query, k=3)
    
    # 2. Check if the best match is below the threshold
    if not chunks or chunks[0]["score"] < threshold:
        return "I could not find this information in the provided documents."
        
    # 3. If valid, build prompt and call LLM
    prompt = assemble_grounded_prompt(query, chunks)
    return llm.generate(prompt)
```

### Layer 3: Negative Constraints
We explicitly state in the system instructions: *"If the context is insufficient, reply EXACTLY with 'I could not find this information in the provided documents.' Do not say anything else."*

---

## 5. Industry Best Practices
- **Deterministic Fallbacks**: Use a simple, static error string. Do not let the LLM generate its own error message (e.g. *"I'm sorry, I looked through the manual but unfortunately..."*), because it might insert hallucinations into the apology.
- **Log Grounding Failures**: Log all queries that trigger the fallback response. This helps identify gaps in your knowledge base documents.
- **Independent Evaluation**: Run an evaluation step (using tools like Ragas or TruLens) that calculates **faithfulness** (are the LLM claims supported by the context?) and **answer relevance**.

---

## 6. Common Mistakes
- **High Temperatures (e.g., > 0.7)**: High temperatures introduce randomness, allowing the model to bypass the grounding instructions and start hallucinating.
- **Relying on LLM Self-Assessment**: Asking the LLM: *"Do you know the answer to this?"* before giving it the context. The LLM cannot assess its own knowledge base accurately.

---

## 7. Visual Diagram (ASCII)
```
[User Query] -> [Vector Search] -> Best Score: 0.58 (Threshold: 0.65)
                                          |
                                          +--> [FAIL] --> Return fallback:
                                                           "I could not find..."
                                                           (Saves API tokens!)

[User Query] -> [Vector Search] -> Best Score: 0.89 (Threshold: 0.65)
                                          |
                                          +--> [PASS] --> Ingest Context 
                                                           with Temp=0.0
                                                           |
                                                           v
                                                      [Grounded LLM] -> Output
```

---

## 8. Interview Questions
1. **How does setting the LLM temperature to 0.0 prevent hallucinations?**
   *Answer*: Temperature controls the randomness of token selection. At 0.0, the model uses greedy decoding, selecting the absolute most probable token at each step. This eliminates the random selection of low-probability words, making the response highly predictable and aligned with the constraints in the prompt.
2. **What is the purpose of a similarity threshold in grounding?**
   *Answer*: A similarity threshold filters out irrelevant matches returned by the vector database when the document library does not contain the answer. By intercepting these low-scoring chunks, we avoid feeding garbage context to the LLM, which would otherwise tempt it to hallucinate.

---

## 9. Key Takeaways
- Grounding restricts LLM answers to the provided context.
- Set LLM temperature to 0.0 for deterministic, factual output.
- Intercept search results with a similarity threshold (e.g. 0.65).
- Force a strict fallback string when information is missing.

---

## 10. Beginner Summary
Hallucination prevention is like putting training wheels on a bicycle. By setting the AI's "temperature" to zero and telling it to say *"I could not find this information"* if the facts are missing, we stop it from making up stories. If it can't find the proof in the files, it will tell you honestly instead of guessing.
