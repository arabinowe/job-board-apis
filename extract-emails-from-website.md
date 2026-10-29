---
title: "How to extract emails, phone numbers and social links from a website in Python"
description: "Pull contact emails, tel: phone numbers and company LinkedIn, X, Facebook and Instagram profiles from any website, while filtering out the junk (image filenames, tracking IDs, share buttons). Free code."
permalink: /extract-emails-from-website/
---

# How to extract emails, phone numbers and social links from a website

Finding a company's contact email and social profiles from its website is a core lead-generation task. A regex over the homepage gets you started, but it also returns image names like `logo@2x.png`, error-tracker IDs and share buttons. Here's a version that avoids those.

```python
import re, httpx
from urllib.parse import urljoin, urlparse

EMAIL = re.compile(r"[a-z0-9][a-z0-9._%+-]*@(?:[a-z0-9-]+\.)+[a-z]{2,24}", re.I)
JUNK_TLDS = {"png", "jpg", "jpeg", "gif", "webp", "svg", "css", "js"}
JUNK_DOMAINS = ("sentry.io", "wixpress.com", "example.com")
SOCIAL = {
    "linkedin": r"linkedin\.com/(company|in)/[^/?#\"']+",
    "twitter": r"(?:twitter|x)\.com/(?!intent|share)[A-Za-z0-9_]{1,15}(?=[/?#\"']|$)",
    "facebook": r"facebook\.com/(?!sharer|share|dialog|plugins)[^/?#\"']+",
    "instagram": r"instagram\.com/(?!p/|reel/)[A-Za-z0-9_.]+",
}

def page_contacts(html):
    emails = {e.lower() for e in EMAIL.findall(html)
              if e.rsplit(".", 1)[-1].lower() not in JUNK_TLDS and not e.lower().endswith(JUNK_DOMAINS)}
    phones = {re.sub(r"[^\d+]", "", t) for t in re.findall(r'href=["\']tel:([^"\']+)', html, re.I)}
    socials = {k: sorted({m.group(0) for m in re.finditer(r"https?://(?:www\.)?" + p, html, re.I)}) for k, p in SOCIAL.items()}
    return emails, phones, socials

def site_contacts(url):
    c = httpx.Client(follow_redirects=True, timeout=20, headers={"User-Agent": "contact-lookup/1.0"})
    home = c.get(url)
    emails, phones, socials = page_contacts(home.text)
    # Then check the contact/about pages, where emails usually live.
    links = {urljoin(str(home.url), h) for h in re.findall(r'href=["\']([^"\']+)', home.text)}
    for link in [l for l in links if re.search(r"contact|about|impressum", l, re.I)
                 and urlparse(l).hostname == home.url.host][:4]:
        e, p, _ = page_contacts(c.get(link).text)
        emails |= e; phones |= p
    return {"emails": sorted(emails), "phones": sorted(phones), "socials": socials}

print(site_contacts("https://plausible.io"))
```

## Pitfalls

- **Emails usually aren't on the homepage.** Crawl the contact, about and imprint (`/impressum` for German sites) pages too.
- **Filter the junk:** retina images (`hero@2x.png`), error-tracker DSNs (`...@sentry.io`, `sentry.wixpress.com`) and placeholders (`you@example.com`) all match email regexes.
- **Use `tel:` links for phones.** Regexing digits from page text picks up prices, dates and order numbers.
- **Share buttons aren't profiles.** Exclude `twitter.com/intent`, `facebook.com/sharer` and Instagram post links (`/p/`).
- **Team pages and testimonials pollute socials.** Customer quotes link to their authors' X accounts, and a page listing 30 employees' LinkedIn profiles isn't the company's account. Prefer profiles found on the homepage and ones whose handle matches the site name.
- **Obfuscated emails** like `info [at] acme [dot] com` need a separate pattern.
- **Respect the law:** only publicly published business contacts, and follow GDPR/CAN-SPAM when you email them.

## Need it for hundreds of websites?

**[Website Contact Extractor](https://apify.com/signalfeed/website-contact-extractor)** handles all of the above (obfuscation, team-page filtering, ranking the site's own inboxes first) and returns one clean row per website. $2.50 per 1,000 websites, and sites that fail to load are free. *(Built by the author of this guide.)*

See also: [detect a website's tech stack](website-tech-stack-lookup.md) · [find which email provider a company uses](email-provider-lookup.md)

[← Home](README.md)
