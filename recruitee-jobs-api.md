---
title: "Recruitee Jobs API: get all open jobs from any Recruitee job board (free, no key)"
description: "How to pull every open job from a company's Recruitee careers page using Recruitee's free public job-board endpoint, with code, pagination and gotchas."
permalink: /recruitee-jobs-api/
---

# Recruitee Jobs API: every open job from any Recruitee job board

Recruitee powers the careers pages of thousands of companies, and exposes a **free, public, keyless JSON endpoint** behind them. Here's how to use it, with the pitfalls we hit running it against real companies.

```bash
curl "https://bunq.recruitee.com/api/offers/"
```

**Gotcha:** dates look like `2026-10-27 12:11:34 UTC`, which isn't ISO 8601. Most date parsers reject them unless you normalize first.

## Other job-board APIs

[Greenhouse](../greenhouse-jobs-api/) · [Lever](../lever-jobs-api/) · [Ashby](../ashby-jobs-api/) · [Workday](../workday-jobs-api/) · [SmartRecruiters](../smartrecruiters-jobs-api/) · [Workable](../workable-jobs-api/) · [All platforms](../)

## Don't want to build and maintain this?

Every gotcha above is handled in ready-made tools (disclosure: built by the author of this guide):

- **[ATS Jobs Scraper](https://apify.com/signalfeed/ats-jobs-scraper)**: all 7 platforms, auto-detect, one normalized format. $1 per 1,000 jobs.
- **[Tech & Startup Job Search](https://apify.com/signalfeed/tech-job-search)**: keyword search across 4,400+ company career pages, no company list needed.
- **[Hiring Companies Finder](https://apify.com/signalfeed/hiring-companies-finder)**: which companies are hiring for a role, ranked by matching openings.
- **[Salary Data from Job Postings](https://apify.com/signalfeed/job-salary-data)**: published pay ranges only.
- **Jobs API on RapidAPI**: `GET /company` and `GET /detect` over HTTP.

Apify has a free plan with monthly credit. *(Some links may be affiliate links; using them supports this guide at no cost to you.)*

