---
title: "How to take a full-page website screenshot in Python (lazy images, mobile, no cookie banners)"
description: "Full-page screenshots with Playwright that actually render lazy-loaded images, hide cookie banners and emulate mobile devices, plus the 16k-pixel limit."
permalink: /full-page-screenshot/
---

# How to take a full-page website screenshot in Python

`page.screenshot(full_page=True)` looks like a one-liner, but on real sites you get grey boxes where images should be, a cookie banner over the content, and failures on long mobile pages. Here's how to fix all three.

```python
import asyncio
from playwright.async_api import async_playwright

SCROLL = """async () => { for (let y = 0; y < document.documentElement.scrollHeight; y += 600) {
  window.scrollTo(0, y); await new Promise(r => setTimeout(r, 120)); } window.scrollTo(0, 0); }"""

async def shot(url, path):
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page(viewport={"width": 1440, "height": 900})
        await page.goto(url, wait_until="load")
        await page.add_style_tag(content="#onetrust-consent-sdk, #CybotCookiebotDialog, .cc-window { display: none !important; }")
        await page.evaluate(SCROLL)  # trigger lazy-loaded images
        await page.screenshot(path=path, full_page=True)
        await browser.close()

asyncio.run(shot("https://example.com", "example.png"))
print("saved example.png")
```

## Pitfalls

- **Lazy-loaded images:** modern sites load images only when scrolled into view. Scroll through the page before capturing, or you'll get placeholders.
- **Cookie banners:** hide known consent-platform containers (OneTrust, Cookiebot, Didomi, Quantcast…) with injected CSS.
- **The 16k limit:** Chromium can't capture beyond roughly 16,000 *device* pixels. On a 3× mobile emulation that's only about 5,300 CSS pixels, so clip accordingly or captures fail with "Unable to capture screenshot".
- **Never-idle pages:** analytics and chat widgets keep the network busy. Don't wait for `networkidle` forever; cap it.

## Skip the build

**[Website Screenshot](https://apify.com/signalfeed/website-screenshot)** handles all of this: full page or viewport, desktop, tablet and mobile presets, lazy images, cookie-banner hiding, PNG/JPEG and dark mode. Failed URLs are free. $3 per 1,000 screenshots. *(Built by the author of this guide.)*

[← Home](../)
