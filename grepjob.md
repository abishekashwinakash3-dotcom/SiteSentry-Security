# Security & Compliance Report — GrepJob

- **Site:** https://grepjob.com/
- **Builder:** bolt
- **Scanned:** 2026-06-06T16:58:33.130Z
- **Score:** 5.5 / 10  (compliance 3.5 / 10 · security 7.5 / 10)
- **Content source:** fixture · **HTTP status:** 200

## Executive summary

GrepJob is in decent shape overall, but there are a few important gaps to address — especially around user privacy. Because the site collects analytics data (via PostHog) and appears to serve students, the top priorities are publishing a privacy policy and adding a consent notice before tracking begins. The remaining issues are mostly security headers and housekeeping that can be fixed in one focused afternoon.

## Findings (10)

_Remediation advice below is tailored to this site._

| Severity | Finding | How to fix |
|---|---|---|
| HIGH | No privacy policy found anywhere | Create a privacy policy page that explains you use PostHog analytics, what data it collects (page views, clicks, device info), and how users can opt out or request deletion. You can generate a solid starting draft at Termly (termly.io) or similar, then publish it as a page on GrepJob and add a 'Privacy Policy' link in your site footer. |
| MEDIUM | Analytics/tracking runs with no cookie-consent notice | PostHog is currently loading and tracking visitors before they've agreed to anything — add a cookie-consent banner that only activates PostHog after the user clicks 'Accept.' PostHog has a built-in consent mode (posthog.opt_in_capturing() / opt_out_capturing()) you can wire up to a simple banner; this is especially important given your student audience. |
| MEDIUM | Email address exposed in raw HTML | Replace the hardcoded email address in your page HTML with a contact form so bots can't scrape it and spam you. If you prefer a mailto link, use a simple JavaScript-based obfuscation or a service like Cloudflare's email obfuscation (free, enabled in the Cloudflare dashboard under Scrape Shield). |
| MEDIUM | Missing Content-Security-Policy header | Add a Content-Security-Policy header to control which scripts are allowed to run on GrepJob — this is your main protection against malicious script injection. Since Bolt projects are commonly deployed on Netlify or Vercel, add it in a netlify.toml [[headers]] block or a vercel.json headers config; start with a policy that allows your own domain plus posthog.com and tighten from there. |
| LOW | No terms of service linked | Add a short Terms of Service page covering what GrepJob is for, any rules around acceptable use, and a standard disclaimer about job listing accuracy. Link it next to your Privacy Policy in the footer; a free generator like Termly can produce a reasonable first draft in minutes. |
| LOW | Build-platform project UUID exposed in page source | Your Bolt project UUID is visible in the page source, which lets anyone identify your hosting platform and project. In your Bolt export, check for any meta tags or comments containing internal IDs and remove them before deploying — this is low-risk today but reduces unnecessary fingerprinting. |
| LOW | Missing X-Frame-Options / frame-ancestors | Add the header 'X-Frame-Options: DENY' to stop other sites from embedding GrepJob in an iframe (a clickjacking risk). In Netlify, add it to your netlify.toml under [[headers]] for '/*'; in Vercel, add it to the headers array in vercel.json — one line of config. |
| LOW | Missing Permissions-Policy header | Add a Permissions-Policy header to disable browser features your site doesn't need, like camera, microphone, and geolocation. A safe default value is 'Permissions-Policy: camera=(), microphone=(), geolocation=()' — add it alongside your other headers in your Netlify or Vercel config file. |
| LOW | Missing Referrer-Policy header | Add the header 'Referrer-Policy: strict-origin-when-cross-origin' so that when users click links leaving GrepJob, the full page URL isn't passed along to external sites. This is a one-line addition to your existing headers config in Netlify or Vercel. |
| INFO | No /.well-known/security.txt published | Optional: publish a security.txt so researchers know how to report vulnerabilities responsibly. |

## Evidence

- **No privacy policy found anywhere** — `{"privacyHref":null}`
- **Analytics/tracking runs with no cookie-consent notice** — `{"analytics":["PostHog"]}`
- **Email address exposed in raw HTML** — `{"emails":["k***@grepjob.com"]}`
- **Missing Content-Security-Policy header** — `{"observed":"absent"}`
- **No terms of service linked** — _(no extra detail)_
- **Build-platform project UUID exposed in page source** — `{"uuid":"c08d6b19-7b8f-4bc5-82e9-42052f46c6cb"}`
- **Missing X-Frame-Options / frame-ancestors** — `{"observed":"absent"}`
- **Missing Permissions-Policy header** — `{"observed":"absent"}`
- **Missing Referrer-Policy header** — `{"observed":"absent"}`
- **No /.well-known/security.txt published** — `{"checked":"/.well-known/security.txt"}`

---
_This assessment is based only on publicly available information returned by the site to a normal web request. No systems were accessed without authorization, and no active or intrusive testing was performed. Findings are provided in good faith to help improve the site's security and compliance posture._
