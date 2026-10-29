---
title: "How to convert a web page to clean Markdown for LLMs (Python, boilerplate removed)"
description: "Extract just the main content of any web page as Markdown, without menus, ads and footers, for RAG, summarization and AI agents. Python code with trafilatura."
permalink: /web-page-to-markdown/
---

# How to convert a web page to clean Markdown for LLMs

Feeding raw HTML (or naive HTML-to-text) into an LLM wastes tokens on navigation, cookie banners and footers, and it hurts answer quality. What you want is the **main content only**, as Markdown that keeps headings, lists and tables.

```python
import httpx, trafilatura

url = "https://en.wikipedia.org/wiki/Web_scraping"
html = httpx.get(url, headers={"User-Agent": "my-rag-bot/1.0 (contact@example.com)"}, follow_redirects=True).text
markdown = trafilatura.extract(html, url=url, output_format="markdown", include_tables=True, favor_precision=True)
meta = trafilatura.extract_metadata(html, default_url=url)
print(meta.title, "|", len(markdown.split()), "words")
print(markdown[:300])
```

## Pitfalls

- **Identify your bot honestly.** Many sites, Wikipedia included, return 403 to crawlers that fake a browser user agent. A descriptive UA with contact info works better and is polite.
- **Homepages aren't articles.** Index and app-like pages have little main content. Extract article, docs and post URLs instead.
- **JavaScript-only pages** return nothing without a headless browser. Detect empty results and route those URLs to a renderer.
- **Check the content type** before parsing: links to PDFs and images aren't HTML.

## Skip the build

**[Article & Main Content Extractor](https://apify.com/signalfeed/article-extractor)** turns any list of URLs into clean Markdown or text with title, author, date and word count, boilerplate removed. Failed URLs are free. $0.50 per 1,000 pages. *(Built by the author of this guide.)*

[← Home](README.md)
