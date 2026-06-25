# Note 02: RAG Architecture (The Two-Phase System)

## 1. Concept Overview
A production RAG system is split into two distinct, decoupled pipelines:
1. **The Ingestion Pipeline (Phase 1 - Offline/Asynchronous)**: Responsible for taking raw files (PDFs, Word documents), extracting their text, cleaning it, breaking it into small pieces (chunks), creating mathematical representations (embeddings), and storing them in a searchable database.
2. **The Retrieval Pipeline (Phase 2 - Online/Real-time)**: Responsible for receiving a user query, converting it to an embedding, searching the database for matching chunks, stuffing those chunks into a prompt, and prompting the LLM to generate a response.

---

## 2. Why It Exists
This two-phase architecture is required for performance, cost-efficiency, and user experience:
- **Scalability**: Processing huge PDFs takes time and computing power. By doing this "offline" during ingestion, the "online" retrieval step is extremely fast (fractions of a second).
- **Decoupling**: You can modify the LLM model used for generation without re-processing or re-embedding all your documents. Similarly, you can swap your vector database without rebuilding the loader or LLM connections.

---

## 3. Real-World Analogy
Think of a premium restaurant:
- **Phase 1 (Ingestion)**: The prep kitchen. Chefs wash vegetables, chop meat, prepare sauces, and store them in labeled bins (databases) in the cold room. This is done before the restaurant opens.
- **Phase 2 (Retrieval)**: The line order service. A customer orders a dish (query). The waiter retrieves the pre-chopped ingredients from the bins, combines them in a pan, cooks them, and serves the hot plate to the customer immediately.

---

## 4. Technical Explanation
The entire architecture maps to the following component flow:

```
[ OFFLINE INGESTION PHASE ]
Docs (PDF/DOCX) -> [Loader] -> Raw Text -> [Cleaner] -> Clean Text -> [Chunker] 
    -> Small Chunks -> [Embedder] -> Vectors -> [Vector DB Storage]

[ ONLINE RETRIEVAL PHASE ]
User Query -> [Embedder] -> Query Vector -> [Vector DB Search] -> Top-K Chunks 
    -> [Prompt Assembly] -> Grounded Prompt -> [LLM Generator] -> Answer
```

Components interact through standardized interfaces (e.g., Abstract Classes for Vector Stores and Embeddings) to make the code highly extensible.

---

## 5. Industry Best Practices
- **Keep Databases Separate**: Separate text storage (metadata) from vector storage (indexes) to make updates or schema changes simple.
- **Use Configurations**: Control chunk sizes, overlaps, and models via settings (e.g., a `.env` file) rather than hardcoding.
- **Asynchronous Ingestion**: In production, document ingestion should run in the background (using message queues like Celery or BullMQ) to avoid blocking the main API thread.

---

## 6. Common Mistakes
- **Tightly Coupling Embeddings to the LLM Client**: Thinking that if you use Gemini for chat, you must use Gemini for embeddings. They are entirely separate processes; using local embeddings saves money and reduces external network dependencies.
- **Neglecting Ingestion Failures**: Files can be corrupt or fail to parse. Always write comprehensive exception handling and logging for every document ingested.

---

## 7. Visual Diagram (ASCII)
```
+------------------ PHASE 1: INGESTION -------------------+
| Raw Files -> [Load] -> [Clean] -> [Chunk] -> [Store]   |
+---------------------------------------------------------+
                                 |
                                 v
                     +-----------------------+
                     | Vector DB / Knowledge |
                     +-----------------------+
                                 ^
                                 | Query & Retrieve
+------------------ PHASE 2: RETRIEVAL -------------------+
| User Query -> [Similarity Search] -> [Generate] -> User |
+---------------------------------------------------------+
```

---

## 8. Interview Questions
1. **Why is it important to separate the Ingestion and Retrieval pipelines in a RAG system?**
   *Answer*: Separation of concerns allows for asynchronous preprocessing of large documents, which saves runtime latency during client queries. It also allows developers to upgrade or scale the ingestion workers independently of the query API.
2. **If we decide to change our LLM provider from OpenAI to Anthropic, do we need to re-ingest our documents?**
   *Answer*: No. The LLM only acts as a text generator in Phase 2. Since the ingestion pipeline generates vector embeddings which depend on the embedding model (e.g. HuggingFace), changing the generator LLM has no impact on stored embeddings.

---

## 9. Key Takeaways
- The RAG architecture has two phases: Ingestion (Prep) and Retrieval (Serve).
- They communicate through the shared Vector Database.
- Phase 1 runs once per document upload; Phase 2 runs on every user question.
- Decoupling these phases is a core software engineering principle for production stability.

---

## 10. Beginner Summary
Think of RAG as building an index card library. In Phase 1, you read large books, summarize them into small index cards, and file them neatly in a cabinet. In Phase 2, when someone asks a question, you search the cabinet for the best index cards and read them to write a letter back to the user.
