---
tags:
  - wiki
  - Frontend
  - page
---
# JavaScript 最佳实践

> 来源：[MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript)、[Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) (148k ⭐)

## 核心原则

1. **使用 `const`，需要重新赋值时用 `let`，不用 `var`** — 块级作用域更安全
2. **优先使用对象/数组字面量** — 不用 `new Object()` / `new Array()`
3. **使用解构赋值** — 减少临时变量，代码更简洁
4. **使用模板字符串** — 替代字符串拼接
5. **优先使用对象展开语法 `...`** — 替代 `Object.assign`

## 项目目录规范

```
src/
├── utils/                  # 纯函数工具
│   ├── formatDate.js
│   └── validators.js
├── components/             # UI 组件
├── services/               # API 调用
├── hooks/                  # 自定义 Hooks
├── constants/              # 常量
├── types/                  # 类型定义 (TS)
└── index.js
```

## 标准代码示例

### 现代 JavaScript 风格

```javascript
// ✅ Good: 使用 const + 解构 + 模板字符串
const user = { firstName: 'John', lastName: 'Doe', age: 30 };
const { firstName, lastName } = user;

const getFullName = ({ firstName, lastName }) => `${firstName} ${lastName}`;
const greeting = `Hello, ${getFullName(user)}!`;

// ✅ Good: 对象展开复制
const updatedUser = { ...user, age: 31 };

// ✅ Good: 数组方法
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((n) => n * 2);
const evens = numbers.filter((n) => n % 2 === 0);
const sum = numbers.reduce((acc, n) => acc + n, 0);
```

### 函数最佳实践

```javascript
// ✅ Good: 默认参数 + Rest 参数
function createUser(name, options = {}) {
  const { role = 'user', active = true } = options;
  return { name, role, active };
}

// ✅ Good: 箭头函数作为回调
const items = ['a', 'b', 'c'];
const uppercased = items.map((item) => item.toUpperCase());

// ✅ Good: 使用 Rest 参数替代 arguments
const sum = (...numbers) => numbers.reduce((a, b) => a + b, 0);
```

## 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| 使用 `var` | 函数作用域，变量提升导致 bug | 使用 `const` / `let` |
| `function foo() {}` 函数声明 | 提升导致可读性差 | 使用 `const foo = () => {}` |
| `Object.assign({}, obj)` | 冗长且可能修改原对象 | 使用 `{ ...obj }` |
| `"Hello " + name + "!"` | 难以阅读和维护 | 使用模板字符串 `` `Hello ${name}!` `` |
| 直接修改函数参数 | 副作用，可能导致意外行为 | 创建新对象返回 |

---

**参考来源**：
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [MDN - JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

---

## 相关笔记

- [[TypeScript]] — JavaScript 的超集与类型系统
- [[React]] — 基于 JavaScript 的声明式 UI 框架
- [[HTML5]] — 结构层与 DOM 基础
