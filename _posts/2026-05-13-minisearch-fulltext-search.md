---
title: "给博客加个搜索引擎：MiniSearch 实战"
date: 2026-05-13
categories:
  - 技术
tags:
  - MiniSearch
  - JavaScript
  - 前端
  - Jekyll
  - 博客
---

最近给博客加了个全文搜索功能，选型时在 MiniSearch 和 Lunr 之间纠结了一下。写篇文章记录下调研过程和实现方案，希望能帮到有类似需求的朋友。

## 什么是 MiniSearch？

**MiniSearch** 是一个轻量级的 JavaScript 全文搜索引擎，能在浏览器和 Node.js 两端运行。它使用倒排索引（Inverted Index）来实现高效的文本搜索——建索引时慢一点，但搜索时直接查字典，速度极快。

核心特点：

- **体积极小**：gzip 后仅 **~6KB**
- **零依赖**：一个 npm install 就能用
- **功能齐全**：支持全文搜索、模糊匹配、前缀搜索、自动补全、权重排序、布尔查询
- **TypeScript 原生支持**：类型定义开箱即用
- **活跃维护**：当前最新版本 v7.2.0，社区活跃

安装非常简单：

```bash
npm install minisearch
```

或者直接在浏览器中用 CDN：

```html
<script src="https://cdn.jsdelivr.net/npm/minisearch@7.2.0/dist/umd/index.min.js"></script>
```

## MiniSearch vs Lunr：怎么选？

Lunr 是老牌的前端搜索库，很多人听说过。但在实际对比后，我发现 MiniSearch 在多个维度上更胜一筹。

| 维度 | MiniSearch (v7.2.0) | Lunr (v2.3.9) |
|------|-------------------|---------------|
| 包体积 | **~6KB gzip** 🔥 | ~24KB gzip |
| 搜索速度 | **更快** | 中等 |
| 中文分词 | ✅ 可配置 tokenizer | ❌ 原生不支持 |
| 模糊搜索 | ✅ 内置 `fuzzy` 参数 | 有限（通配符） |
| 前缀搜索 | ✅ 需配置 `prefix: true` | 默认支持 |
| 自动补全 | ✅ 原生支持 | ❌ |
| TypeScript | ✅ 原生 | 第三方类型定义 |
| 模块格式 | ✅ ESM + CJS + UMD | 仅 CJS |
| 包体更新 | 🟢 活跃（2025 年仍有提交） | 🟡 近乎停滞 |
| GitHub Stars | ~5.9k | ~9.2k |

几个关键差异展开说说：

### 1. 中文搜索

这是最关键的差距。Lunr 的分词器是按空格和标点切分的，对中文基本无效。比如搜"搜索引擎"，Lunr 会把整句当做一个词，什么都搜不到。

MiniSearch 允许自定义 `tokenize` 函数，一行代码就能处理好中文：

```js
const miniSearch = new MiniSearch({
  fields: ['title', 'content'],
  tokenize: (text) => text.split(
    /[\s,.;:!?()\[\]{}\"'，。；：！？（）【】""'']+|(?=[\u4e00-\u9fff])/
  ).filter(Boolean)
})
```

### 2. 维护状态

Lunr 的最后一个版本 v2.3.9 发布于 2022 年，之后就没有实质性更新了。MiniSearch 目前还在活跃维护中，新功能、bug 修复持续在出。

### 3. 搜索质量

MiniSearch 提供了更丰富的搜索功能：

```js
miniSearch.search('react', {
  prefix: true,    // 前缀匹配："rea"能搜到"react"
  fuzzy: 0.2      // 模糊匹配：拼写错误也能找到
})
```

搜索结果还能按字段权重排序，标题匹配的权重远高于正文匹配，用户体验更好。

## 实际应用：给我的 Jekyll 博客加搜索

这里分享一下我是怎么把它集成到 GitHub Pages 博客的。如果你也是用 Jekyll + Minimal Mistakes 主题，可以直接复用。

### 第 1 步：生成索引数据

在博客根目录新建 `search.json`，Jekyll 构建时会自动生成所有文章的索引数据：

```liquid
---
layout: null
---
[
  {% for post in site.posts %}
  {
    "id": "{{ post.url | slugify }}",
    "title": {{ post.title | jsonify }},
    "url": "{{ post.url | relative_url }}",
    "date": "{{ post.date | date: '%Y-%m-%d' }}",
    "categories": [{% if post.categories %}{% for cat in post.categories %}{{ cat | jsonify }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endif %}],
    "tags": [{% if post.tags %}{% for tag in post.tags %}{{ tag | jsonify }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endif %}],
    "content": {{ post.content | strip_html | normalize_whitespace | jsonify }}
  }{% unless forloop.last %},{% endunless %}
  {% endfor %}
]
```

构建后在 `https://你的博客/search.json` 就能访问到完整的文章索引数据。

### 第 2 步：初始化 MiniSearch

```js
fetch('/search.json')
  .then(res => res.json())
  .then(data => {
    const miniSearch = new MiniSearch({
      fields: ['title', 'content', 'tags', 'categories'],
      storeFields: ['title', 'url', 'date', 'tags', 'categories'],
      searchOptions: {
        boost: { title: 4, tags: 2, categories: 1.5, content: 1 },
        prefix: true,
        fuzzy: 0.2,
      }
    })
    miniSearch.addAll(data)
  })
```

几个关键配置：

- **`boost`**：不同字段的搜索权重。标题匹配最重要（权重 4），标签次之（权重 2），正文权重最低（权重 1）
- **`prefix: true`**：开启前缀匹配，输入"Her"能找到"Hermes"
- **`fuzzy: 0.2`**：模糊匹配容忍度，拼写小错误也能搜到结果

### 第 3 步：响应搜索输入

```js
const results = miniSearch.search(query, { prefix: true, fuzzy: 0.2 })

// 渲染结果
results.slice(0, 20).forEach(item => {
  // 展示 title、date、tags、content excerpt
})
```

### 第 4 步：搜索 UI

我做了个悬浮搜索面板（类似 Alfred / Cmd+K 的体验）：

- 导航栏右侧的 🔍 按钮打开搜索
- 键盘快捷键 `Cmd/Ctrl + K` 或 `/` 直接唤出
- `ESC` 或点击遮罩层关闭
- 移动端点击站点标题也能打开

搜索框风格跟博客的暗黑赛博朋克主题保持一致——黑色背景、绿色边框、等宽字体。

## 性能表现

对于个人博客这种数据量（几十到几百篇文章），搜索速度是**毫秒级的**，完全感觉不到延迟。即使数据量增长到数万篇，MiniSearch 也能轻松应对。

官方数据参考：

| 文档数 | 内存占用 | 搜索时间 |
|--------|----------|----------|
| 1 万 | ~50MB | < 10ms |
| 10 万 | ~300MB | ~20ms |
| 100 万 | ~500MB | ~80ms |

## 总结

如果你需要一个前端全文搜索方案，**MiniSearch 是目前的最佳选择**——体积小、功能强、中文友好、维护活跃。它跟 Lunr 是同类产品，但在几乎所有维度上都表现更好。

对于 Jekyll 博客来说，配合 `search.json` 数据源，前后不到 100 行代码就能搞定一个体验不错的站内搜索。

如果不想折腾，Minimal Mistakes 主题自带 Lunr 搜索支持，在 `_config.yml` 里设置 `search: true` 就能开启。但如果你有中文搜索需求，还是 MiniSearch 更值得花那点配置时间。

---

**参考链接：**
- [MiniSearch 官网](https://lucaong.github.io/minisearch/)
- [MiniSearch GitHub](https://github.com/lucaong/minisearch)
- [Lunr.js](https://lunrjs.com/)
