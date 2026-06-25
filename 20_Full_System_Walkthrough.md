# Note 20: Full System Walkthrough (Tracing a Document)

## 1. Concept Overview
This note provides a step-by-step trace of how the system processes a document. We will follow a test document from the moment it is uploaded, through text loading, cleaning, chunking, embedding, vector database indexing, query execution, semantic search, prompt construction, and final LLM response generation.

---

## 2. Why It Exists
Understanding individual components is useful, but seeing how they connect in sequence is essential for debugging, performance tuning, and architectural mastery. This walkthrough helps you see how the system operates as a unified machine.

---

## 3. Real-World Analogy
Imagine the lifecycle of a grain of wheat:
1. Harvested in a field (raw document).
2. Cleaned of dirt (text cleaning).
3. Ground into flour (text chunking).
4. Categorized and packaged in bags (embedding and metadata tags).
5. Placed on shelves in a warehouse (vector database storage).
6. Retrieved by a baker to make bread (retrieval, prompt construction, and LLM generation).

---

## 4. Technical Explanation (Step-by-Step Trace)

### Step 1: File Upload and Loading
The user uploads a PDF file named `corporate_policy.pdf`.
```python
# The raw text is extracted page by page
page_1 = "Employee Handbook... Section A: Working Hours... Standard hours are 9 AM to 5 PM."
page_2 = "Section B: Vacation... All staff get 20 days off. Section C: Dress Code..."
```

### Step 2: Preprocessing and Cleaning
The text cleaner normalizes spaces, resolves hyphenated breaks, and cleans the text.
```python
# The text is standardized
cleaned_page_1 = "Employee Handbook. Section A: Working Hours. Standard hours are 9 AM to 5 PM."
```

### Step 3: Chunking
The text is split into chunks of 200 characters with a 40-character overlap.
```python
chunk_1 = "Employee Handbook. Section A: Working Hours. Standard hours are 9 AM to 5 PM."
metadata_1 = {"filename": "corporate_policy.pdf", "page_number": 1, "chunk_index": 0}
```

### Step 4: Embedding Generation
The text chunks are passed through the embedding model to generate coordinate vectors.
```python
vector_1 = embedding_model.embed(chunk_1)  # Yields [0.012, -0.45, 0.98, ...]
```

### Step 5: Vector DB Storage
The vector, raw text, and metadata are saved in the vector database.
```python
vector_db.add(id="corp_1_0", vector=vector_1, text=chunk_1, metadata=metadata_1)
```

### Step 6: User Query and Embedding
The user asks: *"What are the working hours?"* The query is embedded using the same model.
```python
query_vector = embedding_model.embed("What are the working hours?")
```

### Step 7: Similarity Search
The vector database calculates similarity scores between the query vector and all stored vectors.
```python
results = vector_db.search(query_vector, k=1)
# Returns chunk_1 with a similarity score of 0.88
```

### Step 8: Prompt Construction
The system formats the retrieved chunk and source metadata into the grounded prompt template.
```
Context:
--- START SOURCE [1] ---
Source File: corporate_policy.pdf (Page 1)
Content: Employee Handbook. Section A: Working Hours. Standard hours are 9 AM to 5 PM.
--- END SOURCE [1] ---

Question: What are the working hours?
Answer:
```

### Step 9: LLM Completion
The prompt is sent to Google Gemini (e.g. `gemini-1.5-flash`) with `temperature=0.0`.
```
Output:
"Standard working hours are 9 AM to 5 PM [1]."
```

---

## 5. Industry Best Practices
- **Log Request Identifiers**: Tag each query with a unique UUID to trace logs across database searches and LLM calls.
- **Trace Latency**: Measure and log the time taken by each step (load, clean, embed, query, generate) to identify bottlenecks.
- **Save Raw Uploads**: Keep copies of raw uploaded files in secure storage to allow re-ingestion if the database schema or chunking strategies change.

---

## 6. Common Mistakes
- **Neglecting to Log VectorDB Time vs LLM Time**: Guessing why a system is slow without checking whether the delay is in the vector database search or the LLM response generation.
- **Mismatched Text Encodings**: Not specifying UTF-8 encoding when loading or saving files, which can garble characters on Windows machines.

---

## 7. Visual Diagram (ASCII)
```
[User Document] -> [Load] -> [Clean] -> [Chunk] -> [Embed] -> [Store in DB]
                                                                    |
                                                                    v
[User Query] ----> [Embed] -> [Query DB] -> [Retrieve Chunk] ------>+
                                                   |
                                                   v
[Grounded Answer] <- [Gemini API] <- [Build Prompt] <- [Evaluate Grounding]
```

---

## 8. Interview Questions
1. **Walk through the full lifecycle of a user query in a RAG system.**
   *Answer*: The query is: (1) cleaned and embedded, (2) compared against the vector database index, (3) matched chunks are retrieved, (4) similarity scores are checked against a threshold, (5) context is constructed, (6) system prompts are applied, (7) the query is sent to the LLM (at temperature 0), and (8) the generated response is returned with citations.
2. **If the LLM response is cut off, which parameter in the pipeline should you adjust?**
   *Answer*: Adjust the `max_tokens` or `max_completion_tokens` parameter in the LLM API request config, or increase the context window size budget.

---

## 9. Key Takeaways
- A full RAG cycle links ingestion and retrieval.
- Latency should be measured and monitored at every step.
- Metadata is critical for tracing facts back to their source.
- Keeping temperature at 0.0 makes the system predictable and factual.

---

## 10. Beginner Summary
This note walks you through the journey of a document in the RAG system. It starts as a raw file, gets loaded, cleaned, sliced, converted to coordinates, and stored. When a user asks a question, the system finds the closest coordinates, grabs the corresponding text slices, and hands them to the AI to write a clear, grounded answer.
