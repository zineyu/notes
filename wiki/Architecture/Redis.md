---
tags:
  - wiki
  - Architecture
  - page
  - Redis
---
> 参考来源：[Redis Anti-Patterns](https://redis.io/learn/howtos/antipatterns/)、[Redis Best Practices](https://redis.io/topics/best-practices)、[Redis Memory Optimization](https://redis.antirez.com/fundamental/memory-optimization.html)

---

## 核心原则

### 1. 使用合适的数据结构
为使用场景选择正确的 Redis 类型：Hashes 用于对象，Sorted Sets 用于排行榜/时间序列，Sets 用于唯一性，Strings 用于简单值。绝不要在 Strings 中存储 JSON 大对象。

### 2. 始终为缓存键设置 TTL
每个缓存键都应该有过期时间。没有 TTL，键会无限累积，导致无界内存增长、驱逐风暴和 OOM 错误。

### 3. 为内存效率设计
Redis 是内存数据库。每个键的开销约为 88-112 字节（数据之前）。使用短的、带命名空间的键（例如 `u:1001` 而非 `user:profile:1001`）。将小值聚合到 Hashes 中以减少每个键的开销。

### 4. 避免阻塞操作
绝不要使用 `KEYS`（O(N) 扫描）。使用 `SCAN` 进行迭代。对批量操作使用管道（pipelining）。避免在大型集合上使用复杂度无界的操作。

### 5. 为运维做规划
保持分片低于 25GB 或 25K ops/sec。使用连接池。使用 `redis-cli --hotkeys` 监控热键。当用作主存储时启用持久化（AOF/RDB）。

---

## 项目架构/目录规范

### Redis 键命名规范

```text
格式: namespace:type:id[:field]

示例:
u:1001              # 用户 1001
u:1001:p            # 用户 1001 资料（hash）
s:1001              # 用户 1001 的会话
c:up:1699123456     # /users/profile 端点的缓存
q:email             # 邮件队列
l:leaderboard:daily # 每日排行榜
```

### 内存优化策略

```text
# BAD: 每个属性单独的键（5 个键，5 倍开销）
SET user:1001:name "John"
SET user:1001:email "john@example.com"
SET user:1001:age "30"

# GOOD: 单个 hash（1 个键，小对象可节省约 80% 内存）
HSET u:1001 n "John" e "john@example.com" a "30"
```

---

## 标准代码示例

### 示例 1：连接池与管道（Go）

```go
import "github.com/redis/go-redis/v9"

// Good: 带超时设置的连接池
rdb := redis.NewClient(&redis.Options{
    Addr:         "localhost:6379",
    Password:     "",
    DB:           0,
    PoolSize:     10,           // 连接池大小
    MinIdleConns: 5,            // 保持预热连接
    MaxRetries:   3,            // 失败命令自动重试
    DialTimeout:  5 * time.Second,
    ReadTimeout:  3 * time.Second,
    WriteTimeout: 3 * time.Second,
})

// Good: 批量操作使用管道
pipe := rdb.Pipeline()
pipe.Set(ctx, "key1", "value1", 1*time.Hour)
pipe.Set(ctx, "key2", "value2", 1*time.Hour)
pipe.HSet(ctx, "u:1001", map[string]interface{}{
    "n": "John",
    "e": "john@example.com",
})
_, err := pipe.Exec(ctx)  // 所有命令单次往返
```

### 示例 2：正确使用 Hash 与 TTL

```python
import redis

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

# Good: 将对象存储为带 TTL 的 hash
user_data = {
    'n': 'John Doe',           # 短字段名节省内存
    'e': 'john@example.com',
    'a': '30',
    'r': 'admin'
}

# HSET 存储 hash，EXPIRE 设置 1 小时 TTL
r.hset('u:1001', mapping=user_data)
r.expire('u:1001', 3600)

# Good: 获取特定字段（而非整个对象）
name = r.hget('u:1001', 'n')
profile = r.hgetall('u:1001')
```

---

## 常见反模式

### 1. 使用 KEYS 命令

```bash
# BAD: 阻塞 Redis，O(N) 扫描整个键空间
KEYS user:*

# GOOD: 增量扫描不会阻塞
SCAN 0 MATCH user:* COUNT 100

# 或使用 Redis Search 进行基于内容的查询
FT.SEARCH userIdx "@name:John"
```

### 2. 在 Strings 中存储 JSON

```bash
# BAD: 无法原子更新字段，解析开销
SET user:1001 '{"name":"John","email":"john@example.com"}'

# GOOD: 使用 hash 实现字段级访问
HSET u:1001 n "John" e "john@example.com"
HGET u:1001 n  # 高效获取单个字段

# 或使用 RedisJSON 处理嵌套结构
JSON.SET user:1001 $ '{"name":"John","settings":{"theme":"dark"}}'
JSON.GET user:1001 $.settings.theme
```

### 3. 缓存键不设置 TTL

```bash
# BAD: 键永远增长直到手动删除或 OOM
SET cache:users:1001 "{...}"

# GOOD: 始终为缓存键设置过期时间
SET cache:users:1001 "{...}" EX 3600

# 或使用 Hash 为相关数据设置统一 TTL
HSET session:1001 data "{...}"
EXPIRE session:1001 86400
```

---

## 参考资源

- [Redis Anti-Patterns](https://redis.io/learn/howtos/antipatterns/)（官方）
- [Redis Best Practices](https://redis.io/topics/best-practices)（官方）
- [Redis Memory Optimization](https://redis.antirez.com/fundamental/memory-optimization.html)（官方）
- [Redis Key Naming Conventions](https://oneuptime.com/blog/post/2026-01-25-redis-key-naming-conventions/view)（社区）

---

## 相关笔记

- [[数据库设计]] — 数据存储的整体规划
- [[SQL]] — 关系型数据库查询语言
