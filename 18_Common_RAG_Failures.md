# Note 18: Common RAG Failures & How to Fix Them

## 1. Concept Overview
A RAG system can fail at various stages of the pipeline. These failures are generally grouped into:
1. **Ingestion Failures**: Corrupted text, broken columns, or metadata loss.
2. **Retrieval Failures**: Fetching irrelevant chunks or missing the correct chunk entirely.
3. **Generation Failures**: The LLM ignoring the context, hallucinating, or formatting answers incorrectly.

Understanding these failure modes is critical for diagnosing system issues.

---

## 2. Why It Exists
RAG systems are probabilistic, not deterministic. Because they rely on semantic vector space matching and LLM language generation, bugs do not always show up as explicit code crashes. Instead, the system might quietly return incorrect, misleading, or poorly formatted answers. Knowing the typical failure points helps you build appropriate safety checks.

---

## 3. Real-World Analogy
Imagine a cooking competition:
- **Ingestion Failure**: The delivery truck drops off rotten tomatoes. No matter how good the chef is, the soup will taste bad.
- **Retrieval Failure**: The assistant chef runs to the pantry but grabs salt instead of sugar. The recipe is ruined.
- **Generation Failure**: The assistant chef retrieves the correct ingredients, but the head chef ignores them and makes whatever they want.

---

## 4. Technical Explanation
Below is a matrix of common RAG failure modes, their root causes, and their fixes:

| Failure Mode | Root Cause | Practical Solution |
|---|---|---|
| **Out-of-Context Hallucination** | LLM temperature is too high, or system instructions are weak. | Set `temperature=0.0`, enforce strict system prompting constraints, and use a dedicated grounding layer. |
| **Lost in the Middle** | Important information is buried in the middle of a large context block. | Sort chunks by similarity score, keeping the most relevant ones at the beginning and end. Keep $K$ small (e.g. $K \le 5$). |
| **Scanned Document Blankness** | The PDF loader cannot read text from scanned images. | Implement an OCR parser fallback (e.g., Tesseract or PyMuPDF OCR). |
| **Empty Answer (False Negative)** | Similarity threshold is set too high (e.g. 0.85), blocking valid chunks. | Calibrate your similarity threshold based on test queries. A threshold of 0.60–0.68 is usually standard. |
| **Garbled Column Reading** | The loader reads across tables or columns horizontally. | Use layout-aware PDF parsers (like `pdfplumber` or `unstructured`). |

---

## 5. Industry Best Practices
- **Implement a Logging Audit Trail**: Log the exact chunks retrieved alongside their similarity scores for every user question. This makes debugging easy.
- **Create a Golden Dataset**: Maintain a list of 50 common questions with verified answers. Run these regularly to test for regressions when changing models.
- **Apply Guardrails**: Use frameworks like NeMo Guardrails or LlamaGuard to scan user inputs and LLM outputs for safety.

---

## 6. Common Mistakes
- **Assuming Vector Search is Perfect**: Believing that similarity search will find the correct text 100% of the time. Vector search struggles with acronyms, code syntax, and negation.
- **Upgrading the LLM to Fix a Search Bug**: Upgrading your LLM to a larger model because the system is returning wrong answers, when the real issue is that the retrieval step is not finding the relevant chunks.

---

## 7. Visual Diagram (ASCII)
```
          [ User Question ]
                 |
                 +---- Ingestion Failure? ---> (Did we load/clean/chunk correctly?)
                 |
                 +---- Retrieval Failure? ---> (Did vector search find the right chunk?)
                 |
                 +---- Generation Failure? ---> (Did LLM read the chunk and obey rules?)
                 |
                 v
          [ Grounded Answer ]
```

---

## 8. Interview Questions
1. **What is the "Lost in the Middle" problem in RAG, and how do you resolve it?**
   *Answer*: This happens when an LLM fails to pay attention to information located in the middle of a long prompt. You can resolve it by: (1) sorting retrieved chunks so the most relevant ones are at the very top or bottom of the context block, (2) keeping $K$ small, or (3) using a reranker to filter out irrelevant chunks.
2. **If a user asks a question and the system returns "I could not find this information", but the document library has the answer, how do you debug this?**
   *Answer*: (1) Check if the document was ingested correctly by looking up its filename in the database. (2) Check if the chunking size was too small, separating the question's keywords from the answer. (3) Check if the similarity score of the best chunk was below the threshold. (4) Verify that the embedding model used for the query matches the ingestion model.

---

## 9. Key Takeaways
- RAG failures can happen at the ingestion, retrieval, or generation stage.
- Low temperature and strict prompts are critical for grounding.
- Layout issues (like multi-column PDFs) require specialized loaders.
- Debugging RAG requires checking what was retrieved before checking what was generated.

---

## 10. Beginner Summary
If a chatbot gives you a wrong answer, it's usually because of one of three reasons: it read the file incorrectly in the beginning (ingestion), it grabbed the wrong page of the manual (retrieval), or it ignored the page and made up its own answer (generation). Decoupling these stages makes it easy to find and fix the problem.
