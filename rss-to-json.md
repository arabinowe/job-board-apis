---
title: "How to convert RSS and Atom feeds to JSON (and find a site's feed automatically)"
description: "Parse RSS, Atom and RDF feeds into clean JSON with dates in ISO 8601, and auto-discover a website's feed from its HTML. Python code included."
permalink: /rss-to-json/
---

# How to convert RSS and Atom feeds to JSON

RSS and Atom are still the cleanest way to monitor blogs, news sites, changelogs and podcasts, and they make great input for AI summarizers and alerts. Here's how to turn any feed, or any website, into uniform JSON.

## Find a site's feed

Look in the homepage HTML for `<link rel="alternate" type="application/rss+xml" href="…">` (or `application/atom+xml`). If there isn't one, try common paths: `/feed`, `/rss`, `/rss.xml`, `/feed.xml`, `/atom.xml`, `/index.xml`.

## Parse any feed format

```python
import calendar, datetime, feedparser

feed = feedparser.parse("https://hnrss.org/frontpage")
items = [{
    "title": e.get("title"),
    "link": e.get("link"),
    "published": datetime.datetime.fromtimestamp(calendar.timegm(e.published_parsed), datetime.timezone.utc).isoformat()
                 if e.get("published_parsed") else None,
    "summary": e.get("summary"),
    "categories": [t.term for t in e.get("tags", [])],
} for e in feed.entries]
print(items[0])
```

## Pitfalls

- **Dates:** feeds use RFC 822, ISO 8601 or worse. Normalize to UTC ISO 8601 before filtering.
- **HTML in summaries:** strip tags and decode entities for clean text.
- **Relative links and duplicates:** resolve links against the feed URL, and de-duplicate by `guid` across feeds.

## Skip the build

**[RSS & Atom Feed Reader](https://apify.com/signalfeed/rss-feed-reader)** accepts feed URLs *or* plain website addresses (feeds auto-discovered) and returns clean JSON with date and keyword filters, ready to schedule and send to Slack, Sheets or an AI agent. $0.50 per 1,000 items. *(Built by the author of this guide.)*

[← Home](README.md)
