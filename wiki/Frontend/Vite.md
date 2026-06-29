---
tags:
  - wiki
  - Frontend
  - page
---
# Vite 最佳实践

> 来源：[Vite 官方文档](https://vitejs.dev/guide/)、[Vite Performance Guide](https://vitejs.dev/guide/performance.html)

## 核心原则

1. **明确导入路径扩展名** — 如 `import './Component.jsx'`，避免解析开销
2. **避免 Barrel Files** — 直接导入具体模块，减少不必要的加载
3. **使用 `server.warmup` 预热常用文件** — 减少请求瀑布
4. **优先使用原生 CSS** — 减少预处理开销（如 Sass/Less）
5. **配置 `resolve.extensions` 最小化** — 减少文件系统检查次数

## 项目目录规范

```
project/
├── public/                 # 静态资源 (不经过构建)
│   └── favicon.ico
├── src/
│   ├── assets/             # 会被处理的资源
│   ├── components/
│   ├── pages/
│   ├── styles/
│   ├── utils/
│   ├── App.jsx
│   └── main.jsx
├── index.html              # Vite 入口 (放在根目录)
├── vite.config.js
├── package.json
└── tsconfig.json
```

### 推荐 vite.config.js

```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    // 最小化扩展名列表
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],
  },
  server: {
    // 预热常用文件
    warmup: {
      clientFiles: [
        './src/components/App.jsx',
        './src/utils/api.js',
      ],
    },
  },
  build: {
    target: 'es2022',
    sourcemap: true,
  },
});
```

## 标准代码示例

### TypeScript + Vite 配置

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [tailwindcss()],
  // 明确路径减少解析开销
  resolve: {
    alias: {
      '@': '/src',
      '@components': '/src/components',
    },
  },
});
```

### 避免 Barrel File

```javascript
// ❌ Bad: barrel file (src/utils/index.js)
export * from './color.js';
export * from './dom.js';
export * from './slash.js';

// 使用者: import { slash } from './utils'
// 问题: 所有文件都被加载和转换

// ✅ Good: 直接导入
import { slash } from './utils/slash.js';
```

## 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| `import './Component'` 省略扩展名 | Vite 需要检查多个扩展名 | `import './Component.jsx'` |
| Barrel files 重导出 | 加载和转换不必要的文件 | 直接导入具体模块 |
| 开发时开启浏览器"禁用缓存" | 大幅降低启动和热更新速度 | 保持缓存开启 |
| 过度使用 Sass/Less | 增加构建时间 | 优先使用原生 CSS + PostCSS |

---

**参考来源**：
- [Vite Official Documentation](https://vitejs.dev/guide/)
- [Vite Performance Guide](https://vitejs.dev/guide/performance.html)

---

## 相关笔记

- [[React]] — Vite 生态中最常用的 UI 框架
- [[TypeScript]] — Vite 原生支持的类型系统
- [[JavaScript]] — Vite 处理的底层语言
