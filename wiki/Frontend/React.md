# React 最佳实践

> 来源：[React.dev 官方文档](https://react.dev/learn)、[React Thinking in React](https://react.dev/learn/thinking-in-react)

## 核心原则

1. **组件即函数** — 组件是返回 JSX 的 JavaScript 函数，名称为大写
2. **使用 Hooks 管理状态** — `useState`、`useEffect` 等，仅在顶层调用
3. **状态提升共享数据** — 将状态放到最近的共同父组件
4. **列表渲染提供稳定 `key`** — 帮助 React 识别元素身份
5. **Props 向下传递，事件向上传递** — 单向数据流

## 项目目录规范

```
src/
├── components/             # 可复用组件
│   ├── ui/                 # 基础 UI 组件 (Button, Input)
│   └── layout/             # 布局组件 (Header, Sidebar)
├── pages/                  # 页面级组件
├── hooks/                  # 自定义 Hooks
│   └── useAuth.ts
├── contexts/               # React Context
├── services/               # API 调用
├── utils/                  # 工具函数
├── types/                  # TypeScript 类型
└── App.tsx
```

## 标准代码示例

### 组件与状态管理

```jsx
import { useState } from 'react';

// ✅ Good: 组件名大写，使用 Hooks
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Clicked {count} times
    </button>
  );
}

// ✅ Good: 状态提升
function App() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <Counter count={count} onClick={handleClick} />
      <Counter count={count} onClick={handleClick} />
    </div>
  );
}

function Counter({ count, onClick }) {
  return <button onClick={onClick}>Clicked {count} times</button>;
}
```

### 列表渲染与条件渲染

```jsx
// ✅ Good: 列表使用 key
function ProductList({ products }) {
  const listItems = products.map((product) => (
    <li key={product.id} className="product-item">
      {product.title}
    </li>
  ));

  return <ul>{listItems}</ul>;
}

// ✅ Good: 条件渲染
function UserGreeting({ isLoggedIn, user }) {
  return (
    <div>
      {isLoggedIn ? (
        <h1>Welcome back, {user.name}!</h1>
      ) : (
        <h1>Please sign in.</h1>
      )}
    </div>
  );
}
```

## 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| 在循环/条件中调用 Hooks | 破坏 Hooks 调用顺序 | 仅在组件顶层调用 Hooks |
| 使用数组索引作为 `key` | 重排时状态混乱 | 使用唯一稳定 ID |
| 直接修改 state (`state.value = 1`) | 不触发重新渲染 | 使用 setter 函数 |
| 事件处理器直接调用 `handleClick()` | 页面加载时立即执行 | 传递引用 `onClick={handleClick}` |
| 组件文件过大 | 难以维护 | 拆分为小组件 |

---

**参考来源**：
- [React.dev - Learn](https://react.dev/learn)
- [React.dev - Thinking in React](https://react.dev/learn/thinking-in-react)

---

## 相关笔记

- [[JavaScript]] — React 的核心语言基础
- [[TypeScript]] — React 项目的类型安全实践
- [[Tailwind CSS]] — React 中常用的原子化 CSS 框架
- [[Vite]] — React 项目推荐的构建工具
