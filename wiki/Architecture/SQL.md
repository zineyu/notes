---
tags:
  - wiki
  - Architecture
  - page
  - SQL
---
> 参考来源：[PostgreSQL Documentation](https://www.postgresql.org/docs/current/)、[PostgreSQL Performance Tips](https://www.postgresql.org/docs/current/performance-tips.html)、SQL Query Optimization Guide

---

## 核心原则

### 1. 始终使用 EXPLAIN ANALYZE
在优化之前，先理解查询计划。查找大表上的顺序扫描、错误的行估算和昂贵的排序。绝不盲目优化。

### 2. 战略性索引
在 WHERE、JOIN 和 ORDER BY 子句中使用的列上创建索引。复合索引优先放置等值列，范围列放最后。使用覆盖索引（INCLUDE）实现仅索引扫描。

### 3. 编写精确的查询
始终指定确切的列，而不是 `SELECT *`。这减少 I/O，启用仅索引扫描，并使查询对模式变化更健壮。

### 4. 优先使用 EXISTS/NOT EXISTS 而非 IN/NOT IN
对于子查询，`EXISTS` 在首次匹配时停止，而 `IN` 可能物化整个结果集。`NOT IN` 在存在 NULL 时会意外返回空结果。

### 5. 使用键集分页（Keyset Pagination）
对于大型结果集，避免使用 `OFFSET`（它是 O(n)）。使用 `WHERE id > last_seen ORDER BY id LIMIT N` 实现 O(1) 分页，无论深度如何。

---

## 项目架构/目录规范

### 数据库项目结构

```text
database/
├── migrations/
│   ├── 001_create_users.up.sql
│   ├── 001_create_users.down.sql
│   ├── 002_create_orders.up.sql
│   └── 002_create_orders.down.sql
├── schema/
│   ├── tables/
│   ├── views/
│   ├── functions/
│   └── triggers/
├── seeds/
│   └── dev_data.sql
├── scripts/
│   ├── backup.sh
│   └── analyze_tables.sh
└── docker-compose.yml
```

### Schema 设计最佳实践
- 每个表都应该有合成主键（BIGSERIAL 或 UUID）
- 用 UNIQUE 约束保护自然键，不要将其用作主键
- 使用适当的数据类型（TIMESTAMPTZ、UUID、JSONB）
- 设置显式列约束（NOT NULL、CHECK、DEFAULT）

---

## 标准代码示例

### 示例 1：正确的索引设计

```sql
-- BAD: 无索引，导致顺序扫描
SELECT * FROM orders WHERE status = 'pending' AND created_at > '2026-01-01';

-- GOOD: 复合索引，等值列在前，范围列在后
CREATE INDEX idx_orders_status_created 
ON orders(status, created_at DESC);

-- GOOD: 覆盖索引实现仅索引扫描
CREATE INDEX idx_orders_covering 
ON orders(customer_id) 
INCLUDE (total, status, created_at);

-- 此查询现在只读取索引，不读取表：
SELECT total, status, created_at 
FROM orders 
WHERE customer_id = 123;
```

### 示例 2：高效分页

```sql
-- BAD: OFFSET 随着偏移量增长而变慢（O(n)）
SELECT * FROM posts 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 100000;

-- GOOD: 键集分页（O(1)）
SELECT * FROM posts 
WHERE created_at < '2026-04-20T10:00:00Z'  -- 上次看到的时间戳
ORDER BY created_at DESC 
LIMIT 20;

-- 使用 ID 的替代方案：
SELECT * FROM posts 
WHERE id < :last_seen_id 
ORDER BY id DESC 
LIMIT 20;
```

---

## 常见反模式

### 1. SELECT *

```sql
-- BAD: 获取不必要的列，阻止仅索引扫描
SELECT * FROM users WHERE email = 'user@example.com';

-- GOOD: 只获取需要的列
SELECT id, name, created_at 
FROM users 
WHERE email = 'user@example.com';
```

### 2. 在索引列上使用函数

```sql
-- BAD: 阻止索引使用（函数包装了列）
SELECT * FROM orders 
WHERE DATE(created_at) = '2026-01-15';

-- GOOD: 范围查询使用索引
SELECT * FROM orders 
WHERE created_at >= '2026-01-15' 
  AND created_at < '2026-01-16';
```

### 3. 大规模 OFFSET 分页

```sql
-- BAD: OFFSET 100000 强制读取并丢弃 10万行
SELECT * FROM products 
ORDER BY id 
LIMIT 20 OFFSET 100000;

-- GOOD: 使用 WHERE 子句的键集分页
SELECT * FROM products 
WHERE id > 100000 
ORDER BY id 
LIMIT 20;
```

---

## 参考资源

- [PostgreSQL Documentation](https://www.postgresql.org/docs/current/)（官方）
- [PostgreSQL Performance Tips](https://www.postgresql.org/docs/current/performance-tips.html)（官方）
- [SQL Query Optimization Guide](https://blogs.pavanrangani.com/sql-query-optimization-postgresql-performance/)（社区）

---

## 相关笔记

- [[数据库设计]] — 数据库 schema 设计规范
- [[Redis]] — 缓存层与关系型数据库的互补方案
