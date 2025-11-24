# 2. Analityczny SQL z ClickHouse

## 🎯 Cele

- Opanować zaawansowane zapytania SELECT
- Używać agregacji i GROUP BY
- Zrozumieć funkcje analityczne
- Używać funkcji okienkowych

## Zaawansowane agregacje

### GROUP BY z wieloma kolumnami

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

### HAVING do filtrowania agregacji

```sql
SELECT 
    event_type,
    COUNT(*) as count
FROM events
GROUP BY event_type
HAVING count > 10;
```

## Funkcje agregacji

### Funkcje podstawowe

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

### Funkcje statystyczne

```sql
SELECT 
    event_type,
    quantile(0.5)(value) as median,
    quantile(0.95)(value) as p95,
    stddevPop(value) as std_dev
FROM events
GROUP BY event_type;
```

## Funkcje okienkowe

### ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT 
    user_id,
    event_date,
    value,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY value DESC) as rank
FROM events;
```

### LAG i LEAD

```sql
SELECT 
    event_date,
    value,
    LAG(value) OVER (ORDER BY event_date) as prev_value,
    LEAD(value) OVER (ORDER BY event_date) as next_value
FROM events
ORDER BY event_date;
```

---

**Następny krok :** [Typy danych i tabele](./03-types-tables/README.md)

