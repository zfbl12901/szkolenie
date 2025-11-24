# 5. Advanced Functions

## 🎯 Objectives

- Master date/time functions
- Use mathematical functions
- Manipulate strings
- Use advanced aggregation functions

## Date and Time Functions

### Extraction

```sql
SELECT 
    toYear(event_time) as year,
    toMonth(event_time) as month,
    toDayOfMonth(event_time) as day,
    toHour(event_time) as hour;
```

### Temporal aggregation

```sql
SELECT 
    toStartOfDay(event_time) as day,
    toStartOfHour(event_time) as hour,
    toStartOfWeek(event_time) as week,
    toStartOfMonth(event_time) as month;
```

## Mathematical Functions

### Rounding

```sql
SELECT 
    round(value, 2) as rounded,
    floor(value) as floor_val,
    ceil(value) as ceil_val;
```

### Statistics

```sql
SELECT 
    quantile(0.5)(value) as median,
    quantile(0.95)(value) as p95,
    stddevPop(value) as std_dev;
```

## String Functions

### Manipulation

```sql
SELECT 
    length(name) as name_length,
    upper(name) as upper_name,
    lower(name) as lower_name;
```

### Search

```sql
SELECT 
    position(name, 'test') as pos,
    match(name, 'test.*') as matches,
    replace(name, 'old', 'new') as replaced;
```

## Advanced Aggregation Functions

### Quantiles

```sql
SELECT 
    quantile(0.5)(value) as median,
    quantiles(0.5, 0.9, 0.99)(value) as quantiles;
```

### TopK

```sql
SELECT 
    topK(10)(event_type) as top_10_types,
    topKWeighted(10)(event_type, value) as top_10_weighted;
```

---

**Next step :** [Integration and ETL](./06-integration-etl/README.md)

