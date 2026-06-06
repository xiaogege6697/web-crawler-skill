# 🕷️ web-crawler-skill — Smart Web Crawling with Anti-Detection Strategy Engine

> An [OpenClaw](https://github.com/nicepkg/openclaw) skill that turns any AI agent into an **intelligent web crawler** with a built-in anti-bot decision tree, open-source tool lookup tables, and production-ready templates.

---

## 🤔 Why This Skill?

Web scraping is getting harder. Sites deploy Cloudflare, DataDome, browser fingerprinting, behavioral analysis, and CAPTCHAs. Most developers (and AI agents) waste hours trial-and-erroring solutions.

This skill gives your AI agent a **systematic decision framework**:

1. **Assess** the anti-bot level in 30 seconds
2. **Select** the right tool from 6 categories of open-source solutions
3. **Execute** with proper delays, stealth configs, and error handling
4. **Stay safe** with built-in rate limits and ethical guardrails

---

## 🌳 Anti-Bot Decision Tree

```
Target Website
│
├─ 🟢 Light Protection (no WAF)
│   → Direct fetch / web_fetch
│   → Random User-Agent + basic delay
│
├─ 🟡 Medium Protection (basic WAF, rate limiting)
│   → Browser CDP fetch (stealth mode)
│   → Hide navigator.webdriver etc.
│   → Random delay 2.5–5s per page
│   → Spoofed Referer / Accept-Language
│
├─ 🔴 Heavy Protection (Cloudflare/DataDome/fingerprinting/CAPTCHA)
│   → Camoufox / undetected-chromedriver
│   → Residential proxy rotation
│   → CAPTCHA solving (CapSolver/2Captcha)
│   → Full fingerprint spoofing (Canvas/WebGL/Navigator)
│
└─ ⛔ Extreme Protection (login-gated paid content)
    → STOP. Do not crawl. (Hard rule.)
```

---

## 📊 Open-Source Solutions Lookup Table

### Stealth / Anti-Detection

| Tool | Language | Stars | Best For |
|------|----------|-------|----------|
| **[Camoufox](https://github.com/daijro/camoufox)** | Python | ~5k+ | 🔥 2025–2026 hottest tool. Firefox-based anti-detect browser designed for AI agents |
| **[undetected-chromedriver](https://github.com/ultrafunkamsterdam/undetected-chromedriver)** | Python | ~9k+ | Auto-patches chromedriver, bypasses Distil/Imperva/DataDome |
| **[puppeteer-extra-plugin-stealth](https://github.com/berstend/puppeteer-extra)** | Node.js | ~8k+ | 10+ evasion modules for Puppeteer |
| **[playwright-stealth](https://github.com/Mattwmaster58/playwright_stealth)** | Python | ~1k+ | Python Playwright stealth, actively maintained in 2026 |
| **[DrissionPage](https://github.com/g1879/DrissionPage)** | Python | ~25k+ | No webdriver dependency, great for Chinese sites |

### Anti-WAF

| Tool | Best For |
|------|----------|
| **[FlareSolverr](https://github.com/FlareSolverr/FlareSolverr)** | Cloudflare JS Challenge solving via proxy server |
| **[Camoufox](https://github.com/daijro/camoufox)** | Cloudflare + fingerprint evasion |
| **[DrissionPage](https://github.com/g1879/DrissionPage)** | Chinese WAF (瑞数/加速乐) bypass |

### CAPTCHA Solving

| Service | Supports | Pricing |
|---------|----------|---------|
| **[2Captcha](https://2captcha.com)** | reCAPTCHA/hCaptcha/slider/image | $2.99/1000 |
| **[CapSolver](https://www.capsolver.com)** | reCAPTCHA/hCaptcha/FunCaptcha/Turnstile | Pay-per-use |

---

## 🔮 2025–2026 Trends

1. **Camoufox is the current hottest solution** — Firefox-based anti-detect browser built specifically for AI agents
2. **Python ecosystem dominates** — playwright-stealth Python version is more actively maintained than Node.js
3. **Stealth alone isn't enough** — IP reputation, TLS fingerprinting, and behavioral analysis require deeper tooling
4. **Cloudflare Turnstile replacing reCAPTCHA** — more sites adopting the newer challenge
5. **In-browser fetch is best practice** — leverage existing sessions instead of creating new connections

---

## 🚀 Installation

### Prerequisites
- [OpenClaw](https://github.com/nicepkg/openclaw) installed and running

### Install

```bash
# Clone this skill into your shared-skills directory
cd ~/shared-skills
git clone https://github.com/xiaogege6697/web-crawler-skill.git

# Restart OpenClaw gateway to pick up the new skill
openclaw gateway restart
```

### Usage

Once installed, your AI agent will automatically use this skill when you ask it to crawl or scrape websites. The skill provides:

- **Anti-bot assessment** — automatically evaluates target site protection level
- **Tool selection** — picks the right approach from simple fetch to Camoufox
- **Rate limiting** — enforces 2.5–5s random delays per page
- **Error recovery** — handles 403/429/CAPTCHA with proper backoff

---

## 📋 Quick Reference

### Minimal Viable Crawl (no protection)
```javascript
// Just use web_fetch
web_fetch(url) → extract markdown content
```

### Standard Browser Crawl
```javascript
1. browser(action="navigate", url=target)
2. browser(action="snapshot") → inspect page
3. browser(action="act", kind="evaluate", fn=extractFn) → get data
4. Repeat with random delays
```

### Heavy Protection Crawl
```bash
# Install Camoufox
pip install camoufox && python -m camoufox fetch

# Or use FlareSolverr via Docker
docker run -d -p 8191:8191 flaresolverr/flaresolverr
# POST http://localhost:8191/v1 {"cmd":"request.get","url":"..."}
```

---

## 🛡️ Safety Rules (Built-In)

| Rule | Details |
|------|---------|
| ⏱️ **Random delay** | 2.5–5 seconds per page, always |
| 🛑 **Stop on 403/429** | Never brute-force through blocks |
| 🚫 **No paid content** | Login-gated paid content is off-limits |
| 🤖 **No HeadlessChrome UA** | Use real Chrome UA strings |
| 📊 **Rate cap** | Max 1000 requests/day per domain |
| 🍪 **Cookie expiry alerts** | Plan ahead, don't lose sessions mid-crawl |
| 📜 **Respect robots.txt** | Unless user explicitly overrides |

---

## 📁 Repository Structure

```
web-crawler-skill/
├── README.md        ← You are here
├── SKILL.md         ← The complete skill definition
└── LICENSE          ← MIT
```

---

## 📄 License

MIT — see [LICENSE](LICENSE).

---

### ⭐ If this saved you hours of trial-and-error, please give it a star!

Stars help other developers discover this systematic approach to web crawling. Thank you! 🙏
