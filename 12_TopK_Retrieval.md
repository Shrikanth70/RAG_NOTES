# Note 12: Top-K Retrieval (Choosing the Scope)

## 1. Concept Overview
The parameter **K** in **Top-K Retrieval** represents the number of most-similar text chunks retrieved from the vector database to answer a user's query. 

For instance, if you set $K=3$, the vector database will return exactly the three chunks with the highest similarity scores. These chunks are then sorted and passed to the LLM as context.

---

## 2. Why It Exists
We cannot send every document we own to the LLM because of context window sizes, cost, and processing speeds. On the other hand, sending only a single chunk ($K=1$) might miss crucial supporting facts spread across multiple pages. 

The $K$ parameter is the main dial for balancing:
- **Recall**: Do we have enough information to answer the question?
- **Precision**: How much irrelevant noise are we sending to the LLM?
- **Cost/Latency**: More chunks mean more tokens, which costs more money and increases generation time.

---

## 3. Real-World Analogy
Imagine you are writing a research paper about a historical event:
- **K = 1**: You are allowed to read only a single paragraph from a single book. If that paragraph is missing the date of the event, you fail.
- **K = 5**: You are allowed to read five paragraphs from various books. This is a balanced pile; it has enough detail to cover the event from different angles without overwhelming you.
- **K = 50**: You are given a stack of fifty books. It will take you hours to read them all (high latency), you'll get tired (cost), and you'll find a lot of duplicate or off-topic information (noise).

---

## 4. Technical Explanation
The retrieval function takes the query vector and $K$ as inputs:

```python
def retrieve_top_k(query_vector: list[float], k: int) -> list[dict]:
    # Query vector database with limit K
    raw_results = vector_db.search(query_vector, limit=k)
    
    # Format and return chunks
    retrieved_chunks = []
    for doc in raw_results:
        retrieved_chunks.append({
            "text": doc.page_content,
            "score": doc.similarity_score,
            "metadata": doc.metadata
        })
    return retrieved_chunks
```

### Tradeoffs of Different K values:
- **K = 2**: Low token cost, fast speed. Best for simple, direct lookups (e.g. *"What is the office phone number?"*). Risk: missing relevant context if the answer is complex.
- **K = 5**: Good balance for general Q&A. Captures nuances across a couple of pages.
- **K = 10**: High recall. Useful for synthesis and comparisons (e.g. *"Summarize the differences between Plan A and Plan B"*). Risk: "Lost in the Middle" where the LLM ignores chunks in the middle of the prompt.

---

## 5. Industry Best Practices
- **Implement a Score Cutoff**: Do not blindly accept all K chunks. If $K=5$ but chunks 4 and 5 have very low similarity scores (e.g. < 0.4), drop them. Only pass high-quality matches to the LLM.
- **Combine with Reranking**: Retrieve a slightly larger $K$ (e.g., $K=15$) and then run them through a secondary, more expensive "Reranker" model (like Cohere Rerank or BGE-Reranker) to select the absolute best 4 chunks.
- **Make K Dynamic**: Adapt $K$ based on the complexity of the user's query or the type of document being queried.

---

## 6. Common Mistakes
- **Hardcoding a Large K Without Monitoring Costs**: Setting $K=10$ with chunks of 1000 tokens each means sending 10,000 tokens to the LLM on *every single message*. This will drain your API balance rapidly.
- **Ignoring the Context Window Limit**: Setting $K$ so high that the combined prompt exceeds the LLM's maximum token capacity, leading to API failures.

---

## 7. Visual Diagram (ASCII)
```
          [ Query Vector ]
                 |
                 v
      +----------------------+
      |  Vector DB Search    |
      +----------------------+
                 |
     +-----------+-----------+
     | (Sorted by Similarity)|
     v                       v
Rank 1: Score 0.92  [Keep]   (If K=3, these are
Rank 2: Score 0.88  [Keep]    retrieved and sent
Rank 3: Score 0.81  [Keep]    to the prompt)
--------------------------
Rank 4: Score 0.62  [Discarded]
Rank 5: Score 0.54  [Discarded]
```

---

## 8. Interview Questions
1. **What is the "Lost in the Middle" phenomenon, and how does it relate to Top-K?**
   *Answer*: Research shows that LLMs are very good at identifying information at the very beginning or the very end of a prompt, but tend to ignore or "forget" details located in the middle. If $K$ is large (e.g. $K > 8$), the critical chunk might get buried in the middle of the context, causing the LLM to hallucinate or say it cannot find the answer.
2. **How does adding a reranker change your Top-K strategy?**
   *Answer*: A reranker allows you to separate retrieval into two stages: (1) retrieve a wider net of cheap candidate chunks (e.g., $K=20$) using fast vector search, and (2) re-score and sort those candidates using a cross-encoder model to select the final $K=3$ highest-quality chunks. This increases accuracy without blowing up prompt lengths.

---

## 9. Key Takeaways
- $K$ is the number of text chunks fetched from the database.
- Low $K$ is fast and cheap, but might miss the answer (low recall).
- High $K$ captures more details, but is slower, more expensive, and can confuse the LLM.
- Best practice is to use $K=3$ to $K=5$ for standard systems, filtered by similarity threshold.

---

## 10. Beginner Summary
Top-K is like asking a librarian for a specific number of books. If you say "give me Top-2 books", they will hand you the two books that match your question the best. If you say "give me Top-10", you get a huge stack. You want to pick a number that is just enough to get the answer, but not so big that you spend the whole day reading.
