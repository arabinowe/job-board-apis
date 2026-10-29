---
title: "How to extract text from a PDF URL in Python (page by page, with metadata)"
description: "Download a PDF from a URL and extract clean text per page with pypdf: page ranges, metadata, scanned-PDF detection and common pitfalls."
permalink: /pdf-url-to-text/
---

# How to extract text from a PDF URL in Python

Whether you're feeding research papers into a RAG pipeline, monitoring regulatory filings, or parsing invoices, the first step is the same: turn a PDF link into clean text, ideally page by page so you can cite and chunk it.

```python
import io, httpx
from pypdf import PdfReader

body = httpx.get("https://arxiv.org/pdf/1706.03762", follow_redirects=True).content
assert body.lstrip()[:5].startswith(b"%PDF"), "not a PDF"
reader = PdfReader(io.BytesIO(body))
if reader.is_encrypted:
    reader.decrypt("")  # opens PDFs that only have an empty owner password
pages = [(i + 1, (p.extract_text() or "").strip()) for i, p in enumerate(reader.pages)]
print(len(pages), "pages;", pages[0][1][:120])
```

## Pitfalls

- **Check it's really a PDF:** many "PDF links" return an HTML landing page. Test for the `%PDF` header before parsing.
- **Scanned documents:** if every page returns empty text, the PDF is images only and needs OCR. Detect this and report it instead of returning blanks.
- **Clean the text:** strip NUL characters and soft hyphens, and collapse runs of spaces and blank lines.
- **Big files:** cap download size and process page by page.

## Skip the build

**[PDF Text Extractor](https://apify.com/signalfeed/pdf-text-extractor)** does this for any list of PDF URLs: one clean row per page with title, author and page count, page ranges, and scanned-PDF detection. $0.50 per 1,000 pages. *(Built by the author of this guide.)*

[← Home](README.md)
