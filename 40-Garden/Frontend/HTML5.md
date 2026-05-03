# HTML5 最佳实践

> 来源：[MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTML)、[HTML Living Standard](https://html.spec.whatwg.org/)

## 核心原则

1. **使用语义化标签** — 用正确的标签做正确的事，如 `<header>`、`<nav>`、`<main>`、`<article>`、`<section>`、`<aside>`、`<footer>`
2. **`<main>` 每页仅使用一次** — 直接放在 `<body>` 内，不嵌套在其他元素中
3. **为辅助技术提供结构** — 屏幕阅读器依赖语义化标签导航内容
4. **避免滥用 `<div>` 和 `<span>`** — 仅在无更合适语义标签时使用
5. **保持文档结构清晰** — 使用恰当的标题层级 (`<h1>` ~ `<h6>`)，每个 `<section>` 以标题开头

## 项目目录规范

```
project/
├── index.html              # 入口页面
├── about/
│   └── index.html
├── contact/
│   └── index.html
├── css/
│   └── styles.css
├── js/
│   └── main.js
├── images/
│   └── logo.png
└── favicon.ico
```

## 标准代码示例

### 语义化文档结构

```html
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Page Title</title>
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <header>
      <h1>Site Title</h1>
      <nav>
        <ul>
          <li><a href="/">Home</a></li>
          <li><a href="/about">About</a></li>
        </ul>
      </nav>
    </header>

    <main>
      <article>
        <h2>Article Title</h2>
        <section>
          <h3>Subsection</h3>
          <p>Content...</p>
        </section>
      </article>
      <aside>
        <h2>Related</h2>
        <ul>
          <li><a href="#">Link</a></li>
        </ul>
      </aside>
    </main>

    <footer>
      <p>© 2026 Company Name</p>
    </footer>
  </body>
</html>
```

### 语义化表单

```html
<form action="/submit" method="POST">
  <fieldset>
    <legend>Contact Information</legend>
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required />

    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required />

    <label for="message">Message:</label>
    <textarea id="message" name="message" rows="5"></textarea>

    <button type="submit">Send</button>
  </fieldset>
</form>
```

### 图片可访问性

```html
<figure>
  <img
    src="photo.jpg"
    alt="A bronze statue of two crossed hands delicately holding a human brain"
    width="400"
    height="300"
  />
  <figcaption>
    Homage to Neurosurgery by Marta Colvin Andrade
  </figcaption>
</figure>
```

## 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| 全部使用 `<div>` 包裹 | 失去语义，影响可访问性 | 使用 `<header>`、`<main>`、`<article>` 等语义标签 |
| 跳过标题层级 | 破坏文档大纲 | 保持标题层级连续（h1 → h2 → h3） |
| 为样式而非语义使用标签 | 误导辅助技术 | 用 CSS 实现视觉效果，HTML 负责语义 |

---

**参考来源**：
- [MDN - Structuring Documents](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Structuring_documents)
- [HTML Living Standard](https://html.spec.whatwg.org/)

---

## 相关笔记

- [[CSS]] — 样式层最佳实践
- [[JavaScript]] — 行为层与 DOM 操作
