# Note 13: Context Construction (Assembling the Prompt)

## 1. Concept Overview
**Context Construction** is the step in the retrieval pipeline where retrieved text chunks and their associated metadata are compiled into a single structured string. This string is then injected into the system prompt template sent to the LLM.

Rather than dumping raw text into the model, a structured context builder isolates chunks using standard boundary tags, appends source labels, sorts them by relevance, and fits them within the model's token limits.

---

## 2. Why It Exists
An LLM needs clear boundaries to understand where your document content stops and where the user's question begins. Without structured context construction:
- **Source Confusion**: The LLM might mix up information from Document A and Document B.
- **Lost Citations**: The LLM won't know which chunk came from which page, making it impossible to output reliable citations.
- **Context Exhaustion**: If the retrieved chunks are larger than the LLM's allowed input window, the API call will crash. The builder must actively monitor token counts and truncate safely if needed.

> [!NOTE]
> **Context Quality > LLM Size**: Studies show that a smaller, well-grounded LLM (like an 8B Llama model) provided with clean, highly relevant context will dramatically outperform a massive 400B model provided with messy, irrelevant context.

---

## 3. Real-World Analogy
Imagine a lawyer preparing a case for a judge:
- **Poor Context Construction**: The lawyer dumps a giant box of loose papers, receipts, and diaries onto the judge's desk and says, *"The proof is in there somewhere, read it."* The judge gets confused and makes a poor decision.
- **Structured Context Construction**: The lawyer organizes the files into clear folders: *"Exhibit A (Contract, Page 2)"*, *"Exhibit B (Email, Page 1)"*. The lawyer highlights the key sentences and presents a neat index. The judge makes a fast, correct decision.

---

## 4. Technical Explanation
A robust context builder loops through the retrieved chunks and formats them using clean string templates:

```python
def build_context_string(retrieved_chunks: list[dict]) -> str:
    context_parts = []
    
    for idx, chunk in enumerate(retrieved_chunks):
        # Extract fields
        text = chunk["text"]
        filename = chunk["metadata"].get("filename", "Unknown File")
        page = chunk["metadata"].get("page_number", "N/A")
        chunk_id = chunk.get("chunk_id", idx)
        
        # Build block with explicit structural boundaries
        block = (
            f"--- START SOURCE [{idx + 1}] ---\n"
            f"Source File: {filename}\n"
            f"Page Number: {page}\n"
            f"Chunk ID: {chunk_id}\n"
            f"Content:\n{text}\n"
            f"--- END SOURCE [{idx + 1}] ---"
        )
        context_parts.append(block)
        
    return "\n\n".join(context_parts)
```

This formatted string is then interpolated into the system prompt:
`prompt = system_template.format(context=context_string, query=user_query)`

---

## 5. Industry Best Practices
- **Sort by Relevance**: Place the most similar chunks (highest similarity scores) at the very top and very bottom of the context block. Avoid placing crucial facts in the middle.
- **Use XML or Markdown Delimiters**: XML tags (e.g. `<doc id="1">...</doc>`) or markdown blockquotes work extremely well because LLMs are pre-trained on code and structured markup.
- **Implement Token Counting**: Use a library like `tiktoken` or `huggingface-tokenizers` to calculate the exact token size of the constructed context before sending it to the API.

---

## 6. Common Mistakes
- **Concatenating Chunks Raw**: Joining chunks without separator tokens: `chunk1_text + chunk2_text`. The LLM reads this as a single run-on sentence, causing confusion.
- **Wasting Space with System Boilerplate**: Using massive system prompts filled with repetitive rules, which leaves less room for actual document context.

---

## 7. Visual Diagram (ASCII)
```
+-----------------------------------------------------------+
|                   CONSTRUCTED CONTEXT BLOCK               |
+-----------------------------------------------------------+
|  --- START SOURCE [1] ---                                 |
|  Source File: employee_handbook.pdf                       |
|  Page Number: 12                                          |
|  Content: Employees receive 20 days of paid vacation.     |
|  --- END SOURCE [1] ---                                   |
|                                                           |
|  --- START SOURCE [2] ---                                 |
|  Source File: benefits_summary.docx                       |
|  Page Number: 1                                           |
|  Content: Vacation requests require 2 weeks notice.       |
|  --- END SOURCE [2] ---                                   |
+-----------------------------------------------------------+
```

---

## 8. Interview Questions
1. **Why is it important to wrap retrieved chunks in distinct boundaries (e.g., XML tags or Markdown headers)?**
   *Answer*: LLMs are trained to recognize structural patterns. Separators prevent the LLM from merging different sources together, make it easier for the model to refer to specific source IDs, and help it distinguish document knowledge from the user's conversational text.
2. **If your LLM context window is limited, how do you handle a situation where K retrieved chunks exceed this limit?**
   *Answer*: You must implement a token budget manager. Before formatting, count the tokens of each chunk. Add them one-by-one starting with the highest similarity score, and stop adding chunks once the token budget is reached.

---

## 9. Key Takeaways
- Context construction packages search results for the LLM.
- Clear structural tags (like `--- START SOURCE ---`) prevent context blending.
- Well-constructed context is more important than having a giant LLM.
- The context builder must manage token budgets to prevent API overflows.

---

## 10. Beginner Summary
Context construction is like organizing evidence for a court case. Instead of dumping folders on the judge's desk, you put dividers between the pages, write the source name on top of each page, and number them. This allows the judge (the LLM) to read the evidence clearly and state exactly where they found the proof.
