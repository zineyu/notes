---
tags:
  - Frontend
---

# TypeScript 最佳实践

> 来源：[TypeScript Official Docs](https://www.typescriptlang.org/docs/)、[TypeScript Do's and Don'ts](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)

## 核心原则

1. **启用严格模式** (`strict: true`) — 最大化类型安全性
2. **不使用大写包装类型** — `Number`/`String`/`Boolean` → `number`/`string`/`boolean`
3. **避免使用 `any`** — 使用 `unknown` 或具体类型
4. **回调返回值用 `void` 而非 `any`** — 防止意外使用返回值
5. **函数重载从具体到一般排序** — TypeScript 选择第一个匹配的重载

## 项目目录规范

```
src/
├── types/                  # 全局类型定义
│   ├── api.ts
│   └── models.ts
├── utils/
├── components/
├── hooks/
└── index.ts

tsconfig.json               # 严格模式配置
```

### 推荐 tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "allowImportingTsExtensions": true
  }
}
```

## 标准代码示例

### 类型定义最佳实践

```typescript
// ❌ Don't: 使用包装类型
function reverse(s: String): String;

// ✅ Do: 使用原始类型
function reverse(s: string): string {
  return s.split('').reverse().join('');
}

// ❌ Don't: 使用 any
function processData(data: any): any;

// ✅ Do: 使用 unknown + 类型守卫
function processData(data: unknown): string {
  if (typeof data === 'string') {
    return data.toUpperCase();
  }
  throw new Error('Invalid data');
}
```

### 接口与类型别名

```typescript
// ✅ Do: 使用接口描述对象形状
interface User {
  id: number;
  name: string;
  email: string;
  role?: 'admin' | 'user' | 'guest';
}

// ✅ Do: 使用类型别名描述联合类型
type Status = 'idle' | 'loading' | 'success' | 'error';

// ✅ Do: 函数重载从具体到一般
function processElement(element: HTMLDivElement): string;
function processElement(element: HTMLElement): string;
function processElement(element: unknown): string {
  // implementation
}
```

## 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| `function fn(x: () => any)` | 关闭类型检查，可能导致运行时错误 | `function fn(x: () => void)` |
| `diff(one: string): number; diff(one: string, two: string): number` | 不必要的重载 | 使用可选参数 `diff(one: string, two?: string)` |
| 泛型不使用类型参数 `interface Foo<T> {}` | 无意义的泛型 | 移除未使用的泛型参数 |

---

**参考来源**：
- [TypeScript Official Documentation](https://www.typescriptlang.org/docs/)
- [TypeScript Do's and Don'ts](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)

---

## 相关笔记

- [[JavaScript]] — TypeScript 的编译目标与基础语言
- [[React]] — 与 TypeScript 深度集成的 UI 框架
- [[Vite]] — 原生支持 TypeScript 的构建工具
