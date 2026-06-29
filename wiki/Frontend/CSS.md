---
tags:
  - wiki
  - Frontend
  - page
  - CSS
---
# CSS 最佳实践

> 来源：[MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS)、[BEM Methodology](https://getbem.com/naming/)、[OOCSS](https://github.com/stubbornella/oocss/wiki)

## 核心原则

1. **使用 BEM 命名规范** — Block `__` Element `--` Modifier，如 `.card__title--highlighted`
2. **避免过度具体的选择器** — 优先使用类选择器，减少嵌套
3. **分离结构与表现** — OOCSS 思想：提取可复用模式
4. **使用 CSS 自定义属性（变量）** — 统一管理颜色、间距等设计令牌
5. **移动优先的响应式设计** — 先写基础样式，再用媒体查询增强

## 项目目录规范

```
styles/
├── base/
│   ├── reset.css           # CSS Reset / Normalize
│   ├── typography.css      # 字体、行高等
│   └── variables.css       # CSS 自定义属性
├── components/
│   ├── button.css
│   ├── card.css
│   └── nav.css
├── layouts/
│   ├── grid.css
│   └── header.css
├── utilities/
│   └── helpers.css
└── main.css                # @import 所有模块
```

## 标准代码示例

### BEM + CSS 变量

```css
/* variables.css */
:root {
  --color-primary: #3b82f6;
  --color-text: #1f2937;
  --spacing-md: 1rem;
  --radius-lg: 0.5rem;
}

/* button.css */
.btn {
  display: inline-flex;
  align-items: center;
  padding: var(--spacing-md) calc(var(--spacing-md) * 2);
  border-radius: var(--radius-lg);
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}

.btn--primary {
  background-color: var(--color-primary);
  color: white;
}

.btn--primary:hover {
  background-color: color-mix(in srgb, var(--color-primary) 80%, black);
}

.btn--large {
  font-size: 1.125rem;
  padding: 1rem 2rem;
}
```

### OOCSS 媒体对象模式

```css
.media {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: var(--spacing-md);
  align-items: start;
}

.media__image {
  width: 64px;
  height: 64px;
  border-radius: var(--radius-lg);
}

.media__content {
  font-size: 0.875rem;
  color: var(--color-text);
}
```

## 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| `article.main p.box` 过度具体 | 难以复用，增加特异性战争 | 使用 `.box` 类直接选择 |
| 使用 `!important` 解决冲突 | 破坏层叠，难以维护 | 降低选择器特异性 |
| 内联样式管理布局 | 无法响应式，无法复用 | 使用 CSS 类 + 媒体查询 |
| 每个组件写死颜色值 | 难以维护主题 | 使用 CSS 自定义属性 |

---

**参考来源**：
- [MDN - CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [BEM Methodology](https://getbem.com/naming/)
- [OOCSS Wiki](https://github.com/stubbornella/oocss/wiki)

---

## 相关笔记

- [[HTML5]] — 结构层最佳实践
- [[Tailwind CSS]] — 原子化 CSS 框架
