# Note 05: Text Cleaning and Preprocessing

## 1. Concept Overview
Text Cleaning is the preprocessing phase that standardizes raw extracted text. It removes formatting noise, corrects encoding errors, and strips non-semantic content (like control characters or page numbers in headers) while keeping the core semantic meaning intact.

Common operations include:
- **Whitespace Normalization**: Replacing multiple consecutive tabs/spaces/newlines with a single space or line break.
- **Unicode Normalization**: Converting characters like curved curly quotes (`“”`) to straight quotes (`""`) or combining decomposed Unicode symbols.
- **Noise Stripping**: Removing footers, headers, page numbers, and decorative elements (like `---` or lines).

---

## 2. Why It Exists
Raw parsers often output messy strings containing junk characters, page headers, footers, and erratic formatting (e.g., words broken by hyphens across lines like `sys- \n tem`). 

If you embed this messy text:
1. The quality of the embeddings declines because the embedding model gets confused by noise.
2. You waste valuable token space in your LLM prompt window by sending useless spaces and page headers.
3. Search matches degrade because formatting variations distort word semantics.

---

## 3. Real-World Analogy
Imagine receiving a transcript of a lecture where the typist repeatedly wrote *"um"*, *"ah"*, sneezed occasionally, and wrote down the page numbers at the top of every page. 

Before studying this transcript for an exam, you would cross out all the *"ums"*, clear out the sneezes, and erase the repetitive header text so that you can focus purely on the teacher's lesson. That is text cleaning.

---

## 4. Technical Explanation
A robust text cleaner applies regex rules and normalization forms sequentially:

```python
import re
import unicodedata

class TextCleaner:
    @staticmethod
    def clean(text: str) -> str:
        if not text:
            return ""
        
        # 1. Normalize Unicode characters
        text = unicodedata.normalize("NFKC", text)
        
        # 2. Fix hyphenated word breaks at line ends (e.g. "deve- \n lopment" -> "development")
        text = re.sub(r"(\w+)-\s*\n\s*(\w+)", r"\1\2", text)
        
        # 3. Standardize newlines (replace multiple empty lines with a single newline)
        text = re.sub(r"\n\s*\n+", "\n\n", text)
        
        # 4. Remove excessive horizontal spacing
        text = re.sub(r"[ \t]+", " ", text)
        
        # 5. Strip leading/trailing margins
        return text.strip()
```

---

## 5. Industry Best Practices
- **Preserve Sentence Boundaries**: Avoid stripping standard punctuation (`.`, `?`, `!`) or line breaks that represent paragraph shifts, as downstream chunkers need these boundaries.
- **Avoid Over-Cleaning**: Do not strip out specialized characters (like math symbols, punctuation, or code blocks) if your RAG system needs to answer technical or mathematical questions.
- **Log Cleaning Metrics**: Track the percentage of characters removed. A sudden drop of 50%+ characters could mean your cleaner is eating valid content.

---

## 6. Common Mistakes
- **Lowercasing Everything**: Lowercasing is standard in traditional search (like Elasticsearch), but modern neural embeddings and LLMs rely on case casing {sensing} (e.g. "US" the country vs "us" the pronoun) to understand meaning.
- **Stripping Code Indentations**: If you are ingesting code, stripping whitespace destroys Python scopes or nested blocks.

---

## 7. Visual Diagram (ASCII)
```
[Messy Raw String] 
  "The   system    com- \n ponents are  ready... Page 1"
          |
          v
  [Unicode NFKC Normalization]  (Normalizes characters)
          |
          v
  [Hyphen Resolver]             "com- \n ponents" -> "components"
          |
          v
  [Space Compressor]            "The   system" -> "The system"
          |
          v
[Clean Standardized Text]
  "The system components are ready... Page 1"
```

---

## 8. Interview Questions
1. **Why shouldn't you lowercase text before feeding it to an embedding model?**
   *Answer*: Modern embedding models are transformer-based and trained on case-sensitive text. Case provides critical context (e.g., distinguishing "Bush" the president from "bush" the plant). Lowercasing degrades embedding quality.
2. **What is the risk of over-cleaning text in a technical RAG system?**
   *Answer*: Over-cleaning can strip out code formatting, mathematical operators, or acronyms that carry high semantic density. This leads to poor retrieval accuracy on specialized technical topics.

---

## 9. Key Takeaways
- Text cleaning removes non-semantic noise (whitespace, line hyphenations, control characters).
- It improves similarity matching because clean text results in better vector embeddings.
- It saves LLM token costs by deleting useless formatting characters.
- Never strip case casing or sentence-ending punctuation.

---

## 10. Beginner Summary
Text cleaning is like cleaning a window. If the window is covered in mud (extra spaces, weird code characters, hyphen breaks), you can't see the view clearly. By wiping the window clean, the search engine and LLM can see the sentences perfectly and understand exactly what the document is trying to say.
