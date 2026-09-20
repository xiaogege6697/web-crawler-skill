# web-crawler-skill — Responsible Web Crawling for AI Agents

> An agent skill that picks the lightest lawful path to collect public web data — with clear scope, low impact, provenance, validation, and hard stop conditions. Not an anti-detection escalation toolkit.
>
> **中文**：面向 AI Agent 的负责任网页采集技能——**公开或授权、从轻到重、够用就停、被挡即停。** 帮 Agent 选一条最轻的合法访问路径拿到可靠数据，而不是教它绕过一切防护。

## Why This Skill Exists

Web collection is genuinely useful to agents, but it drifts easily into fragile, noisy, or unauthorized behavior:

- An agent crawls 200 pages, gets blocked at page 12, and silently retries into a wall.
- A task that an official API or RSS feed would solve in 10 requests becomes 500 browser renders.
- Login walls, paywalls, CAPTCHAs, and personal-data boundaries get treated as obstacles to defeat instead of signals to respect.

The core of this skill is **not** "bypass everything." It is helping an agent choose the lightest lawful access path that produces reliable data — and stop with a useful diagnostic when it can't.

**中文摘要**：Agent 采集网页时容易漂移成脆弱、脏乱或越权的行为。本技能的核心不是"绕过一切"，而是选最轻的合法路径拿到可靠数据；拿不到就带着诊断信息停下来。

## The Stable Contract

Every crawl passes through this narrow waist before any tool is chosen:

| # | Clause | What it means |
|---|--------|---------------|
| 1 | **Scope** | Target URLs, fields, expected page count, and output format are explicit |
| 2 | **Permission posture** | Public page, user-owned page, official API, or separately authorized access |
| 3 | **Access path** | Prefer official API, RSS/sitemap/export, simple fetch — browser rendering last |
| 4 | **Limits** | Low concurrency, 2.5–5 s delays, page caps, retry caps, stop conditions set before running |
| 5 | **Evidence** | Source URLs, timestamps, status codes, item counts, validation summary |

Tools, browsers, parsers, and storage formats are replaceable. The contract is the stable part. See [`docs/responsible-crawling-contract.md`](docs/responsible-crawling-contract.md).

**中文**：五条采集契约——范围、权限姿态、访问路径、限制、证据。工具可换，契约不变。

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
+-- Blocked or challenged access (CAPTCHA / WAF / 403 / 429)
|   -> Stop and produce diagnostics
|   -> Prefer official API/export or documented authorization
|
+-- Login-gated, paid, or private content
    -> Stop. Do not crawl without clear authorization.
```

Before choosing a rung, assess the target: fetch the page and observe status codes and redirects, check `robots.txt` and visible terms, open it in a browser to see whether it is truly public, and search for an official API, RSS feed, sitemap, or export.

**中文**：访问策略阶梯——官方/静态来源优先，其次浏览器渲染公开页，被阻断就停下出诊断，登录/付费/私有内容一律不爬。

## Features

- **Decision-tree access strategy** — API → export → static fetch → browser rendering, always starting from the lightest rung.
- **Hard stop conditions** — login walls, paywalls, CAPTCHA/bot challenges, WAF blocks, repeated 403/429, `robots.txt` or ToS prohibitions, and personal-data boundaries stop the crawl instead of escalating.
- **Low-impact defaults** — random 2.5–5 s delays per page, page and retry caps, no burst concurrency.
- **Provenance-first output** — every delivery carries source URLs, timestamps, attempted/fetched/skipped/failed counts, a validation summary, and stop reasons.
- **Ready-made templates** — browser crawl loop, field-validation and dedupe script, random-delay function, and a blocked-access diagnostic format.
- **Blocked-access diagnostics** — when stopped, reports the target, status, stop reason, strategies attempted, and recommended authorized alternatives (official API, export, narrowed scope, manual sampling).
- **Eval suite** — [`evals/evals.json`](evals/evals.json) locks in the behavior: prefer official APIs, stop on paywalls and CAPTCHAs, respect `robots.txt`, protect personal data.

## Quick Start

```bash
# Install into any Agent Skills-compatible client (OpenClaw / Claude Code / Codex etc.)
git clone https://github.com/xiaogege6697/web-crawler-skill.git \
  ~/.agents/skills/web-crawler-skill
```

No extra dependencies. The skill drives whatever fetch, browser, and search tools your agent runtime already provides — it adds the contract, the ladder, the limits, and the stop lines on top of them.

## Usage Examples

```text
Collect titles and links from the 20 public pages under https://example.com/docs.

Crawl this site's blog index, but check first whether they offer an RSS or API.

The site started showing a CAPTCHA after two pages — continue with a solver.   # -> the skill refuses and reports instead

Scrape all articles behind our competitor's paid login wall.                  # -> stopped: needs clear authorization
```

Representative behavior for each: public docs → simple fetch with delays and validation; blog index → RSS/sitemap preferred over crawling; CAPTCHA → stop and produce a diagnostic, never automate challenge solving; paywalled content → stop unless the user is clearly authorized.

## Ready-Made Templates

**Browser crawl loop** (JavaScript-rendered public pages):

```text
1. browser(action="navigate", url=target)
2. browser(action="snapshot")  -> inspect structure
3. browser(action="act", kind="evaluate", fn=<extractor>)  -> get data
4. wait 2.5-5 s (randomized)
5. next page or stop
```

**Validation and dedupe**:

```javascript
async () => {
  const data = []; // collected items
  const valid = data.filter(item =>
    item.title && item.title.length > 0 &&
    item.link && item.link.startsWith('http')
  );
  const deduped = [];
  const seen = new Set();
  for (const item of valid) {
    if (!seen.has(item.link)) { seen.add(item.link); deduped.push(item); }
  }
  return { total: data.length, valid: valid.length, deduped: deduped.length };
}
```

**Random delay**:

```javascript
async function randomDelay(minMs = 2500, maxMs = 5000) {
  const delay = minMs + Math.random() * (maxMs - minMs);
  return new Promise(resolve => setTimeout(resolve, delay));
}
```

## Responsible Use — Read This First

This skill is designed for **authorized, low-impact collection of public information** only:

- Check `robots.txt` and the site's terms of service before crawling; respect what they disallow.
- Do not bypass CAPTCHAs, WAF blocks, login walls, or paywalls. Blocked access produces diagnostics, not workarounds.
- Do not collect personal data at scale without explicit authorization; treat emails, phone numbers, private messages, and tokens as out of scope by default.
- Site-owner QA, internal testing, archival, and authorized research are fine — document the authorization and keep the same rate limits and evidence requirements.

**中文（合规边界）**：本技能仅用于**授权场景下的公开信息低影响采集**——遵守 `robots.txt` 与网站条款；不绕过验证码、WAF、登录墙与付费墙；默认不批量采集个人数据。被挡即停，输出诊断与合规替代方案。

## Repository Structure

```text
web-crawler-skill/
├── README.md
├── SKILL.md                                 # 主工作流与安全红线（中文）
├── docs/
│   └── responsible-crawling-contract.md     # 采集契约：范围/权限/路径/限制/证据
├── evals/
│   └── evals.json                           # 行为回归：API 优先、停止条件、个人数据边界
├── CHANGELOG.md
├── VERSION
└── LICENSE
```

## License

MIT — see [LICENSE](LICENSE).

## Related

- [xiaogege6697](https://github.com/xiaogege6697) — more AI agent skills (topic research, comic workflow, persona skills, and more)

<!-- AI/Friendly Search Metadata -->
**keywords: web crawling, web scraping, responsible crawling, provenance, validation, anti-bot handling, data collection, Claude Code, OpenClaw, Codex, skill, 网页采集, 爬虫, 负责任采集, 溯源, 校验**

