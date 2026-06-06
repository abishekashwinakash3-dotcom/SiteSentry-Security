# SiteSentry

An automated **passive** security & compliance scanner for websites. It reads only what a
site publicly serves to a normal browser, produces a professional per-site report, scores it
against a transparent rubric, and tracks findings over time. Built to power an
**audit-as-a-service** offering first, and a self-serve product later.

> **Legal boundary (important).** SiteSentry's default scanner is *passive* — it never
> exploits, fuzzes, injects, or brute-forces paths. It only reads public responses, the page's
> own linked resources, and a fixed allowlist of well-known files (`robots.txt`,
> `/.well-known/security.txt`, `sitemap.xml`). Active penetration testing is a **separate,
> authorization-gated** capability and must never be run against a site without the owner's
> written permission. Unsolicited active testing is illegal in most jurisdictions.

## Quick start

```bash
npm install                 # optional deps only; core uses Node built-ins
npm test                    # regression + unit tests
node run.js --scan          # scan every target in targets.json
node run.js --scan --site=prilo      # scan one target
node run.js --scan --no-firecrawl    # transport-only, $0 Firecrawl spend
```

Then open `dashboard/index.html` in a browser to browse results and the funnel.

## How it works

```
content (Firecrawl or cached fixture)  +  passive https fetch (headers/TLS/cookies)
        │
        ▼
  checks/  compliance · headers · cookies · tls · exposure
        │   (each emits findings defined in findings-catalog.js)
        ▼
  redact.js   (mask secret values + personal-email local parts)
        ▼
  score.js    (generic rubric, overrides.json for manual corrections)
        ▼
  report.js   →  reports/<id>.md + .html  +  dashboard/data.js
        ▼
  runs/<timestamp>.json   (raw snapshot — git-ignored)
```

## Layout

| Path | Purpose |
|------|---------|
| `config.json` | scan etiquette (UA + contact, delay, timeout, robots, Firecrawl toggle) |
| `targets.json` | sites to scan (id, url, platform, relationship, funnel status) |
| `overrides.json` | manual per-target corrections (privacy status, youth audience, add/remove findings) |
| `findings-catalog.js` | single source of truth: id → severity, deduction, fix text |
| `lib/fetch.js` | passive HTTP (status, headers, TLS, cookies, well-known) — honors robots.txt |
| `lib/firecrawl.js` | rendered content (cached fixture first, live Firecrawl when needed) |
| `lib/checks/` | compliance, headers, cookies, tls, exposure |
| `lib/redact.js` | strips raw secret values before anything is written/shared |
| `lib/score.js` | transparent scoring |
| `lib/report.js` | report + dashboard rendering |
| `fixtures/` | cached scrapes + frozen regression reference |
| `tests/` | regression (compliance parity) + unit tests |

## Safety guarantees enforced by tests

- The compliance checks reproduce the original hand-built analyzer **exactly** (regression test).
- Secret values are **masked** before reaching any shareable artifact; raw snapshots are git-ignored.
- Secret detection is **high-confidence only** — keys that are public by design (Firebase/Maps
  `AIza`, Stripe publishable `pk_`, anon JWTs) are deliberately **not** flagged.

## Roadmap

- [x] MVP: scan → checks → redact → score → report → dashboard
- [x] Split scoring (independent compliance + security 0-10 scores)
- [x] LLM remediation layer (the genuinely "AI-powered" differentiator)
- [x] Change-tracking digest + email to the operator (nodemailer / Gmail)
- [x] Scheduled automation (Windows Task Scheduler / Claude `/schedule`)
- [x] Funnel / sales-pipeline view
- [x] Authorization-gated active-testing module

## AI remediation layer

When an `ANTHROPIC_API_KEY` is present, each report's executive summary and per-finding
"how to fix" are rewritten by Claude into advice tailored to the specific site and builder
— the difference between "tool output" and a consultant deliverable.

```bash
cp .env.example .env          # then paste your key into ANTHROPIC_API_KEY
node run.js --scan --site=prilo
```

Notes:
- Runs on **redacted** findings only — raw secret values are never sent to the API.
- **Degrades gracefully**: with no key it falls back to the catalog fix text, so the tool
  always works.
- Results are cached in `runs/ai-cache.json`, so re-runs don't re-spend.
- Model/effort are configurable in `config.json` (`ai.model`, `ai.effort`). Default is
  `claude-opus-4-8`; switch to `claude-sonnet-4-6` or `claude-haiku-4-5` to cut cost.

## Digest + automation (hands-off + pings you)

Each run with `--notify` diffs against the previous run and produces a digest of **what's
new or worse** (score drops, new findings, new critical/private-disclosure items), plus a
priority list of the lowest-scoring sites. Output goes to `digest.md` / `digest.html`, and
is emailed when SMTP creds are set.

```bash
node run.js --scan --report --notify
```

**To auto-email it:** add a Gmail **App Password** to `.env` (`SMTP_USER` + `SMTP_PASS` —
see `.env.example`). Without them, the digest is still written for you (or for a scheduled
Claude run to send via the Gmail MCP).

**To run it on a schedule (Windows):**

```powershell
powershell -ExecutionPolicy Bypass -File scripts\register-task.ps1
```

That registers a daily 09:00 task running `--scan --report --notify`. Remove it with
`Unregister-ScheduledTask -TaskName SiteSentry -Confirm:$false`. (Alternatively, drive it
from a Claude Code `/schedule` routine, which can also send the digest via the Gmail MCP.)

## Sales funnel

Each target moves through a pipeline from lead to revenue:
`scanned → outreach_sent → replied → authorized → fixed → paid` (plus `lost`). The dashboard
shows a live pipeline strip and a coloured stage per site.

```bash
node funnel.js                              # show the pipeline summary
node funnel.js set tymora outreach_sent "sent value-first email"
node funnel.js set tymora paid "invoice #1 paid"
node funnel.js list authorized              # list targets in one stage
```

Stage changes are written to `targets.json` (with a date + optional note) and appear on the
dashboard after the next `--report`. This is the same target list the scanner uses, so your
leads, your scans, and your revenue all live in one place.

## Authorized active testing (gated)

Everything above is **passive**. Deeper / active testing lives behind a hard gate in
`pentest/` and **fails closed** — it refuses unless the site owner has signed off.

```bash
node pentest/active.js --site=acme.com --i-have-authorization
```

The gate (`pentest/gate.js`) allows the run only when ALL hold:

1. an authorization record for the domain exists in `pentest/authorizations.json`,
2. the referenced signed-scope file exists (created from `pentest/AUTHORIZATION.md`),
3. today is inside the authorization's start/end window,
4. you pass `--i-have-authorization`.

Every decision (allow **or** deny) is appended to `pentest/audit.log`. The shipped active
capability is **re-verification** — it re-runs the passive checks and reports which reported
findings are now fixed / still open / newly introduced (the core of a paid engagement). Heavier
tooling (OWASP ZAP, nuclei) is a documented integration point in `pentest/active.js`, to be
wired in only behind this same gate and only against authorized domains. Authorizations, scopes,
and the audit log are git-ignored (client records).

**Setup:** copy `pentest/AUTHORIZATION.md` → have the owner sign → save as
`pentest/scopes/<domain>.md` → add a record to `pentest/authorizations.json` (see
`authorizations.example.json`).
