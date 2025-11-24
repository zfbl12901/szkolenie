# 3. Data Types and Tables

## 🎯 Objectives

- Understand ClickHouse data types
- Create optimized tables
- Choose the right table engine
- Configure partitioning

## Data Types

### Integer types

```sql
UInt8, UInt16, UInt32, UInt64  -- Unsigned integers
Int8, Int16, Int32, Int64      -- Signed integers
```

### Decimal types

```sql
Float32, Float64               -- Floating point numbers
Decimal32, Decimal64, Decimal128  -- Precise decimals
```

### String types

```sql
String                        -- Character string
FixedString(N)                -- Fixed length string
```

### Date/Time types

```sql
Date                          -- Date (YYYY-MM-DD)
DateTime                      -- Date and time
DateTime64                    -- Date and time with precision
```

## Creating Tables

### MergeTree table (recommended)

```sql
CREATE TABLE events
(
    id UInt64,
    event_date Date,
    event_time DateTime,
    user_id UInt32,
    event_type String,
    value Float64
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, user_id);
```

### Memory table

```sql
CREATE TABLE temp_data
(
    id UInt64,
    name String
)
ENGINE = Memory;
```

## Partitioning

### By date (recommended)

```sql
PARTITION BY toYYYYMM(event_date)
```

### By hash

```sql
PARTITION BY intHash32(user_id) % 10
```

## Table Engines

- **MergeTree** : For persistent data, large volumes
- **Memory** : For temporary data
- **Log** : For small volumes, logs

---

**Next step :** [Performance and Optimization](./04-performance/README.md)

