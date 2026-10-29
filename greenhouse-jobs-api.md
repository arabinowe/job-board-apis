---
title: "Greenhouse Jobs API: get all open jobs from any Greenhouse job board (free, no key)"
description: "How to pull every open job from a company's Greenhouse careers page using Greenhouse's free public job-board endpoint, with code, pagination and gotchas."
permalink: /greenhouse-jobs-api/
---

# Greenhouse Jobs API: every open job from any Greenhouse job board

Greenhouse powers the careers pages of thousands of companies, and exposes a **free, public, keyless JSON endpoint** behind them. Here's how to use it, with the pitfalls we hit running it against real companies.

```bash
curl "https://boards-api.greenhouse.io/v1/boards/airbnb/jobs?content=true"
```

```python
import html, httpx
jobs = httpx.get("https://boards-api.greenhouse.io/v1/boards/airbnb/jobs", params={"content": "true"}).json()["jobs"]
for j in jobs:
    print(j["title"], "|", j["location"]["name"], "|", j["absolute_url"])
    description_html = html.unescape(j["content"])  # content is entity-escaped HTML
```

**Gotchas**
- `content=true` includes full descriptions but makes large boards several MB (Airbnb is ~2.7 MB). Omit it to list titles cheaply, then fetch one job with `…/jobs/{id}`.
- `content` is **double-encoded** (`&lt;p&gt;`): `html.unescape` it before parsing.
- `absolute_url` often points at the company's own careers site, which may block bots; the Greenhouse job URL redirects there too.

## Other job-board APIs

[Lever](lever-jobs-api.md) · [Ashby](ashby-jobs-api.md) · [Workday](workday-jobs-api.md) · [SmartRecruiters](smartrecruiters-jobs-api.md) · [Recruitee](recruitee-jobs-api.md) · [Workable](workable-jobs-api.md) · [All platforms](README.md)

## Don't want to build and maintain this?

Every gotcha above is handled in ready-made tools (disclosure: built by the author of this guide):

- **[ATS Jobs Scraper](https://apify.com/signalfeed/ats-jobs-scraper)**: all 7 platforms, auto-detect, one normalized format. $1 per 1,000 jobs.
- **[Tech & Startup Job Search](https://apify.com/signalfeed/tech-job-search)**: keyword search across 4,400+ company career pages, no company list needed.
- **[Hiring Companies Finder](https://apify.com/signalfeed/hiring-companies-finder)**: which companies are hiring for a role, ranked by matching openings.
- **[Salary Data from Job Postings](https://apify.com/signalfeed/job-salary-data)**: published pay ranges only.
- **Jobs API on RapidAPI**: `GET /company` and `GET /detect` over HTTP.

Apify has a free plan with monthly credit. *(Some links may be affiliate links; using them supports this guide at no cost to you.)*

