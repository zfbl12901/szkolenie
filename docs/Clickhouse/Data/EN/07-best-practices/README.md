# 7. Best Practices

## 🎯 Objectives

- Model data efficiently
- Choose partitioning strategies
- Manage memory
- Secure access

## Data Modeling

### Choose right types

```sql
-- ✅ Good : UInt32 for IDs
user_id UInt32

-- ❌ Less good : UInt64 unnecessary
user_id UInt64

-- ✅ Good : Date for dates
event_date Date

-- ❌ Less good : String for dates
event_date String
```

## Partitioning Strategies

### By date (recommended)

```sql
PARTITION BY toYYYYMM(event_date)
```

### Avoid too many partitions

```sql
-- ✅ Good : Monthly partition
PARTITION BY toYYYYMM(date)

-- ❌ Less good : Daily partition (too many)
PARTITION BY date
```

## Memory Management

### LIMIT queries

```sql
-- ✅ Good
SELECT * FROM events LIMIT 1000;

-- ❌ Less good
SELECT * FROM events;
```

## Security

### Create users

```sql
CREATE USER analyst IDENTIFIED BY 'password';
GRANT SELECT ON analytics.* TO analyst;
```

### Granular permissions

```sql
GRANT SELECT ON analytics.events TO analyst;
GRANT INSERT ON analytics.temp_table TO analyst;
```

## Maintenance

### Check partitions

```sql
SELECT 
    partition,
    rows,
    formatReadableSize(bytes_on_disk) as size
FROM system.parts
WHERE active
ORDER BY partition;
```

### Clean old data

```sql
ALTER TABLE events DROP PARTITION '202301';
```

### Optimize tables

```sql
OPTIMIZE TABLE events FINAL;
```

---

**Next step :** [Practical Projects](./08-projets/README.md)

