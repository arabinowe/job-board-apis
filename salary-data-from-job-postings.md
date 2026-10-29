---
title: "How to get salary data from job postings (free, from employers' own job boards)"
description: "Collect published salary ranges for any role from live job postings on Ashby, Lever and Recruitee job boards. Field names, code and the multi-tier pitfall."
permalink: /salary-data-from-job-postings/
---

# How to get salary data from job postings

Pay-transparency laws (New York, California, Colorado, Washington, and the EU Pay Transparency Directive) mean employers increasingly publish salary ranges in job posts. Company job boards expose that data as structured fields. Here's where it lives.

| ATS | Where salary lives | How often it's present |
|---|---|---|
| Ashby | `compensation.compensationTiers[].components[]` (`compensationType: "Salary"`, `minValue`, `maxValue`, `currencyCode`, `interval`) plus `compensation.scrapeableCompensationSalarySummary` | Very often for US roles |
| Lever | `salaryRange` (`min`, `max`, `currency`, `interval`) | Sometimes |
| Recruitee | `salary` (`min`, `max`, `currency`, `period`) | Sometimes |
| Greenhouse | Usually only inside the description text | Rarely structured |

## Ashby example (with the multi-tier fix)

```python
import httpx

jobs = httpx.get("https://api.ashbyhq.com/posting-api/job-board/ramp", params={"includeCompensation": "true"}).json()["jobs"]
for j in jobs:
    comp = j.get("compensation") or {}
    salary = [c for t in comp.get("compensationTiers", []) for c in t.get("components", []) if c.get("compensationType") == "Salary"]
    if not salary:
        continue
    lo = min(c["minValue"] for c in salary if c.get("minValue") is not None)
    hi = max(c["maxValue"] for c in salary if c.get("maxValue") is not None)
    print(j["title"], comp.get("scrapeableCompensationSalarySummary"), lo, hi, salary[0]["currencyCode"])
```

**The pitfall:** a posting can have several tiers (by level or location). Reading only the first tier reports the wrong range. In our tests, "$174K–$266K" postings read as $194K–$266K. Take the min and max across all salary components.

## Skip the build

**[Salary Data from Job Postings](https://apify.com/signalfeed/job-salary-data)** returns only jobs with published pay (min/max, currency, interval, plus the posted text) across 4,400+ company boards, filterable by role and location. $2 per 1,000 jobs. *(Built by the author of this guide.)*

[← All job-board APIs](../)
