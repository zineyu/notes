---
tags:
  - wiki
  - Frontend
  - page
  - Tailwind
  - CSS
---
# Tailwind CSS 最佳实践

> 来源：[Tailwind CSS 官方文档](https://tailwindcss.com/docs)、[Utility-First Fundamentals](https://tailwindcss.com/docs/styling-with-utility-classes)

## 核心原则

1. **Utility-first 设计** — 使用原子类直接在 HTML/JSX 中组合样式
2. **使用设计系统约束** — 从预定义的设计令牌中选择，避免魔法数字
3. **使用组件封装重复模式** — React/Vue 组件封装重复样式
4. **使用变体处理状态** — `hover:`、`focus:`、`dark:`、`sm:` 等
5. **使用 `@layer components` 创建抽象** — 对于简单重复模式使用自定义 CSS

## 项目目录规范

```
src/
├── components/             # React 组件封装 Tailwind 类
│   ├── Button.jsx
│   └── Card.jsx
├── styles/
│   └── main.css            # @import "tailwindcss";
├── app.css                 # 自定义组件层
└── App.jsx
```

## 标准代码示例

### Utility-first 组件

```jsx
// ✅ Good: 使用原子类构建组件
function Card({ title, description, imageUrl }) {
  return (
    <div className="mx-auto flex max-w-sm items-center gap-x-4 rounded-xl bg-white p-6 shadow-lg outline outline-black/5 dark:bg-slate-800 dark:shadow-none dark:-outline-offset-1 dark:outline-white/10">
      <img
        className="size-12 shrink-0"
        src={imageUrl}
        alt={title}
      />
      <div>
        <div className="text-xl font-medium text-black dark:text-white">
          {title}
        </div>
        <p className="text-gray-500 dark:text-gray-400">
          {description}
        </p>
      </div>
    </div>
  );
}
```

### 响应式 + 暗色模式

```jsx
// ✅ Good: 响应式设计和暗色模式
function ProfileCard({ user }) {
  return (
    <div className="flex flex-col gap-2 p-8 sm:flex-row sm:items-center sm:gap-6 sm:py-4">
      <img
        className="mx-auto block h-24 rounded-full sm:mx-0 sm:shrink-0"
        src={user.avatar}
        alt=""
      />
      <div className="space-y-2 text-center sm:text-left">
        <p className="text-lg font-semibold text-black">{user.name}</p>
        <p className="font-medium text-gray-500">{user.role}</p>
        <button className="border-purple-200 text-purple-600 hover:border-transparent hover:bg-purple-600 hover:text-white active:bg-purple-700">
          Message
        </button>
      </div>
    </div>
  );
}
```

### 自定义组件类

```css
/* app.css */
@import "tailwindcss";

@layer components {
  .btn-primary {
    border-radius: calc(infinity * 1px);
    background-color: var(--color-violet-500);
    padding-inline: --spacing(5);
    padding-block: --spacing(2);
    font-weight: var(--font-weight-semibold);
    color: var(--color-white);
    box-shadow: var(--shadow-md);

    &:hover {
      @media (hover: hover) {
        background-color: var(--color-violet-700);
      }
    }
  }
}
```

## 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| 到处复制粘贴 20+ 个类名 | 重复、难以维护 | 提取为组件或自定义类 |
| 使用 `!important` 解决冲突 | 破坏层叠 | 使用组件属性控制样式 |
| `className="grid flex"` 冲突类 | 后加载的类生效，行为不可预测 | 条件渲染选择单一类 |
| 使用任意值滥用魔法数字 | 破坏设计系统一致性 | 优先使用主题值 |
| 全部用内联样式 | 无法处理 hover/focus/响应式 | 使用 Tailwind 工具类 |

---

**参考来源**：
- [Tailwind CSS Official Documentation](https://tailwindcss.com/docs)
- [Utility-First Fundamentals](https://tailwindcss.com/docs/styling-with-utility-classes)

---

## 相关笔记

- [[CSS]] — Tailwind 的底层 CSS 原理
- [[React]] — 常与 Tailwind 搭配使用的 UI 框架
