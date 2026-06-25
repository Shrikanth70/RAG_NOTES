# Note 19: RAG Interview Questions & Answers

## 1. Concept Overview
This note compiles a comprehensive list of technical interview questions and answers focused on Retrieval-Augmented Generation (RAG) system design. The questions span five progressive levels of expertise: foundational concepts, implementation details, vector database mechanics, production optimizations, and troubleshooting.

---

## 2. Why It Exists
RAG has become one of the most highly sought-after skills in modern software engineering and AI architecture. Interviewers evaluate candidates on their understanding of:
- The tradeoffs between different chunking and embedding strategies.
- Vector indexing algorithms (like HNSW).
- Latency and cost optimizations.
- Grounding and hallucination mitigations.

---

## 3. Real-World Analogy
Think of this note as a study guide for a driver's license exam. Instead of just learning how to turn the steering wheel (code syntax), you learn how the engine works, what to do if the brakes fail on a wet road (troubleshooting), and how to navigate busy highways safely (production scale).

---

## 4. Technical Q&A (15+ Questions)

### Foundational Concepts (Level 1)
#### Q1: What is the primary difference between RAG and Fine-Tuning?
**Answer**: Fine-tuning updates the weights of an LLM, teaching it new styles, tones, formatting, or task behaviors. RAG provides the LLM with external knowledge directly in the prompt window. Fine-tuning does not prevent hallucinations, whereas RAG restricts the model to factual references.

#### Q2: What are the two main phases of a RAG architecture?
**Answer**: (1) **Ingestion Pipeline**: An offline phase that loads, cleans, chunks, embeds, and stores documents in a vector database. (2) **Retrieval Pipeline**: An online phase that processes a user's query, searches the database for relevant chunks, constructs a grounded prompt, and calls the LLM.

#### Q3: Why is a system temperature of 0.0 recommended for RAG generation?
**Answer**: Temperature controls the randomness of token generation. Setting it to 0.0 forces the model to use greedy decoding, selecting the absolute most probable words. This makes the output deterministic and helps prevent hallucinations.

---

### Implementation Concepts (Level 2)
#### Q4: What is the purpose of chunk overlap?
**Answer**: Chunk overlap ensures that semantic information located at the boundary of a split is not lost or severed in half. It guarantees that any sentence or phrase is fully captured in at least one chunk.

#### Q5: Why is recursive character splitting preferred over fixed-size character splitting?
**Answer**: Fixed-size splitting slices text at exact character lengths, which can break sentences or words in half. Recursive splitting attempts to split by paragraphs (`\n\n`), then sentences (`\n`), then words (` `), keeping related text structures together.

#### Q6: How do you handle multi-column PDFs during document loading?
**Answer**: Standard loaders read text left-to-right, which can garble multi-column text. To prevent this, use layout-aware PDF parsers (like `pdfplumber` or `unstructured`) that analyze spatial layouts to reconstruct columns in their proper reading order.

---

### Vector Space Mechanics (Level 3)
#### Q7: Explain the difference between Cosine Similarity and Dot Product.
**Answer**: Cosine similarity measures the angle between two vectors, ignoring their length (magnitude). Dot product multiplies matching coordinates and sums them, which is sensitive to length. If vectors are normalized to unit length, Dot Product is identical to Cosine Similarity.

#### Q8: How does HNSW indexing work, and what are its trade-offs?
**Answer**: HNSW (Hierarchical Navigable Small World) constructs a multi-layered graph where upper layers contain sparse, long-distance jumps and lower layers contain dense, short-distance jumps. It allows for fast search speeds ($O(\log N)$) but requires keeping the graph in RAM, which increases memory usage.

#### Q9: What is metadata pre-filtering, and why is it preferred over post-filtering?
**Answer**: Pre-filtering applies metadata filters (like `tenant_id`) *before* the vector search, ensuring that only eligible vectors are compared. Post-filtering performs the vector search first, and then discards results that do not match the filter, which can lead to retrieving fewer than K results if many are filtered out.

---

### Production & Optimization (Level 4)
#### Q10: What is Semantic Caching?
**Answer**: Semantic caching stores past user queries and generated answers. When a new query is received, we generate its embedding and measure similarity against cached queries. If a match is found above a high threshold (e.g., 0.95), we return the cached response. This saves the latency and cost of a database search and LLM call.

#### Q11: Explain the "Lost in the Middle" problem and how to mitigate it.
**Answer**: LLMs are good at identifying information at the very beginning or the very end of a prompt, but tend to ignore details located in the middle. Mitigation: (1) Sort retrieved chunks so the most relevant ones are at the very top or bottom of the context block, (2) keep $K$ small, or (3) use a reranker.

#### Q12: How would you design a RAG system to support real-time data updates (e.g. editing a file)?
**Answer**: Use deterministic chunk IDs (like a hash of `filename` + `chunk_index`). When a file is updated, delete all stored chunks matching its filename, re-ingest the file, and insert the new chunks. This maintains consistency and avoids stale data.

---

### Troubleshooting (Level 5)
#### Q13: If a RAG system answers a query with information not present in the documents, how do you debug this?
**Answer**: (1) Check if the prompt template has strict grounding constraints. (2) Verify that the LLM temperature is set to 0.0. (3) Confirm if the similarity threshold is too low, allowing irrelevant chunks to pass as valid context.

#### Q14: How do you choose the optimal chunk size and overlap for a dataset?
**Answer**: Run evaluations using different settings on a test dataset. Use smaller chunks (e.g., 200–500 characters) for short, factual lookup questions; use larger chunks (e.g., 1000–2000 characters) for synthesis or summarization questions.

#### Q15: What is a hybrid search, and when should you use it?
**Answer**: Hybrid search combines keyword search (like BM25) with semantic vector search. The results are merged using algorithms like Reciprocal Rank Fusion (RRF). Use hybrid search when your queries contain both exact terms (like serial numbers or SKU codes) and conceptual descriptions.

---

## 5. Visual Diagram (ASCII)
```
                  [ Interview Prep Knowledge Map ]
                                |
         +----------------------+----------------------+
         |                                             |
    [ Pipeline Math ]                             [ System Design ]
    - Cosine Similarity: angle metric             - Pre-filtering vs Post-filtering
    - HNSW Graph: highway navigation              - Semantic Caching for speed
    - Embeddings: conceptual mapping              - Celery for async ingestion
```

---

## 6. Key Takeaways
- Understand how vector index algorithms (HNSW, IVF) trade RAM for search speed.
- Know the differences between Cosine, Euclidean, and Dot Product similarity metrics.
- Be ready to explain grounding, temperature settings, and negative constraints.
- Explain semantic caching and async queues as scaling solutions.

---

## 7. Beginner Summary
This note is a compilation of 15 essential questions commonly asked in engineering interviews for RAG positions. It covers everything from the basics of chunking and embedding to advanced topics like graph-based database search and cost saving using semantic caching. Use this guide to verify your understanding of RAG architectures.
