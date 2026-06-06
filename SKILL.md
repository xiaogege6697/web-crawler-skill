# Web Crawler Skill — 智能爬虫技能（反爬策略库）

> 适用场景：用户需要爬取网页数据、批量采集信息、抓取API数据时使用。
> 核心原则：**从轻到重调度，够用就停。**

---

## 1. 反爬策略决策树

接到爬取任务后，按以下流程评估并选择策略：

```
目标网站
├─ 评估反爬等级（访问首页观察响应）
│
├─ 🟢 轻度防护（普通网站、无WAF）
│   → 直接 fetch / web_fetch
│   → 随机 User-Agent + 基础延迟
│
├─ 🟡 中度防护（WAF 基础检测、频率限制）
│   → 浏览器内 CDP fetch（OpenClaw browser 工具）
│   → stealth 配置：隐藏 navigator.webdriver 等
│   → 随机延迟 2.5-5 秒/页
│   → 伪造 Referer / Accept-Language
│
├─ 🔴 重度防护（Cloudflare/DataDome/滑块验证码/指纹检测）
│   → Camoufox / undetected-chromedriver
│   → 代理轮换（住宅IP优先）
│   → 验证码处理（CapSolver/2Captcha API）
│   → 指纹伪装（Canvas/WebGL/Navigator 全套）
│
└─ ⛔ 极端防护（需要登录的付费内容）
    → 停止，不爬取（红线）
```

### 反爬等级评估方法

1. **直接 fetch 首页**，观察：
   - HTTP 状态码：200 ✅ / 403 🔴 / 503 + challenge 🔴
   - 响应头：有无 `cf-ray`（Cloudflare）、`x-datadome`（DataDome）
   - HTML 内容：有无 challenge 页面、JS 重定向
2. **检查 robots.txt**：`<target_url>/robots.txt`
3. **浏览器打开观察**：有无弹窗验证码、JS Challenge

---

## 2. 开源方案速查表

### 2.1 Stealth / 反检测

| 方案 | 语言 | GitHub | Stars | 核心能力 | 适用场景 |
|------|------|--------|-------|----------|----------|
| **puppeteer-extra-plugin-stealth** | Node.js | [berstend/puppeteer-extra](https://github.com/berstend/puppeteer-extra/tree/master/packages/puppeteer-extra-plugin-stealth) | ~8k+ | 隐藏 webdriver、模拟 plugins、伪造 chrome runtime 等 10+ evasions | Puppeteer/Playwright Node.js 爬虫 |
| **playwright-stealth** | Python | [Mattwmaster58/playwright_stealth](https://github.com/Mattwmaster58/playwright_stealth) | ~1k+ | Playwright Python stealth，2026 活跃维护，context-manager API | Python Playwright 爬虫（推荐） |
| **playwright-extra + stealth** | Node.js | [berstend/puppeteer-extra](https://github.com/berstend/puppeteer-extra/tree/master/packages/playwright-extra) | ~8k+ | playwright-extra 封装 + stealth 插件 | Node.js Playwright（维护较少） |
| **undetected-chromedriver** | Python | [ultrafunkamsterdam/undetected-chromedriver](https://github.com/ultrafunkamsterdam/undetected-chromedriver) | ~9k+ | 自动 patch chromedriver，绕过 Distil/Imperva/DataDome | Selenium Python 爬虫 |
| **Camoufox** | Python | [daijro/camoufox](https://github.com/daijro/camoufox) | ~5k+ | 基于 Firefox 反检测浏览器，专为 AI Agent 设计，完整指纹伪装 | 2025-2026 新兴方案，重度防护首选 |
| **DrissionPage** | Python | [g1879/DrissionPage](https://github.com/g1879/DrissionPage) | ~25k+ | 不基于 webdriver，浏览器控制+数据包收发合一，中文文档 | 中文网站爬虫、国产WAF绕过 |

### 2.2 反 WAF / Cloudflare

| 方案 | 说明 | GitHub |
|------|------|--------|
| **FlareSolverr** | 代理服务器，用 undetected-chromedriver 解决 Cloudflare challenge，返回 HTML+cookies | [FlareSolverr/FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) |
| **Camoufox** | 基于 Firefox 的反检测浏览器，也能处理 Cloudflare | [daijro/camoufox](https://github.com/daijro/camoufox) |
| **DrissionPage** | 不触发 webdriver 检测，天然绕过部分 WAF | [g1879/DrissionPage](https://github.com/g1879/DrissionPage) |

### 2.3 验证码处理

| 方案 | 类型 | 说明 |
|------|------|------|
| **2Captcha** | API 服务 | 支持reCAPTCHA/hCaptcha/滑块/图片，$2.99/1000次，[2captcha.com](https://2captcha.com) |
| **CapSolver** | API 服务 | 支持reCAPTCHA/hCaptcha/FunCaptcha/Turnstile，按量计费，[capsolver.com](https://www.capsolver.com) |
| **CapMonster Cloud** | API 服务 | ZenRows 团队维护，支持主流验证码类型 |

> 注：验证码解决方案均为商业 API 服务，无成熟全开源免费方案。

### 2.4 代理轮换

| 方案 | 类型 | 说明 |
|------|------|------|
| **住宅代理（商业）** | 商业服务 | Thordata、ProxyEmpire、Bright Data 等，轮换住宅IP |
| **proxy-pool** | 开源 | [jhao104/proxy_pool](https://github.com/jhao104/proxy_pool) — 免费代理池，质量不稳定 |
| **scylla** | 开源 | [scylla-proxy/scylla](https://github.com/scylla-proxy/scylla) — 智能代理池 |

### 2.5 浏览器指纹伪装

| 方案 | 说明 |
|------|------|
| **Camoufox** | 内置完整指纹伪装（Canvas/WebGL/Audio/Fonts/ClientRect/WebRTC等） |
| **undetectable-fingerprint-browser** | [itbrowser-net](https://github.com/itbrowser-net/undetectable-fingerprint-browser) — 开源反检测浏览器 |
| **CloakBrowser** | [CloakHQ](https://gitlab.com/CloakHQ/cloakbrowser) — 模块化指纹种子，每实例独立指纹 |
| **playwright-stealth** | 通过 init_scripts 修补 navigator/plugins/webGL 等 |

### 2.6 频率控制策略

- **随机延迟**：每页 2.5-5 秒随机间隔（红线，必须遵守）
- **指数退避**：遇到 429 时，等待 2^n 秒后重试（最多 3 次）
- **令牌桶**：限制每分钟请求数（如 10 req/min）
- **时段分散**：大量任务分散到非高峰时段

---

## 3. 标准工作流模板

```
接到爬取任务
  │
  ├─ Step 1: 信息收集
  │   • 确认目标 URL、数据范围、采集量
  │   • 评估反爬等级（决策树 §1）
  │   • 检查 robots.txt
  │
  ├─ Step 2: 选择策略
  │   • 根据反爬等级选择方案（§1 决策树）
  │   • 确定是否需要代理、验证码处理
  │
  ├─ Step 3: 配置参数
  │   • 延迟范围、User-Agent、并发数
  │   • 代理地址（如需）
  │   • 验证码 API key（如需）
  │
  ├─ Step 4: 执行爬取
  │   • 从第一页开始，逐页爬取
  │   • 每页随机延迟 2.5-5s
  │   • 实时监控状态码
  │
  ├─ Step 5: 异常处理
  │   • 405/429 → 立即停止，等待后降速重试
  │   • 验证码弹出 → 调用验证码处理方案
  │   • Cookie 过期 → 重新获取
  │   • 页面结构变化 → 记录并调整选择器
  │
  ├─ Step 6: 数据验证
  │   • 校验数据完整性（字段非空、数量匹配）
  │   • 去重
  │   • 保存到指定格式
  │
  └─ Step 7: 收尾
      • 清理 cookies/session
      • 记录爬取日志（URL、时间、状态、数据量）
```

---

## 4. OpenClaw 环境适配

### 4.1 优先使用 OpenClaw browser 工具

OpenClaw 已内置 Chrome CDP 浏览器，**优先使用 `browser` 工具**而非启动外部浏览器。

**典型工作流：**

```
1. browser(action="snapshot") — 获取页面快照，检查页面结构
2. browser(action="act", kind="evaluate", fn="...") — 在页面内执行 JS/fetch
3. browser(action="navigate", url="...") — 导航到目标页
4. browser(action="snapshot") — 获取新页面内容
```

### 4.2 CDP 直接连接（高级场景）

当 browser 工具不够用时，可通过 CDP WebSocket 直接操作：

- **CDP 地址**: `ws://127.0.0.1:18800`
- **获取 tab 列表**: `fetch('http://127.0.0.1:18800/json')`
- **Runtime.evaluate**: 执行页面内 fetch 请求

### 4.3 浏览器内 fetch 模板（推荐）

在 OpenClaw browser 工具中使用 `act: evaluate` 执行页面内 fetch：

```javascript
// 在浏览器上下文中执行 fetch（自动携带目标站 cookies）
async () => {
  const resp = await fetch('https://target.com/api/data', {
    headers: {
      'Accept': 'application/json',
      'Referer': 'https://target.com/',
    }
  });
  return await resp.text();
}
```

---

## 5. 安全红线

> ⚠️ **必须遵守，不可绕过**

1. **随机延迟 2.5-5 秒/页** — 不贪快，不做并发轰炸
2. **遇到 405/429 立即停止** — 不硬冲，等待后降速重试
3. **不爬取需要登录的付费内容** — 不碰账号体系
4. **Cookie 过期提前预警** — 提前规划好，不要爬到一半失效
5. **遵守 robots.txt** — 除非用户明确要求且了解风险
6. **开工前先调研反爬方案** — 不盲目开干
7. **数据脱敏** — 敏感信息不存储明文
8. **单次任务量控制** — 单域名日请求不超过 1000 次
9. **不使用 HeadlessChrome UA** — 用真实的 Chrome UA 字符串

---

## 6. 常用脚本模板

### 6.1 OpenClaw browser 工具 — 页面数据采集

```javascript
// 通过 browser(action="act", kind="evaluate", fn="...") 执行
async () => {
  const items = document.querySelectorAll('.item-selector');
  return Array.from(items).map(item => ({
    title: item.querySelector('.title')?.textContent?.trim(),
    price: item.querySelector('.price')?.textContent?.trim(),
    link: item.querySelector('a')?.href,
  }));
}
```

### 6.2 CDP fetch 模板（Node.js）

当需要独立脚本时：

```javascript
const http = require('http');

async function cdpFetch(targetDomain) {
  // 获取 tab 列表
  const tabs = await new Promise((resolve, reject) => {
    http.get('http://127.0.0.1:18800/json', (res) => {
      let data = '';
      res.on('data', (chunk) => { data += chunk; });
      res.on('end', () => { resolve(JSON.parse(data)); });
    }).on('error', reject);
  });
  const tab = tabs.find(t => t.url.includes(targetDomain)) || tabs[0];
  return tab;
}
module.exports = { cdpFetch };
```

### 6.3 分页爬取模板

```
工作流：
1. browser(action="navigate", url="第1页URL")
2. browser(action="act", kind="evaluate", fn="提取数据函数")
3. browser(action="act", kind="evaluate", fn="点击下一页")
4. 等待 2.5-5 秒随机延迟
5. 重复步骤 2-4 直到没有下一页
6. 汇总所有数据，去重验证
```

### 6.4 数据验证模板

```javascript
async () => {
  const data = []; // 已采集数据
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

### 6.5 随机延迟函数

```javascript
async function randomDelay(minMs = 2500, maxMs = 5000) {
  const delay = minMs + Math.random() * (maxMs - minMs);
  return new Promise(resolve => setTimeout(resolve, delay));
}
```

---

## 7. 方案速查表（按反爬类型）

| 反爬类型 | 推荐工具 | 优先级 |
|----------|----------|--------|
| **无防护/轻量** | web_fetch / browser evaluate | 首选 |
| **UA 检测** | 伪造真实 Chrome UA | 内置 |
| **Referer 检测** | fetch 时携带 Referer 头 | 内置 |
| **频率限制** | 随机延迟 2.5-5s + 指数退避 | 内置 |
| **Cookie 验证** | browser 工具自动管理 | 内置 |
| **Cloudflare JS Challenge** | FlareSolverr / Camoufox | 🟡-🔴 |
| **Cloudflare Turnstile** | CapSolver / 2Captcha API | 🔴 |
| **DataDome** | Camoufox + 住宅代理 | 🔴 |
| **PerimeterX/HUMAN** | Camoufox + 住宅代理 + 指纹伪装 | 🔴 |
| **reCAPTCHA v2/v3** | 2Captcha / CapSolver API | 🔴 |
| **hCaptcha** | CapSolver / 2Captcha API | 🔴 |
| **滑块验证码** | CapSolver / 手动处理 | 🔴 |
| **浏览器指纹检测** | Camoufox / playwright-stealth | 🟡-🔴 |
| **TLS 指纹** | curl-impersonate / Camoufox | 🔴 |
| **IP 封锁** | 住宅代理轮换 | 🔴 |
| **行为分析（鼠标/滚动）** | Camoufox 内置行为模拟 | 🔴 |
| **国产 WAF（瑞数/加速乐）** | DrissionPage | 🟡-🔴 |

---

## 8. 关键注意事项

### 2025-2026 趋势

1. **Camoufox 是当前最热门方案** — 基于 Firefox 反检测浏览器，专为 AI Agent 设计
2. **Python 生态强于 Node.js** — playwright-stealth Python 版维护更活跃
3. **Stealth 不够用** — 仅解决指纹级检测，无法解决 IP 声誉、TLS 指纹、行为分析
4. **Cloudflare Turnstile 取代 reCAPTCHA** — 越来越多站点使用
5. **浏览器内 fetch 是最佳实践** — 利用已有 session，不触发额外检测

### OpenClaw 内置能力（优先使用）

- `browser` 工具 — 已启动的 Chrome CDP，直接使用
- `web_fetch` — 轻量 HTTP 请求，适合无防护站点
- `web_search` — 搜索引擎查询
- `exec` — 运行外部脚本（Python/Node.js）

### 外部工具引入条件

**只在以下情况引入外部工具：**
1. browser 工具被目标站检测（403/challenge）
2. 需要代理轮换
3. 需要验证码处理
4. 用户明确要求使用特定工具

**引入外部工具前必须先安装依赖，不可假设已安装。**

---

## 9. 快速参考

### 最小可行爬取（OpenClaw 内置）
```
1. web_fetch(url) → 提取 markdown 内容
```

### 标准浏览器爬取
```
1. browser(action="navigate", url=target)
2. browser(action="snapshot") → 检查页面
3. browser(action="act", kind="evaluate", fn=提取函数) → 获取数据
4. 重复 + 随机延迟
```

### 重度防护爬取
```
1. 安装 camoufox: pip install camoufox && python -m camoufox fetch
2. 或启动 FlareSolverr: docker run -d -p 8191:8191 flaresolverr/flaresolverr
3. POST http://localhost:8191/v1 {"cmd":"request.get","url":"..."}
4. 获取 cookies 后用 web_fetch 继续爬取
```
