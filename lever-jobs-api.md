---
title: "Lever Jobs API: get all open jobs from any Lever job board (free, no key)"
description: "How to pull every open job from a company's Lever careers page using Lever's free public job-board endpoint, with code, pagination and gotchas."
permalink: /lever-jobs-api/
---

# Lever Jobs API: every open job from any Lever job board

Lever powers the careers pages of thousands of companies, and exposes a **free, public, keyless JSON endpoint** behind them. Here's how to use it, with the pitfalls we hit running it against real companies.

```bash
curl "https://api.lever.co/v0/postings/palantir?mode=json"
```

**Gotchas**
- EU customers live on `api.eu.lever.co`. Try it when the US host returns 404.
- `createdAt` is a millisecond timestamp. `salaryRange` (`min`, `max`, `currency`, `interval`) is present when the company publishes pay.
- The description is split across `descriptionPlain`, `lists[]` (requirements and so on, as HTML) and `additionalPlain`.

## Other job-board APIs

[Greenhouse](greenhouse-jobs-api.md) · [Ashby](ashby-jobs-api.md) · [Workday](workday-jobs-api.md) · [SmartRecruiters](smartrecruiters-jobs-api.md) · [Recruitee](recruitee-jobs-api.md) · [Workable](workable-jobs-api.md) · [All platforms](README.md)

## Don't want to build and maintain this?

Every gotcha above is handled in ready-made tools (disclosure: built by the author of this guide):

- **[ATS Jobs Scraper](https://apify.com/signalfeed/ats-jobs-scraper)**: all 7 platforms, auto-detect, one normalized format. $1 per 1,000 jobs.
- **[Tech & Startup Job Search](https://apify.com/signalfeed/tech-job-search)**: keyword search across 4,400+ company career pages, no company list needed.
- **[Hiring Companies Finder](https://apify.com/signalfeed/hiring-companies-finder)**: which companies are hiring for a role, ranked by matching openings.
- **[Salary Data from Job Postings](https://apify.com/signalfeed/job-salary-data)**: published pay ranges only.
- **Jobs API on RapidAPI**: `GET /company` and `GET /detect` over HTTP.

Apify has a free plan with monthly credit. *(Some links may be affiliate links; using them supports this guide at no cost to you.)*

