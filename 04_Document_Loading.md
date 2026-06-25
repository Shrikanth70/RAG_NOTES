# Note 04: Document Loading (PDF & DOCX Parsing)

## 1. Concept Overview
Document Loading is the first stage of the RAG ingestion pipeline. It involves reading raw files in various formats (such as PDF and DOCX) and extracting their text content while tracking structural boundaries (like page numbers or sections).

Standard libraries like `pypdf` (for PDF) and `python-docx` (for Word documents) parse the underlying file structures (XML schemas, compressed binary streams) and return raw, unformatted text strings coupled with document metadata.

---

## 2. Why It Exists
LLMs cannot read raw file binaries (like a `.pdf` or `.docx` file) directly. These formats are designed for visual rendering or word processing editors, containing complex styling, zip compression, and metadata structures. 

Document loaders bridge this gap by stripping away the layout tags and styles, isolating the readable characters, and outputting clean plain text suitable for downstream natural language processing.

---

## 3. Real-World Analogy
Imagine receiving a letter written in secret code inside an ornate, heavy steel lockbox. 
- The lockbox is the **PDF format** (designed for security and layout containment).
- The document loader is the **key and locksmith** that opens the box, takes out the paper, reads the words out loud, and writes them down on a plain white index card.

---

## 4. Technical Explanation
Different document formats require separate extraction strategies:
- **PDF (Portable Document Format)**: PDFs organize text by coordinate layout objects on a canvas. Standard loaders like `pypdf` loop through page objects, extract characters, and join them with newline characters based on visual positioning.
- **DOCX (Office Open XML)**: A `.docx` file is actually a zipped archive containing XML files. The loader extracts the XML (specifically `document.xml`), parses paragraphs (`<w:p>`) and runs (`<w:r>`), and aggregates them into linear text blocks.

```python
# PDF Loading Example
from pypdf import PdfReader

def load_pdf(file_path: str) -> list[dict]:
    reader = PdfReader(file_path)
    pages = []
    for idx, page in enumerate(reader.pages):
        text = page.extract_text()
        pages.append({
            "text": text,
            "metadata": {"page_number": idx + 1, "source_type": "pdf"}
        })
    return pages
```

---

## 5. Industry Best Practices
- **Fallback Parsers**: If a PDF reader returns empty text (often due to scanned image PDFs), catch this and run a secondary Optical Character Recognition (OCR) engine like PyTesseract.
- **Extract Document Structure**: Keep track of titles, headings, and paragraph boundaries. Losing this structure makes chunking much harder.
- **Limit File Size**: Implement API limits on uploaded files to prevent memory exhaustion when parsing multi-gigabyte documents.

---

## 6. Common Mistakes
- **Assuming PDF Text is Always Selectable**: Many PDFs are scanned scans (images). Running standard text extractors on them yields an empty string.
- **Losing Page Margins and Line Break Meanings**: Some loaders fail to reconstruct columns properly, reading across two columns simultaneously. Always inspect your loader's raw text outputs.

---

## 7. Visual Diagram (ASCII)
```
[User PDF File]
       |
       v
+--------------+
| PDF Reader   | ---> Extracts page objects (Page 1, Page 2...)
+--------------+
       |
       v
+--------------+
| Layout Engine| ---> Converts spatial text coordinate objects to plain text string
+--------------+
       |
       v
"Page 1: System requirements consist of..." (Plain Text string)
```

---

## 8. Interview Questions
1. **What are the primary challenges when extracting text from PDF files?**
   *Answer*: PDFs store text as coordinates on a canvas, meaning reading order (like multi-column layouts) can be easily garbled. Other challenges include scanned images (which require OCR), embedded non-standard fonts, and embedded tables which lose structure when flattened.
2. **How do DOCX loaders work under the hood?**
   *Answer*: A DOCX file is a ZIP file containing XML files. Loaders extract this ZIP, locate `word/document.xml`, parse the XML nodes for paragraphs and text runs, and return them as plain strings.

---

## 9. Key Takeaways
- Loaders convert binary file formats (PDF, DOCX) into clean text strings.
- PDF extraction relies on coordinate positioning; DOCX extraction parses structured XML.
- Metadata (like page number) should be collected during the loading phase.
- Scanned documents require OCR, while native digital documents can be parsed programmatically.

---

## 10. Beginner Summary
Document loading is like copying text from a book into a notepad. Since computer programs can't "see" PDF files the way humans do, we use document loaders to open the file, extract all the sentences page by page, and save them as plain text that our code can read.
