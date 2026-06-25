# Note 14: Prompt Engineering for RAG

## 1. Concept Overview
**Prompt Engineering for RAG** is the practice of designing the system instructions and user inputs to ensure the LLM behaves as an accurate, fact-based synthesis engine rather than a creative writer.

In a RAG setting, the prompt must define:
1. **The Persona**: A precise, factual document assistant.
2. **The Rules**: Strict constraints prohibiting the use of external knowledge.
3. **The Data**: The formatted context retrieved from the database.
4. **The Task**: The user's query and the required output format (e.g. including citations).

---

## 2. Why It Exists
By default, LLMs are trained to be helpful and conversational, drawing from their entire pre-trained dataset. If you ask an LLM about your company's holiday policy without strict prompt engineering, it might answer based on general holiday policies it saw on the internet. 

Prompt engineering acts as a set of guardrails, narrowing the model's focus to *only* the pages provided in the prompt window.

---

## 3. Real-World Analogy
Imagine hiring a temporary assistant:
- **Without explicit instructions**: You ask them, *"What did John say in his email?"* The assistant guesses, makes assumptions, or answers based on emails they read at their *previous* job.
- **With strict instructions**: You say, *"Here are three emails on this desk. Read ONLY these three emails. Tell me what John said in them. If John did not send an email in this pile, say 'I cannot find it.' Do not guess."* Now the assistant operates with perfect precision.

---

## 4. Technical Explanation
A production RAG prompt is divided into clean blocks using clear delimiters:

```
[SYSTEM INSTRUCTION]
You are a factual AI assistant. Your task is to answer the user's question using ONLY the facts contained in the "Retrieved Context" section below.

Strict Constraints:
1. Do not use external knowledge or make assumptions.
2. If the context does not contain the answer, reply EXACTLY with:
   "I could not find this information in the provided documents."
3. Every claim in your answer must cite its source by including the source index in brackets, e.g. [1] or [2].

[RETIREVED CONTEXT]
--- START SOURCE [1] ---
Source: sales_report.pdf (Page 2)
Content: Q3 revenue grew by 15% to $4.2M.
--- END SOURCE [1] ---

--- START SOURCE [2] ---
Source: policy_doc.docx (Page 1)
Content: Sales bonuses are paid out in December.
--- END SOURCE [2] ---

[USER QUERY]
By how much did Q3 revenue grow, and when are bonuses paid?

[RESPONSE GENERATION STARTS HERE]
```

---

## 5. Industry Best Practices
- **Use Uppercase Constraints**: Use strong, capitalized words for rules (e.g., "ONLY", "EXACTLY", "DO NOT").
- **Enforce Fallback Phrasing**: Specify the *exact* string to output on failure. This makes it trivial for backend code to detect when a query could not be answered (e.g., to trigger a human handoff).
- **Zero-Shot works best**: Avoid overloading the prompt with dozens of examples (few-shot) unless your formatting is highly complex. A simple, direct system instruction is usually faster and cheaper.

---

## 6. Common Mistakes
- **Vague Constraints**: Saying *"Please try to use the context if possible."* The LLM will treat this as a suggestion and hallucinate when it gets stuck.
- **Putting the Query before the Context**: If the user query is placed at the very top of a long prompt, the LLM may forget the query by the time it finishes reading 5000 words of context. Always place the user query at the **bottom**, closest to where the LLM starts writing.

---

## 7. Visual Diagram (ASCII)
```
+-----------------------------------------------------------+
|                     RAG PROMPT ANATOMY                     |
+-----------------------------------------------------------+
|  1. System Rules (Constraint Guardrails)                  |
|     - "Answer ONLY using context..."                      |
|     - "If unknown, output 'I could not find...'"          |
+-----------------------------------------------------------+
|  2. Retrieved Context (Structured Documents)              |
|     - <doc id="1">Text...</doc>                            |
|     - <doc id="2">Text...</doc>                            |
+-----------------------------------------------------------+
|  3. User Query (The Question)                             |
|     - "What is the warranty period?"                      |
+-----------------------------------------------------------+
|  4. Output Anchor (Trigger Generation)                    |
|     - "Answer: "                                          |
+-----------------------------------------------------------+
```

---

## 8. Interview Questions
1. **How do you format a prompt to ensure the LLM cites its sources in its response?**
   *Answer*: In the system prompt, instruct the model to append source numbers in brackets (e.g., `[1]`, `[2]`) directly after the sentence containing the cited claim. Provide a simple example in the instructions to demonstrate the desired behavior.
2. **Why should the user query be placed after the retrieved context in the prompt template?**
   *Answer*: Placing the query at the bottom leverages the "recency bias" of LLMs. It ensures the question is the last thing the model reads before generating text, reducing the risk of it losing track of the user's intent when processing long context blocks.

---

## 9. Key Takeaways
- Prompt engineering for RAG enforces strict grounding constraints.
- System instructions must forbid assumptions and external knowledge.
- Clear delimiters (XML, Markdown) prevent text bleeding.
- The user query should always be placed at the end of the prompt.

---

## 10. Beginner Summary
Prompt engineering is like giving a chef strict instructions for a recipe. Instead of saying *"Cook a meal,"* you say: *"Use ONLY the ingredients in this basket. Do not add salt or spices from your cupboards. If there are not enough ingredients to make the soup, say 'I cannot cook this.'"* This guarantees the chef makes exactly what is in the basket.
