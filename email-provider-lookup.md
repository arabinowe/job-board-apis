---
title: "How to find which email provider a company uses (MX lookup, SPF and DMARC in Python)"
description: "Detect whether a domain uses Google Workspace, Microsoft 365, Proofpoint, Mimecast and more from its MX records, and check SPF and DMARC, with free DNS-over-HTTPS. Code included."
permalink: /email-provider-lookup/
---

# How to find which email provider a company uses

A domain's **MX records** reveal who handles its email: Google Workspace, Microsoft 365, or a security gateway like Proofpoint or Mimecast. Sales teams use this to segment accounts (sell a Microsoft 365 add-on to Microsoft shops), and IT and security teams use it with **SPF** and **DMARC** to audit email security.

```python
import httpx

def dns(name, rtype):
    r = httpx.get("https://cloudflare-dns.com/dns-query", params={"name": name, "type": rtype}, headers={"accept": "application/dns-json"})
    return [a["data"] for a in r.json().get("Answer", [])]

PROVIDERS = {"google.com": "Google Workspace", "outlook.com": "Microsoft 365", "pphosted.com": "Proofpoint",
             "mimecast": "Mimecast", "messagelabs.com": "Broadcom/Symantec", "zoho": "Zoho Mail"}

def email_setup(domain):
    mx = [m.split()[-1].rstrip(".") for m in dns(domain, "MX")]
    provider = next((name for key, name in PROVIDERS.items() for host in mx if key in host), "Other" if mx else None)
    spf = next((t.strip('"') for t in dns(domain, "TXT") if t.strip('"').startswith("v=spf1")), None)
    dmarc = next((t.strip('"') for t in dns(f"_dmarc.{domain}", "TXT")), None)
    return {"domain": domain, "mx": mx, "provider": provider, "spf": spf, "dmarc": dmarc}

print(email_setup("stripe.com"))
```

## Pitfalls

- **Security gateways hide the mailbox provider.** A domain behind Proofpoint or Mimecast may still use Microsoft 365 or Google underneath; the MX shows the gateway. SPF `include:` entries (`_spf.google.com`, `spf.protection.outlook.com`) often reveal the real provider.
- **Null MX:** an MX record of `0 .` (RFC 7505) means the domain accepts no email at all. Don't report it as "Other".
- **DMARC policy matters:** `p=none` only monitors; `quarantine` and `reject` actually protect against spoofing.

## Want registration data too?

**[Domain Intelligence](https://apify.com/signalfeed/domain-intelligence)** does this for any list of domains, plus official RDAP registration data: registrar, creation and expiry dates, days until expiry and nameservers. It recognizes 25+ email providers and gateways and flags null MX. $1 per 1,000 domains. *(Built by the author of this guide.)*

[← Home](README.md)
