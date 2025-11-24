# 2. Analytical SQL with ClickHouse

## 🎯 Objectives

- Master advanced SELECT queries
- Use aggregations and GROUP BY
- Understand analytical functions
- Use Window Functions

## Advanced Aggregations

### GROUP BY with multiple columns

```sql
SELECT 
    event_date,
    event_type,
    COUNT(*) as count,
    SUM(value) as total_value,
    AVG(value) as avg_value
FROM events
GROUP BY event_date, event_type
ORDER BY event_date, event_type;
```

### HAVING to filter aggregations

```sql
SELECT 
    event_type,
    COUNT(*) as count
FROM events
GROUP BY event_type
HAVING count > 10;
```

## Aggregation Functions

### Basic functions

```sql
SELECT 
    COUNT(*) as total_rows,
    COUNT(DISTINCT user_id) as unique_users,
    SUM(value) as total_value,
    AVG(value) as avg_value,
    MIN(value) as min_value,
    MAX(value) as max_value
FROM events;
```

### Statistical functions

```sql
SELECT 
    event_type,
    quantile(0.5)(value) as median,
    quantile(0.95)(value) as p95,
    stddevPop(value) as std_dev
FROM events
GROUP BY event_type;
```

## Window Functions

### ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT 
    user_id,
    event_date,
    value,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY value DESC) as rank
FROM events;
```

### LAG and LEAD

```sql
SELECT 
    event_date,
    value,
    LAG(value) OVER (ORDER BY event_date) as prev_value,
    LEAD(value) OVER (ORDER BY event_date) as next_value
FROM events
ORDER BY event_date;
```

### SUM OVER (cumulative)

```sql
SELECT 
    event_date,
    value,
    SUM(value) OVER (ORDER BY event_date) as cumulative_sum
FROM events
ORDER BY event_date;
```

---

**Next step :** [Data Types and Tables](./03-types-tables/README.md)

