# Job Board APIs: get jobs from Greenhouse, Lever, Ashby, Workday & more (free, no API key)

Most companies host their careers page on an applicant tracking system (ATS). Seven of the most common ones expose **public, keyless JSON endpoints** that power those careers pages, so you can pull every open job straight from the employer, live, without scraping HTML.

This guide documents each endpoint with copy-paste examples, pagination rules, and the gotchas we hit while running them against thousands of real companies.

| ATS | Find the slug in | Endpoint | Pagination | Salary |
|---|---|---|---|---|
| [Greenhouse](greenhouse-jobs-api.md) | `boards.greenhouse.io/{slug}` | `GET boards-api.greenhouse.io/v1/boards/{slug}/jobs` | none (all jobs) | rarely |
| [Lever](lever-jobs-api.md) | `jobs.lever.co/{slug}` | `GET api.lever.co/v0/postings/{slug}?mode=json` | none | sometimes |
| [Ashby](ashby-jobs-api.md) | `jobs.ashbyhq.com/{slug}` | `GET api.ashbyhq.com/posting-api/job-board/{slug}` | none | **often** |
| [Workday](workday-jobs-api.md) | `{tenant}.wd{N}.myworkdayjobs.com/{site}` | `POST …/wday/cxs/{tenant}/{site}/jobs` | 20/page, max 2,000 | no |
| [SmartRecruiters](smartrecruiters-jobs-api.md) | `jobs.smartrecruiters.com/{slug}` | `GET api.smartrecruiters.com/v1/companies/{slug}/postings` | 100/page | no |
| [Recruitee](recruitee-jobs-api.md) | `{slug}.recruitee.com` | `GET {slug}.recruitee.com/api/offers/` | none | sometimes |
| [Workable](workable-jobs-api.md) | `apply.workable.com/{slug}` | `POST apply.workable.com/api/v3/accounts/{slug}/jobs` | token | no |

> **Be a good citizen:** these endpoints exist so careers pages and job boards can display postings. Cache results, keep request rates modest, link back to the original posting, and don't collect personal data.

---

## Greenhouse

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

## Lever

```bash
curl "https://api.lever.co/v0/postings/palantir?mode=json"
```

**Gotchas**
- EU customers live on `api.eu.lever.co`. Try it when the US host returns 404.
- `createdAt` is a millisecond timestamp. `salaryRange` (`min`, `max`, `currency`, `interval`) is present when the company publishes pay.
- The description is split across `descriptionPlain`, `lists[]` (requirements and so on, as HTML) and `additionalPlain`.

## Ashby

```bash
curl "https://api.ashbyhq.com/posting-api/job-board/ramp?includeCompensation=true"
```

**Gotchas**
- **`isRemote` is `true` for hybrid roles too.** Use `workplaceType` (`Remote` / `Hybrid` / `OnSite`) instead. In our tests ~45% of "isRemote" jobs were actually hybrid.
- With `includeCompensation=true`, `compensation.compensationTiers[].components[]` holds salary components. A posting can have **several tiers** (by level or location): take the min of `minValue` and max of `maxValue` across `Salary` components, or you'll report the wrong range.
- Filter out `isListed: false`.

## Workday

```bash
curl -X POST "https://nvidia.wd5.myworkdayjobs.com/wday/cxs/nvidia/NVIDIAExternalCareerSite/jobs" \
  -H "content-type: application/json" \
  -d '{"appliedFacets":{},"limit":20,"offset":0,"searchText":""}'
```

**Gotchas**
- You need the full careers URL: tenant (`nvidia`), shard (`wd5`) and site (`NVIDIAExternalCareerSite`). Names aren't unique, so slugs alone don't work.
- `limit` maxes at 20; `total` is **only returned on the first page**; results stop at **2,000**.
- List items have only relative dates (`"Posted 3 Days Ago"`) and `"6 Locations"`. Fetch `GET …/{site}{externalPath}` for the real `startDate`, all locations and the description.
- Bursts from cloud IPs get rate-limited with HTML error pages: limit concurrency (~4) and retry with backoff.

## SmartRecruiters

```bash
curl "https://api.smartrecruiters.com/v1/companies/BoschGroup/postings?limit=100&offset=0"
```

**Gotchas**
- Unknown companies return `200` with `totalFound: 0`, not 404.
- Descriptions need a second call to each posting's `ref` URL (`jobAd.sections`).
- Big employers have thousands of postings (Bosch: 5,000+). Fetch pages in parallel once you know `totalFound`.

## Recruitee

```bash
curl "https://bunq.recruitee.com/api/offers/"
```

**Gotcha:** dates look like `2026-10-27 12:11:34 UTC`, which isn't ISO 8601. Most date parsers reject them unless you normalize first.

## Workable

```bash
curl -X POST "https://apply.workable.com/api/v3/accounts/huggingface/jobs" \
  -H "content-type: application/json" -d '{"query":"","location":[],"department":[],"worktype":[],"remote":[]}'
```

**Gotchas**
- Paginate by sending `nextPage` back as `token`.
- Descriptions: `GET apply.workable.com/api/v2/accounts/{slug}/jobs/{shortcode}`. Company name: `GET …/api/v1/accounts/{slug}`.
- Prefer `workplace` (`remote`/`hybrid`/`on_site`) over the boolean `remote`.

---

## More guides

- [How to find companies hiring for a specific role](companies-hiring-for-a-role.md) (hiring signals for sales and recruiting)
- [How to get salary data from job postings](salary-data-from-job-postings.md)
- [How to extract every URL from a website's sitemap](extract-urls-from-sitemap.md)
- [How to convert RSS and Atom feeds to JSON](rss-to-json.md)
- [How to extract text from a PDF URL in Python](pdf-url-to-text.md)
- [How to convert a web page to clean Markdown for LLMs](web-page-to-markdown.md)
- [How to take a full-page website screenshot in Python](full-page-screenshot.md)
- [How to find which email provider a company uses (MX, SPF, DMARC)](email-provider-lookup.md)
- [How to detect a website's tech stack (CMS, analytics, CDN) in Python](website-tech-stack-lookup.md)
- [How to extract emails, phone numbers and social links from a website](extract-emails-from-website.md)

## Finding which ATS a company uses

Open the company's careers page and click a job. The URL usually reveals the ATS and slug (`job-boards.greenhouse.io/stripe/…`, `jobs.lever.co/palantir/…`). With only a name, try the slug against each endpoint above and keep the board with the most open jobs: the same slug can exist on several platforms as a stale or test board.

## Don't want to build and maintain this?

Every gotcha above is handled in ready-made tools (disclosure: built by the author of this guide):

- **[ATS Jobs Scraper](https://apify.com/signalfeed/ats-jobs-scraper)**: all 7 platforms, auto-detect, one normalized format. $1 per 1,000 jobs.
- **[Tech & Startup Job Search](https://apify.com/signalfeed/tech-job-search)**: keyword search across 4,400+ company career pages, no company list needed.
- **[Hiring Companies Finder](https://apify.com/signalfeed/hiring-companies-finder)**: which companies are hiring for a role, ranked by matching openings.
- **[Salary Data from Job Postings](https://apify.com/signalfeed/job-salary-data)**: published pay ranges only.
- **Jobs API on RapidAPI**: `GET /company` and `GET /detect` over HTTP.

Apify has a free plan with monthly credit. *(Some links may be affiliate links; using them supports this guide at no cost to you.)*

## License

Text: CC BY 4.0. Code snippets: MIT. Corrections welcome via pull request.
