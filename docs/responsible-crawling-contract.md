# Responsible Crawling Contract

## Project Identity

`web-crawler-skill` helps an agent collect public web information in a controlled, observable, and respectful way. Its core is choosing the lightest lawful access path that can produce reliable data.

## Narrow Waist

Every crawl should pass through this small contract before implementation details are chosen:

1. **Scope**: target URLs, fields, expected page count, and output format are explicit.
2. **Permission posture**: public page, user-owned page, official API, or separately authorized access.
3. **Access path**: prefer official API, RSS/sitemap/export, simple fetch, then browser-rendered reading.
4. **Limits**: low concurrency, delay, max pages, retry cap, and stop conditions are set before running.
5. **Evidence**: save source URLs, timestamps, status codes, item counts, and validation summary.

Tools, browsers, parsers, and storage formats are replaceable. The contract above is the stable part.

## Stop Conditions

Stop and report instead of escalating when any of these appear:

- Login wall, paywall, account-only content, or session that the user is not clearly authorized to use.
- CAPTCHA, bot challenge, WAF block, 403/429 patterns, or repeated redirects to challenge pages.
- `robots.txt`, terms, or page copy clearly disallow the requested collection.
- The target includes personal data, secrets, tokens, private messages, or non-public files.
- The task requires high volume, evading detection, rotating identities, or continuing after blocks.

For site-owner, internal QA, archival, or explicitly authorized research, document the authorization and keep the same rate limits and evidence requirements.

## Default Strategy Ladder

1. Use an official API, export, RSS feed, sitemap, or static file when available.
2. Use a single request or reader mode for simple public pages.
3. Use a browser snapshot only when JavaScript rendering is required.
4. Use pagination with fixed caps and delays.
5. If access is blocked, stop and produce a diagnostic summary rather than trying to defeat the block.

## Minimum Output

A successful crawl should include:

- collected data in the requested format,
- source URL list or provenance field,
- count of attempted, fetched, skipped, and failed pages,
- validation summary,
- any stop reason or incomplete area.
