# Note 06: Chunking Strategies (Slicing the Text)

## 1. Concept Overview
Chunking is the process of breaking a continuous body of text into smaller, discrete segments (chunks) before embedding and storing them. 

Rather than treating a 100-page book as one giant document, we chop it into hundreds of small paragraphs or sections. The two main parameters that govern this are:
- **Chunk Size**: The maximum length of each segment (measured in characters or tokens).
- **Chunk Overlap**: The number of characters/tokens shared between consecutive chunks to prevent semantic boundary loss.

---

## 2. Why It Exists
Chunking is necessary because:
1. **Context Limits**: LLMs have a maximum context window. We cannot feed a massive file to the LLM on every query.
2. **Search Granularity**: If we embed an entire book as a single vector, the specific details get averaged out. Slicing the book ensures that small, detailed facts get their own individual vectors.
3. **Relevance**: Chunking enables us to retrieve only the 3 specific paragraphs containing the answer, rather than the entire chapter.

---

## 3. Real-World Analogy
Imagine you have a long rope and you want to find the exact spot where a small knot is tied:
- If you keep the rope coiled up (no chunking), it's hard to analyze its details quickly.
- If you cut the rope into 1-foot pieces (small chunks), you can examine each piece individually and find the knot immediately.
- If you overlap the cuts slightly (overlap), you ensure you don't accidentally cut directly *through* the knot and destroy its structure.

---

## 4. Technical Explanation
There are several chunking algorithms:
1. **Fixed-Size Chunking**: Slices text at exact character lengths. (Fast but breaks sentences in half).
2. **Recursive Character Chunking (Best Practice)**: Splits text by a list of separators in order (typically `\n\n` (paragraphs), `\n` (lines), ` ` (words), `""` (characters)), attempting to keep paragraphs and sentences together until the chunk size limit is reached.
3. **Semantic Chunking**: Computes distance metrics between consecutive sentences using embedding models. When the similarity between sentence $i$ and sentence $i+1$ drops below a certain threshold, it indicates a change of topic, and a split is made.

```
Example of Overlapping Chunks (Size: 100 chars, Overlap: 20 chars):

[ --- Chunk 1 --- ]
"Modern RAG systems utilize vector search to locate text. They are highly efficient."

            [ --- Chunk 2 --- ]
           "to locate text. They are highly efficient. However, tuning is required."
```

---

## 5. Industry Best Practices
- **Use Recursive Splitters**: Standardize on recursive character splitting. It is computationally cheap and preserves structural meaning.
- **Overlap is Crucial**: Always set an overlap (typically 10% to 20% of the chunk size). This ensures that facts split across a chunk boundary are still captured coherently in at least one chunk.
- **Match Chunk Size to Use Case**: Use smaller chunks (e.g., 200–500 characters) for short, factual lookup questions; use larger chunks (e.g., 1000–2000 characters) for synthesis or summarization questions.

---

## 6. Common Mistakes
- **Zero Overlap**: This frequently cuts critical information (e.g., a phone number, a formula, or a key relationship) in half, rendering it unsearchable.
- **Chunks Too Small**: If your chunks are only 50 characters long, they lose the surrounding context (e.g., getting the chunk "he failed" without knowing *who* "he" refers to).

---

## 7. Visual Diagram (ASCII)
```
[Continuous Text Document: Slices 1 through 1000]

Fixed Split (No Overlap):
|-- Chunk 1 (1-400) --||-- Chunk 2 (401-800) --||-- Chunk 3 (801-1200) --|

Recursive Split (With Overlap):
|-- Chunk 1 (1-400) --|
          |-- Chunk 2 (300-700) --|
                    |-- Chunk 3 (600-1000) --|
```

---

## 8. Interview Questions
1. **Explain the trade-offs of large vs small chunk sizes.**
   *Answer*: Small chunks (e.g. 200 tokens) yield highly specific embeddings, resulting in precise retrieval, but may lack surrounding context. Large chunks (e.g. 1000 tokens) retain rich context and relationships but dilute specific facts, making retrieval less precise and consuming more prompt tokens.
2. **What is the purpose of chunk overlap?**
   *Answer*: Overlap ensures that semantic information located at the boundary of a cut is not severed. It guarantees that any sentence or phrase is fully contained in at least one chunk, maintaining context for the embedding model.

---

## 9. Key Takeaways
- Chunking splits text into manageable sizes.
- Size vs. Overlap are the main configurations.
- Small chunks: high precision, low context. Large chunks: low precision, high context.
- Recursive splitters are the default choice for general text; semantic splitters split by topic.

---

## 10. Beginner Summary
Chunking is like cutting a long scroll of text into smaller paragraphs. If we keep the text too long, the system gets confused. If we cut it too small, we lose the story. We cut the text into small paragraphs that overlap slightly at the edges, making it easy to search and read.
