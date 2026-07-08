# Web Crawler Skill — 负责任网页采集技能

> 适用场景：用户需要采集公开网页数据、批量整理资料、读取开放 API/RSS/sitemap 时使用。  
> 核心原则：**公开或授权、从轻到重、够用就停、被挡即停。**

---

## 1. 先建立采集契约

开始前确认五件事：

1. **范围**：目标 URL、字段、页数上限、输出格式。
2. **权限姿态**：公开页面、用户自有页面、官方 API、或明确授权访问。
3. **访问路径**：优先 API、RSS、sitemap、export、静态页面，再考虑浏览器渲染。
4. **限制**：延迟、并发、重试上限、停止条件。
5. **证据**：来源 URL、时间、状态码、采集数量、跳过原因、校验结果。

---

## 2. 访问策略决策树

```text
目标网站
|
+-- 官方或静态来源
|   -> API / RSS / sitemap / export / web_fetch
|   -> 低频、记录来源、验证数量
|
+-- JavaScript 渲染公开页面
|   -> OpenClaw browser snapshot / evaluate
|   -> 随机延迟 2.5-5 秒/页，限制页数
|   -> 记录失败、跳过和校验结果
|
+-- 被阻断或出现 challenge
|   -> 停止采集，输出诊断
|   -> 建议官方 API、数据导出、授权窗口或缩小范围
|
+-- 登录、付费、私有或账号授权不明
    -> 停止，不爬取
```

### 访问状态评估

1. 直接读取首页或目标页，观察状态码、重定向、提示文案。
2. 检查 `<target_url>/robots.txt` 和页面上的明显限制信号。
3. 浏览器打开观察是否为公开页面，是否需要登录、付费、验证码或账号状态。
4. 查找官方 API、RSS、sitemap、导出文件或公开数据集。

---

## 3. 标准工作流

```text
接到采集任务
  |
  +-- Step 1: 明确范围
  |   - URL、字段、页数、输出格式
  |   - 公开/授权状态
  |
  +-- Step 2: 选择最轻路径
  |   - API/RSS/sitemap/export
  |   - web_fetch / reader
  |   - browser snapshot / evaluate
  |
  +-- Step 3: 设置限制
  |   - 延迟 2.5-5 秒/页
  |   - 页数上限
  |   - 重试上限
  |   - 停止条件
  |
  +-- Step 4: 小样本执行
  |   - 先采少量页面
  |   - 校验字段和去重
  |
  +-- Step 5: 扩大或停止
  |   - 通过验证后扩大
  |   - 403/429/CAPTCHA/登录/付费/robots 禁止时停止
  |
  +-- Step 6: 交付
      - 数据
      - 来源与统计
      - 跳过/失败原因
      - 校验摘要
```

---

## 4. OpenClaw 环境适配

优先使用 OpenClaw 内置能力：

- `web_fetch`：读取无防护公开页面。
- `browser`：处理需要 JavaScript 渲染的公开页面。
- `web_search`：发现官方 API、RSS、sitemap 或资料入口。
- `exec`：仅在需要运行小型解析脚本时使用。

### Browser 模板

```text
1. browser(action="navigate", url=target)
2. browser(action="snapshot") -> 检查页面
3. browser(action="act", kind="evaluate", fn=提取函数) -> 获取数据
4. 等待 2.5-5 秒
5. 继续下一页或停止
```

### 数据验证模板

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
    if (!seen.has(item.link)) {
      seen.add(item.link);
      deduped.push(item);
    }
  }
  return { total: data.length, valid: valid.length, deduped: deduped.length };
}
```

### 随机延迟函数

```javascript
async function randomDelay(minMs = 2500, maxMs = 5000) {
  const delay = minMs + Math.random() * (maxMs - minMs);
  return new Promise(resolve => setTimeout(resolve, delay));
}
```

---

## 5. 安全红线

必须停止并报告，不自动升级绕过：

1. 需要登录、付费、账号授权不明或私有内容。
2. CAPTCHA、bot challenge、WAF block、反复 403/429。
3. robots.txt、条款或页面提示明确禁止采集。
4. 批量采集个人数据、秘密、令牌、私信或非公开文件。
5. 用户要求继续突破限制、轮换身份、绕过封禁或隐藏自动化。

对于用户自有站点、内部 QA、归档或明确授权研究，也要保留低频、证据和停止条件。

---

## 6. 被阻断时的诊断输出

```text
无法继续采集，因为目标触发了停止条件。

目标:
状态:
停止原因:
已尝试的低影响策略:
建议:
- 使用官方 API/export/RSS/sitemap
- 缩小范围
- 提供明确授权或测试环境
- 改为人工抽样整理
```

---

## 7. 外部工具引入条件

只在以下情况引入外部工具：

1. 内置 fetch/browser 不能正确渲染公开页面。
2. 用户自有站点或明确授权的 QA/归档任务。
3. 需要解析特定公开格式，如 RSS、sitemap、CSV、PDF。
4. 用户明确要求使用特定工具，且不违反红线。

引入外部工具前必须确认依赖，不可假设已安装。

---

## 8. 最小交付格式

每次采集至少输出：

- 数据文件或结构化结果。
- 来源 URL 或 provenance 字段。
- attempted / fetched / skipped / failed 计数。
- 校验摘要与去重结果。
- 不完整区域和停止原因。
