---
name: query-optimizer
description: |
  Optimize database queries for performance using indexes, query analysis, and best practices.
  Identifies slow queries and implements fixes.
---

# Query Optimizer Skill

## Purpose
Identify and optimize slow database queries for better performance.

## Query Analysis

### PostgreSQL EXPLAIN
```sql
-- Basic explain
EXPLAIN SELECT * FROM posts WHERE author_id = 'abc123';

-- With execution stats
EXPLAIN ANALYZE SELECT * FROM posts WHERE author_id = 'abc123';

-- Full details
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM posts WHERE author_id = 'abc123';
```

### Reading EXPLAIN Output
```
Seq Scan on posts  (cost=0.00..1000.00 rows=100 width=200)
  Filter: (author_id = 'abc123')
```
- **Seq Scan**: Full table scan (bad for large tables)
- **cost**: Estimated startup..total cost
- **rows**: Estimated rows returned
- **Filter**: Condition applied after scan

```
Index Scan using idx_posts_author on posts  (cost=0.00..10.00 rows=100 width=200)
  Index Cond: (author_id = 'abc123')
```
- **Index Scan**: Using index (good!)
- **Index Cond**: Condition used with index

### Finding Slow Queries
```sql
-- PostgreSQL: Enable pg_stat_statements
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Find slowest queries
SELECT
  query,
  calls,
  mean_exec_time,
  total_exec_time,
  rows
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

## Index Strategies

### Single Column Index
```sql
-- Good for: WHERE column = value
CREATE INDEX idx_users_email ON users(email);

-- Query that uses it:
SELECT * FROM users WHERE email = 'user@example.com';
```

### Composite Index
```sql
-- Order matters! Left-to-right
CREATE INDEX idx_posts_author_status ON posts(author_id, status);

-- Uses index:
SELECT * FROM posts WHERE author_id = 'abc' AND status = 'PUBLISHED';
SELECT * FROM posts WHERE author_id = 'abc';

-- Does NOT use this index (missing leftmost column):
SELECT * FROM posts WHERE status = 'PUBLISHED';
```

### Covering Index
```sql
-- Include non-indexed columns to avoid table lookup
CREATE INDEX idx_posts_list ON posts(author_id, created_at DESC)
INCLUDE (title, status);

-- Query is fully satisfied by index:
SELECT title, status, created_at
FROM posts
WHERE author_id = 'abc'
ORDER BY created_at DESC;
```

### Partial Index
```sql
-- Index only rows matching condition
CREATE INDEX idx_posts_published ON posts(published_at DESC)
WHERE status = 'PUBLISHED';

-- Uses index:
SELECT * FROM posts
WHERE status = 'PUBLISHED'
ORDER BY published_at DESC;
```

### Expression Index
```sql
-- Index on expression result
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

-- Uses index:
SELECT * FROM users WHERE LOWER(email) = 'user@example.com';
```

## Query Optimization Patterns

### N+1 Query Problem
```typescript
// BAD: N+1 queries
const posts = await prisma.post.findMany();
for (const post of posts) {
  const author = await prisma.user.findUnique({
    where: { id: post.authorId }
  });
}

// GOOD: Include relation
const posts = await prisma.post.findMany({
  include: { author: true }
});

// GOOD: Select only needed fields
const posts = await prisma.post.findMany({
  include: {
    author: {
      select: { id: true, name: true }
    }
  }
});
```

### Pagination
```typescript
// BAD: OFFSET pagination (slow for large offsets)
const posts = await prisma.post.findMany({
  skip: 10000,
  take: 10
});

// GOOD: Cursor pagination
const posts = await prisma.post.findMany({
  take: 10,
  cursor: { id: lastPostId },
  skip: 1, // Skip the cursor itself
  orderBy: { createdAt: 'desc' }
});
```

### Select Only Needed Fields
```typescript
// BAD: Select all
const users = await prisma.user.findMany();

// GOOD: Select only needed
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true
  }
});
```

### Batch Operations
```typescript
// BAD: Individual inserts
for (const item of items) {
  await prisma.item.create({ data: item });
}

// GOOD: Batch insert
await prisma.item.createMany({
  data: items,
  skipDuplicates: true
});
```

### Efficient Counting
```sql
-- Slow: Exact count on large table
SELECT COUNT(*) FROM posts;

-- Fast: Approximate count
SELECT reltuples::bigint AS estimate
FROM pg_class
WHERE relname = 'posts';

-- Fast: Count with limit
SELECT COUNT(*) FROM (
  SELECT 1 FROM posts LIMIT 1000
) t;
```

## Common Performance Issues

### Missing Index
```sql
-- Problem: Full table scan
SELECT * FROM orders WHERE customer_id = 'abc';

-- Solution: Add index
CREATE INDEX idx_orders_customer ON orders(customer_id);
```

### Wrong Index Order
```sql
-- Existing index
CREATE INDEX idx_orders_status_date ON orders(status, created_at);

-- This query can't use it efficiently
SELECT * FROM orders WHERE created_at > '2024-01-01';

-- Solution: Create appropriate index
CREATE INDEX idx_orders_date ON orders(created_at);
```

### Over-fetching Data
```sql
-- Problem: Selecting all columns
SELECT * FROM posts WHERE id = 'abc';

-- Solution: Select only needed
SELECT id, title, excerpt FROM posts WHERE id = 'abc';
```

### Expensive JOINs
```sql
-- Problem: Multiple large table joins
SELECT *
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
JOIN categories c ON p.category_id = c.id;

-- Solutions:
-- 1. Add indexes on join columns
-- 2. Denormalize if read-heavy
-- 3. Use materialized views
-- 4. Cache results
```

## Monitoring

```typescript
// Prisma query logging
const prisma = new PrismaClient({
  log: [
    { level: 'query', emit: 'event' },
    { level: 'warn', emit: 'stdout' },
    { level: 'error', emit: 'stdout' },
  ],
});

prisma.$on('query', (e) => {
  if (e.duration > 100) { // Log slow queries (>100ms)
    console.warn('Slow query:', {
      query: e.query,
      duration: e.duration,
    });
  }
});
```

## Quality Checklist

- [ ] Foreign keys have indexes
- [ ] WHERE columns are indexed
- [ ] ORDER BY columns are indexed
- [ ] No N+1 queries
- [ ] Pagination uses cursors
- [ ] Only needed fields selected
- [ ] Batch operations used
- [ ] Slow queries logged
- [ ] EXPLAIN used for complex queries
