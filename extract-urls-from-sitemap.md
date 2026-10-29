---
title: "How to extract every URL from a website's sitemap (Python, with sitemap index and gzip)"
description: "Find a site's sitemaps via robots.txt, follow sitemap indexes, handle .gz and text sitemaps, filter by lastmod, safely. Code included."
permalink: /extract-urls-from-sitemap/
---

# How to extract every URL from a website's sitemap

Sitemaps list the pages a site wants crawled, which makes them the fastest way to inventory a site for SEO audits, content monitoring, migrations or seeding a crawler or RAG pipeline.

## Steps

1. **Find the sitemaps:** read `https://site/robots.txt` and collect every `Sitemap:` line. If there are none, try `/sitemap.xml`, `/sitemap_index.xml`, `/wp-sitemap.xml`.
2. **Parse safely:** sitemaps are XML from the internet. Use a hardened parser (`defusedxml`) so a malicious file can't trigger external-entity attacks.
3. **Recurse:** a `<sitemapindex>` lists more sitemaps; a `<urlset>` lists pages.
4. **Handle gzip and text:** `.gz` files (check the `1f 8b` magic bytes, not just the extension) and plain-text sitemaps (one URL per line).

```python
import gzip, httpx
from defusedxml import ElementTree as ET

def sitemap_urls(site):
    robots = httpx.get(f"https://{site}/robots.txt", follow_redirects=True).text
    queue = [l.split(":", 1)[1].strip() for l in robots.splitlines() if l.lower().startswith("sitemap:")] or [f"https://{site}/sitemap.xml"]
    seen, urls = set(), []
    while queue:
        sm = queue.pop()
        if sm in seen:
            continue
        seen.add(sm)
        body = httpx.get(sm, follow_redirects=True).content
        if body[:2] == b"\x1f\x8b":
            body = gzip.decompress(body)
        root = ET.fromstring(body)
        locs = [e.text.strip() for e in root.iter() if e.tag.endswith("loc") and e.text]
        (queue if root.tag.endswith("sitemapindex") else urls).extend(locs)
    return urls

print(len(sitemap_urls("stripe.com")))
```

## Pitfalls

- **Dead sitemap links:** some robots.txt or CMS links 404. Fall back to discovery rather than failing.
- **Missing `lastmod`:** many large sites (Stripe, for example) omit it entirely, so date filters can't apply. Count and report what you skipped.

## Skip the build

**[Sitemap URL Extractor](https://apify.com/signalfeed/sitemap-url-extractor)** handles discovery, indexes, gzip, text sitemaps, lastmod and regex filters, and optional HTTP status checks to find broken pages. $0.30 per 1,000 URLs. *(Built by the author of this guide.)*

[← Home](../)
