---
title: "How to find companies hiring for a specific role (hiring signals from public job boards)"
description: "Build a list of companies actively hiring for any role, ranked by open positions, using free public ATS job-board endpoints. Code and pitfalls."
permalink: /companies-hiring-for-a-role/
---

# How to find companies hiring for a specific role

Hiring is one of the strongest buying signals in B2B sales. A company posting five data-engineer roles is about to buy data tools, and one hiring SDRs is building a sales team. Recruiting agencies use the same signal to find clients. Here's how to build that list yourself from public job boards.

## The method

1. **Build a company directory.** Collect careers-board slugs for Greenhouse (`boards.greenhouse.io/{slug}`), Lever (`jobs.lever.co/{slug}`) and Ashby (`jobs.ashbyhq.com/{slug}`). A good free source is [Common Crawl's URL index](https://index.commoncrawl.org/): query `url=jobs.lever.co&matchType=domain` and extract the first path segment. Then **verify each slug** against its API and drop empty boards. Roughly 75% of discovered slugs have open jobs.
2. **List each board cheaply.** For Greenhouse, call `boards-api.greenhouse.io/v1/boards/{slug}/jobs` *without* `content=true`. That returns titles and locations in kilobytes instead of megabytes. Lever and Ashby return full postings in one call.
3. **Match and aggregate per company.**

```python
import collections, httpx

def greenhouse_jobs(slug):
    r = httpx.get(f"https://boards-api.greenhouse.io/v1/boards/{slug}/jobs", timeout=30)
    return [{"title": j["title"], "location": (j.get("location") or {}).get("name")} for j in r.json().get("jobs", [])] if r.status_code == 200 else []

def hiring_summary(slug, keywords):
    jobs = greenhouse_jobs(slug)
    matched = [j for j in jobs if any(k in j["title"].lower() for k in keywords)]
    if not matched:
        return None
    return {"company": slug, "openJobs": len(jobs), "matchingJobs": len(matched),
            "topLocations": [l for l, _ in collections.Counter(j["location"] for j in matched).most_common(3)],
            "sampleTitles": [j["title"] for j in matched[:5]]}

print(hiring_summary("airbnb", ["engineer"]))
```

## Pitfalls

- **Check workplace type, not "remote" flags.** Ashby marks hybrid roles `isRemote: true`. Use `workplaceType`.
- **Stale duplicate boards.** The same slug can exist on several ATSs (a test board with 1 job). Keep the largest.
- **Scale.** Thousands of boards means thousands of requests. Use concurrency (~30) with retries, and expect a full pass to take a couple of minutes.

## Skip the build

**[Hiring Companies Finder](https://apify.com/signalfeed/hiring-companies-finder)** does all of this over a verified directory of 4,400+ companies: type a role and location, and get one row per hiring company with open and matching jobs, remote share, top locations and departments, and sample titles. $10 per 1,000 companies. *(Built by the author of this guide.)*

[← All job-board APIs](README.md)
