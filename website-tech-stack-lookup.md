---
title: "How to detect a website's tech stack in Python (CMS, analytics, CDN) without false positives"
description: "Find out whether a site runs WordPress, Shopify, Webflow, Next.js, HubSpot, Cloudflare and more from its HTML and HTTP headers. Free Python code and the pitfalls that cause false positives."
permalink: /website-tech-stack-lookup/
---

# How to detect a website's tech stack

Knowing that a prospect runs Shopify, HubSpot or WordPress is useful for sales targeting, competitor research and agency outreach. You don't need a paid tool for the basics: most technologies leave **distinctive fingerprints** in the page HTML, script URLs and HTTP response headers.

```python
import re, httpx

RULES = [  # (name, where, pattern)
    ("WordPress", "html", r"/wp-content/|/wp-includes/"),
    ("Shopify", "html", r"cdn\.shopify\.com"),
    ("Webflow", "html", r"data-wf-site="),
    ("Next.js", "html", r'id="__NEXT_DATA__"|/_next/static/'),
    ("HubSpot", "script", r"js\.hs-scripts\.com|js\.hsforms\.net"),
    ("Google Tag Manager", "html", r"googletagmanager\.com/gtm\.js"),
    ("Stripe", "script", r"js\.stripe\.com"),
    ("Cloudflare", "header:cf-ray", r"."),
    ("Vercel", "header:x-vercel-id", r"."),
]

def tech_stack(domain):
    r = httpx.get(f"https://{domain}/", follow_redirects=True, timeout=20,
                  headers={"User-Agent": "tech-lookup/1.0"})
    html, headers = r.text, {k.lower(): v for k, v in r.headers.items()}
    scripts = " ".join(re.findall(r"<script[^>]+src=[\"']([^\"']+)", html, re.I))
    found = []
    for name, where, pattern in RULES:
        if where.startswith("header:"):
            text = headers.get(where.split(":", 1)[1], "")
        else:
            text = scripts if where == "script" else html
        if text and re.search(pattern, text, re.I):
            found.append(name)
    return found

print(tech_stack("stripe.com"))
```

## Pitfalls that cause false positives

- **Mentions are not usage.** A blog post that says "WooCommerce" doesn't mean the site runs it. Match asset paths (`/wp-content/plugins/woocommerce/`) and script `src` URLs, never bare product names.
- **Match scripts by `src`, not anywhere in the HTML.** Inline text, JSON blobs and comparison pages mention competitors constantly. Restrict vendor-script rules to `<script src>` values.
- **Headers are the most reliable signal** for hosting and CDN (`cf-ray` for Cloudflare, `x-vercel-id`, `x-nf-request-id` for Netlify, `x-amz-cf-id` for CloudFront).
- **Try `www.` too.** Some apex domains only redirect or return an error page; fetch `https://www.{domain}/` before concluding nothing was found.
- **JavaScript-rendered sites** may load trackers after page load. HTML fingerprints still catch frameworks (Next.js, Nuxt, Gatsby) reliably.

## Need it for a list of domains?

**[Domain Intelligence](https://apify.com/signalfeed/domain-intelligence)** checks any list of domains for 40+ technologies with the high-precision rules above, plus registrar and expiry dates (RDAP), DNS, email provider and SPF/DMARC. $1 per 1,000 domains. *(Built by the author of this guide.)*

See also: [find which email provider a company uses](../email-provider-lookup/).

[← Home](../)
