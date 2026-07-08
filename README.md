# web-crawler-skill — Responsible Web Crawling Strategy

> An OpenClaw skill that helps AI agents collect public web information with clear scope, low impact, provenance, validation, and stop conditions.

## Why This Skill Exists

Web collection is useful, but it can easily drift into fragile, noisy, or unauthorized behavior. The core of this project is not "bypass everything"; it is helping an agent choose the lightest lawful access path that can produce reliable data.

The stable contract is:

1. Define target URLs, fields, volume, and output format.
2. Prefer official APIs, exports, RSS, sitemaps, static files, and simple public fetches.
3. Use browser rendering only when public pages require JavaScript.
4. Apply low concurrency, delays, retry caps, and source provenance.
5. Stop on login walls, paywalls, CAPTCHA, WAF blocks, personal data risk, or disallowed paths.

See [`docs/responsible-crawling-contract.md`](docs/responsible-crawling-contract.md) for the project core.

## Access Strategy Ladder

```text
Target website
|
+-- Official or static source
|   -> API / RSS / sitemap / export / simple fetch
|   -> Low rate, provenance, validation
|
+-- JavaScript-rendered public pages
|   -> Browser snapshot / browser evaluate
|   -> Delay 2.5-5s per page, no parallel burst
|   -> Stop on repeated errors
|
+-- Blocked or challenged access
|   -> Stop and produce diagnostics
|   -> Prefer official API/export or documented authorization
|
+-- Login-gated paid/private content
    -> Stop. Do not crawl without clear authorization.
```

## Safety Rules

| Rule | Details |
|------|---------|
| Scope first | Confirm URLs, fields, expected count, output format, and allowed access path |
| API/export first | Prefer official or static sources before browser automation |
| Low impact | Use delays, page caps, retry caps, and no burst concurrency |
| Stop on blocks | Stop on 403/429, CAPTCHA, bot challenges, paywalls, or login walls |
| Respect site signals | Check robots.txt and obvious terms or page restrictions |
| Minimize sensitive data | Do not collect personal data at scale without explicit authorization |
| Keep evidence | Save URLs, timestamps, status counts, skipped reasons, and validation summary |

## Quick Reference

### Minimal Public Page Read

```text
1. Fetch or read the public URL.
2. Extract only requested fields.
3. Save source URL and timestamp.
4. Validate required fields and dedupe.
```

### Browser-Rendered Public Page

```text
1. Navigate with browser tooling.
2. Snapshot and inspect page structure.
3. Extract requested fields with evaluate.
4. Wait 2.5-5 seconds before next page.
5. Stop on repeated failures or challenge pages.
```

### Blocked Access Diagnostic

```text
Report:
- target URL
- status code or challenge indicator
- robots/terms/login/paywall signal
- low-impact strategy attempted
- recommended authorized alternative
```

## Repository Structure

```text
web-crawler-skill/
├── README.md
├── SKILL.md
├── docs/
│   └── responsible-crawling-contract.md
├── evals/
│   └── evals.json
├── VERSION
└── LICENSE
```

## License

MIT — see [LICENSE](LICENSE).
