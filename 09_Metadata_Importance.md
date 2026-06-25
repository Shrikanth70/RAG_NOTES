# Note 09: Metadata Importance (Tracking Context Sources)

## 1. Concept Overview
**Metadata** refers to the descriptive attributes attached to each text chunk stored in a vector database. 

While the *vector* represents the semantic meaning and the *chunk text* represents the raw content, the *metadata* acts as an identity tag. In a production-grade system, every chunk is stored with attributes such as:
- `filename`: The name of the source document.
- `page_number`: The specific page where the text resides.
- `chunk_id`: A unique hash representing the chunk.
- `source_type`: The file extension (e.g. `pdf` or `docx`).
- `timestamp`: The date and time the document was processed.

---

## 2. Why It Exists
Vector search is blind to file structure. When the system retrieves the top 3 matching chunks, it receives raw strings of text. Without metadata:
1. **No Citations**: The system cannot tell the user *where* it found the information. This makes the chatbot feel like a black box, reducing user trust.
2. **No Access Controls or Scope**: The database searches everything. If you want to search only documents uploaded in the last week, or only documents matching a specific category, you cannot do so.
3. **No Update Capacity**: If a file is deleted, you cannot locate and purge its corresponding chunks from the database.

---

## 3. Real-World Analogy
Imagine a researcher who cuts out important paragraphs from hundreds of newspapers and pastes them onto index cards. 
- **Without Metadata**: The cards are blank on the back. The researcher has a pile of quotes but doesn't know who wrote them, what date they were published, or which newspaper they came from.
- **With Metadata**: On the back of each card, the researcher writes: *"New York Times, Page A4, June 12, 2026"*. Now, the quotes are verifiable, credible, and easy to organize.

---

## 4. Technical Explanation
When storing a chunk in a database like Chroma or FAISS, the payload is structured as follows:

```json
{
  "id": "doc_hash_chunk_004",
  "vector": [0.043, -0.198, ..., 0.722],
  "document": "The product warranty is valid for two years from the date of purchase.",
  "metadata": {
    "filename": "warranty_policy_v2.pdf",
    "page_number": 4,
    "chunk_index": 3,
    "source_type": "pdf",
    "created_at": "2026-06-12T18:20:00Z"
  }
}
```

During retrieval, the query can include a metadata filter:

```python
# Query only files matching this filename
results = vector_db.query(
    query_vector=user_query_vector,
    k=3,
    filter={"filename": "warranty_policy_v2.pdf"}
)
```

---

## 5. Industry Best Practices
- **Deterministic IDs**: Generate IDs using hashes of the file content and chunk index. This makes the ingestion idempotent (running it twice overrides, not duplicates).
- **Index Filter Fields**: Ensure the vector database indexes metadata fields used for pre-filtering (like `filename` or `user_id`) to keep search times fast.
- **Keep Metadata Minimal**: Do not store massive amounts of structured text in metadata. Keep it to identification tags and small strings to save memory.

---

## 6. Common Mistakes
- **Neglecting to Cast Data Types**: Storing page numbers as strings (e.g., `"4"`) instead of integers (`4`). This prevents numeric filtering (like `page_number > 5`).
- **Storing User Credentials in Metadata**: Placing unencrypted sensitive user keys in metadata blocks that are cached or logged.

---

## 7. Visual Diagram (ASCII)
```
+-----------------------------------------------------------+
|                      STORED VECTOR RECORD                 |
+-----------------------------------------------------------+
|  [Vector Embeddings]                                      |
|    [0.12, -0.95, 0.43, ..., 0.08]                         |
+-----------------------------------------------------------+
|  [Raw Text Chunk]                                         |
|    "Section 12.3: Battery life is estimated at 8 hours."  |
+-----------------------------------------------------------+
|  [Metadata Identity Tag]                                  |
|    - filename: "manual_v1.pdf"                            |
|    - page_number: 14                                      |
|    - chunk_id: "m_14_sec12"                               |
|    - timestamp: 1718216400                                |
+-----------------------------------------------------------+
```

---

## 8. Interview Questions
1. **How does metadata help in multi-tenant RAG systems (systems with multiple users)?**
   *Answer*: Metadata allows storing a `user_id` or `tenant_id` alongside each vector. During retrieval, we apply a metadata pre-filter for that user's ID, ensuring they can only retrieve and view their own documents, preventing cross-user data leakage.
2. **Why is it recommended to generate deterministic chunk IDs?**
   *Answer*: If chunk IDs are random (e.g., UUIDs), re-uploading the same file creates duplicate entries in the database. If they are deterministic (e.g. SHA-256 hash of the filename + chunk index), re-uploading updates the existing entries, ensuring idempotency.

---

## 9. Key Takeaways
- Metadata is the "who, what, when, where" of a text chunk.
- It is stored alongside the raw text and vector embedding.
- It enables source citations, which increase trust.
- It enables query pre-filtering to scope search results.

---

## 10. Beginner Summary
Metadata is like a label on a jar. If you have jars of white powder (text chunks), you don't know if it's sugar, salt, or flour. Metadata is the label that says *"Sugar, bought from Tesco on Tuesday, Page 1 of recipe"*. It tells you exactly what the text is, where it came from, and how to use it safely.
