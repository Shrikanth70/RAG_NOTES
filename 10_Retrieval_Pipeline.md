# Note 10: The Retrieval Pipeline (Phase 2)

## 1. Concept Overview
The **Retrieval Pipeline** is the real-time execution engine of a RAG system. It is activated the moment a user submits a query and coordinates five sequence steps to return a response:
1. **Query Processing**: Clean and normalize the user's text question.
2. **Query Embedding**: Convert the question into a coordinate vector using the same model used for ingestion.
3. **Similarity Search**: Query the vector database to locate the closest stored text chunks.
4. **Context Construction**: Sort, rank, format, and stuff the retrieved text chunks into the LLM system prompt.
5. **Grounded Generation**: Execute the LLM call using the structured prompt to stream a hallucination-free answer.

---

## 2. Why It Exists
An LLM alone cannot answer questions about private or new documents because it does not possess that knowledge. The retrieval pipeline acts as the pipeline that extracts the correct knowledge card from the database and inserts it into the LLM's short-term memory (its context window), enabling the LLM to write a factually accurate response.

---

## 3. Real-World Analogy
Imagine walking up to a smart librarian and asking: *"How do I fix error code 5 on my toaster?"*:
1. The librarian hears your question (Query).
2. The librarian mentally translates it to find related concepts like "Toaster Troubleshooting" (Query Embedding).
3. The librarian walks down the aisles and grabs the 3 most relevant manuals (Similarity Search).
4. The librarian opens the manuals to the relevant pages and lays them open on the table (Context Construction).
5. The librarian reads the pages and speaks the steps to you clearly (Grounded Generation).

---

## 4. Technical Explanation
The retrieval lifecycle runs synchronously to keep latencies low:

```
[User Query] 
     |
     v
+------------------+
| Embedder Model   | ---> Generates Vector representation of Query (e.g. 1 x 384)
+------------------+
     |
     v
+------------------+
| Vector Database  | ---> Performs Cosine Similarity lookup; returns Top-K documents
+------------------+
     |
     v
+------------------+
| Prompt Assembler | ---> Injects Chunks and citations into Prompt Template
+------------------+
     |
     v
+------------------+
| Gemini Developer API | ---> Streams token-by-token response back to User Interface
+------------------+
```

To prevent the model from answering from its general knowledge, the Grounding Layer enforces a strict system prompt: if no chunks match, it immediately halts and outputs: *"I could not find this information in the provided documents."*

---

## 5. Industry Best Practices
- **Match Embedding Models**: Never use one embedding model for ingestion (e.g., MiniLM) and a different one for retrieval (e.g., OpenAI). The mathematical coordinates will not align, and search will return random noise.
- **Set a Minimum Threshold**: Define a similarity threshold (e.g. 0.65 for Cosine Similarity). If the best match score is lower than this, reject the search and return the fallback message immediately without calling the LLM. This saves money and prevents hallucination.
- **Optimize Prompt Formatting**: Format the retrieved chunks clearly with clear delimiters (like `---` or `<doc>`) so the LLM can separate one source from another.

---

## 6. Common Mistakes
- **High Latency Embedding Generation**: Using a slow, distant cloud API for embedding queries. The user's query must be embedded immediately; local models or highly available endpoints are preferred for this step.
- **Excessive Top-K**: Setting K too high (e.g. K=15). This feeds too much text to the LLM, inflating tokens, increasing latency, and introducing irrelevant details.

---

## 7. Visual Diagram (ASCII)
```
+-------------------------------------------------------------------------+
|                        RETRIEVAL PIPELINE SEQUENCE                      |
+-------------------------------------------------------------------------+
 User Query: "Who signed the contract?"
   |
   +--> [Embedder] -> Vector: [0.08, -0.12, 0.54]
                         |
                         v
                    [Vector DB] -> Searches Index
                         |
                         v
                    Matches: Chunk #14 (Score: 0.88), Chunk #2 (Score: 0.82)
                         |
                         v
                    [Prompt Builder] -> Fills Template:
                    "Context: Chunk #14 Text ... Chunk #2 Text ..."
                         |
                         v
                      [Gemini]   -> LLM processes prompt and generates stream.
```

---

## 8. Interview Questions
1. **If a RAG system answers a query with information not present in the documents, which stage of the retrieval pipeline failed?**
   *Answer*: This is a grounding failure. It means either: (1) the Prompt Builder did not enforce strict system rules, allowing the LLM to hallucinate, or (2) the retrieval stage fetched irrelevant chunks but the LLM tried to guess the answer anyway using its pre-trained weights.
2. **How do you optimize retrieval pipeline latency?**
   *Answer*: (1) Use local embedding models, (2) index the vector database with HNSW, (3) cache common queries, and (4) stream the LLM response tokens to the UI as they are generated.

---

## 9. Key Takeaways
- The retrieval pipeline is real-time and user-facing.
- It translates text queries to vectors, searches, builds prompts, and generates text.
- Perfect alignment between ingestion and retrieval embedding models is mandatory.
- Guardrails are applied at the prompt level to enforce strict document grounding.

---

## 10. Beginner Summary
The Retrieval Pipeline is the motor under the hood of a chatbot. When you type a question, the motor instantly runs a search in the vector database, grabs the best matching text blocks, wraps them inside a strict instruction packet, and hands them to the AI writer, who reads the facts and writes a clean response back to your screen.
