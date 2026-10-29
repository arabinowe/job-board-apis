---
title: "Ashby Jobs API: get all open jobs from any Ashby job board (free, no key)"
description: "How to pull every open job from a company's Ashby careers page using Ashby's free public job-board endpoint, with code, pagination and gotchas."
permalink: /ashby-jobs-api/
---

# Ashby Jobs API: every open job from any Ashby job board

Ashby powers the careers pages of thousands of companies, and exposes a **free, public, keyless JSON endpoint** behind them. Here's how to use it, with the pitfalls we hit running it against real companies.

```bash
curl "https://api.ashbyhq.com/posting-api/job-board/ramp?includeCompensation=true"
```

**Gotchas**
- **`isRemote` is `true` for hybrid roles too.** Use `workplaceType` (`Remote` / `Hybrid` / `OnSite`) instead. In our tests ~45% of "isRemote" jobs were actually hybrid.
- With `includeCompensation=true`, `compensation.compensationTiers[].components[]` holds salary components. A posting can have **several tiers** (by level or location): take the min of `minValue` and max of `maxValue` across `Salary` components, or you'll report the wrong range.
- Filter out `isListed: false`.

## Other job-board APIs

[Greenhouse](greenhouse-jobs-api.md) · [Lever](lever-jobs-api.md) · [Workday](workday-jobs-api.md) · [SmartRecruiters](smartrecruiters-jobs-api.md) · [Recruitee](recruitee-jobs-api.md) · [Workable](workable-jobs-api.md) · [All platforms](README.md)

## Don't want to build and maintain this?

Every gotcha above is handled in ready-made tools (disclosure: built by the author of this guide):

- **[ATS Jobs Scraper](https://apify.com/signalfeed/ats-jobs-scraper)**: all 7 platforms, auto-detect, one normalized format. $1 per 1,000 jobs.
- **[Tech & Startup Job Search](https://apify.com/signalfeed/tech-job-search)**: keyword search across 4,400+ company career pages, no company list needed.
- **[Hiring Companies Finder](https://apify.com/signalfeed/hiring-companies-finder)**: which companies are hiring for a role, ranked by matching openings.
- **[Salary Data from Job Postings](https://apify.com/signalfeed/job-salary-data)**: published pay ranges only.
- **Jobs API on RapidAPI**: `GET /company` and `GET /detect` over HTTP.

Apify has a free plan with monthly credit. *(Some links may be affiliate links; using them supports this guide at no cost to you.)*

