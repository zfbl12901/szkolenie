# 4. Performance and Optimization

## 🎯 Objectives

- Understand ClickHouse indexes
- Optimize queries
- Manage compression
- Monitor performance

## Indexes and Projections

### Primary index (ORDER BY)

```sql
CREATE TABLE events
(
    id UInt64,
    event_date Date,
    user_id UInt32
)
ENGINE = MergeTree()
ORDER BY (event_date, user_id);  -- Primary index
```

## Query Optimization

### Use WHERE on indexed columns

```sql
-- ✅ Good : uses index
SELECT * FROM events 
WHERE event_date = '2024-01-15';

-- ❌ Less good : full scan
SELECT * FROM events 
WHERE value > 100;
```

### LIMIT to limit results

```sql
SELECT * FROM events 
ORDER BY event_date DESC 
LIMIT 100;
```

### Avoid SELECT *

```sql
-- ✅ Good
SELECT event_date, COUNT(*) 
FROM events 
GROUP BY event_date;

-- ❌ Less good
SELECT * FROM events;
```

## Compression

### Check compression

```sql
SELECT 
    table,
    formatReadableSize(sum(data_compressed_bytes)) as compressed,
    formatReadableSize(sum(data_uncompressed_bytes)) as uncompressed,
    round(sum(data_uncompressed_bytes) / sum(data_compressed_bytes), 2) as ratio
FROM system.parts
WHERE active
GROUP BY table;
```

## Monitoring

### Slow queries

```sql
SELECT 
    query,
    query_duration_ms,
    read_rows,
    read_bytes
FROM system.query_log
WHERE type = 'QueryFinish'
ORDER BY query_duration_ms DESC
LIMIT 10;
```

---

**Next step :** [Advanced Functions](./05-fonctions-avancees/README.md)

