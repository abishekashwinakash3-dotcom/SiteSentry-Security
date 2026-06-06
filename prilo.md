# Security & Compliance Report — Prilo

- **Site:** https://www.prilo.ai/
- **Builder:** bolt
- **Scanned:** 2026-06-06T16:56:31.508Z
- **Score:** 5 / 10  (compliance 2.5 / 10 · security 7.5 / 10)
- **Content source:** fixture · **HTTP status:** 200

## Executive summary

Prilo is in a fairly risky spot right now because it collects data from students (via Google Analytics and Microsoft Clarity) without a privacy policy or cookie-consent notice in place — these are the two things to fix first, before anything else. The site also has several missing security headers that are straightforward to add through your hosting provider. Once the privacy and consent gaps are closed, the remaining items are quick technical tweaks that will meaningfully improve the site's security posture.

## Findings (10)

_Remediation advice below is tailored to this site._

| Severity | Finding | How to fix |
|---|---|---|
| HIGH | No privacy policy found anywhere | Write and publish a privacy policy page that explains what data Prilo collects (e.g. analytics, any account info), why, and how a user can ask for their data to be deleted. Add a link to it in the footer on every page — many free generators like Termly or Iubenda can produce a solid starting draft in minutes. |
| HIGH | Collects data from a student/youth audience with no privacy policy | Because Prilo is explicitly aimed at FBLA students (who may be under 18), having a privacy policy isn't just best practice — it's worth treating as urgent before you grow your user base further. Your policy should specifically address how you handle student data and, if any users could be under 13, what parental-consent steps you take. |
| MEDIUM | Analytics/tracking runs with no cookie-consent notice | Right now Google Analytics and Microsoft Clarity load for every visitor automatically — add a simple cookie-consent banner (Termly and Cookie Script both have free tiers and easy embed codes) so that tracking only activates after a student clicks 'Accept.' This is the standard expectation for sites with international or school-aged visitors. |
| MEDIUM | Email address exposed in raw HTML | Replace the hardcoded email address in your page HTML with a contact form (Bolt projects can embed a free Formspree or Tally form in a few clicks) — this stops spam bots from harvesting the address automatically. |
| MEDIUM | Missing Content-Security-Policy header | Add a Content-Security-Policy header through your hosting platform — if Prilo is deployed on Netlify, add it in a netlify.toml file under [[headers]]; on Vercel, use vercel.json under "headers". Start with a policy that only allows scripts and styles from your own domain and the specific third-party sources you use (Google, Microsoft Clarity). |
| MEDIUM | Missing Strict-Transport-Security (HSTS) header | Add the header 'Strict-Transport-Security: max-age=31536000; includeSubDomains' in the same headers config file you use for CSP (netlify.toml or vercel.json). This tells browsers to always use the secure HTTPS connection and never fall back to plain HTTP. |
| LOW | No terms of service linked | Add a short Terms of Service page covering what Prilo is for, what users may and may not do, and a basic liability disclaimer — Termly can generate one for free. Link it next to your privacy policy in the footer. |
| LOW | Missing Permissions-Policy header | In your hosting headers config, add 'Permissions-Policy: camera=(), microphone=(), geolocation=()' to explicitly turn off browser features Prilo doesn't need — this reduces risk if any third-party script ever misbehaves. |
| LOW | Missing Referrer-Policy header | Add 'Referrer-Policy: strict-origin-when-cross-origin' to your headers config alongside the other headers — this prevents the full page URL (which might contain student-related query parameters) from being sent to Google Analytics or Clarity when a user navigates away. |
| INFO | No /.well-known/security.txt published | Optional: publish a security.txt so researchers know how to report vulnerabilities responsibly. |

## Evidence

- **No privacy policy found anywhere** — `{"privacyHref":null}`
- **Collects data from a student/youth audience with no privacy policy** — _(no extra detail)_
- **Analytics/tracking runs with no cookie-consent notice** — `{"analytics":["GA","Clarity"]}`
- **Email address exposed in raw HTML** — `{"emails":["r***@prilo.ai","t***@prilo.ai","s***@prilo.ai"]}`
- **Missing Content-Security-Policy header** — `{"observed":"absent"}`
- **Missing Strict-Transport-Security (HSTS) header** — `{"observed":"absent"}`
- **No terms of service linked** — _(no extra detail)_
- **Missing Permissions-Policy header** — `{"observed":"absent"}`
- **Missing Referrer-Policy header** — `{"observed":"absent"}`
- **No /.well-known/security.txt published** — `{"checked":"/.well-known/security.txt"}`

---
_This assessment is based only on publicly available information returned by the site to a normal web request. No systems were accessed without authorization, and no active or intrusive testing was performed. Findings are provided in good faith to help improve the site's security and compliance posture._
