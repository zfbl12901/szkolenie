# 4. Wydajność i optymalizacja

## 🎯 Cele

- Zrozumieć indeksy ClickHouse
- Optymalizować zapytania
- Zarządzać kompresją
- Monitorować wydajność

## Indeksy i projekcje

### Indeks podstawowy (ORDER BY)

```sql
CREATE TABLE events
(
    id UInt64,
    event_date Date,
    user_id UInt32
)
ENGINE = MergeTree()
ORDER BY (event_date, user_id);  -- Indeks podstawowy
```

## Optymalizacja zapytań

### Używać WHERE na kolumnach zindeksowanych

```sql
-- ✅ Dobrze : używa indeksu
SELECT * FROM events 
WHERE event_date = '2024-01-15';

-- ❌ Mniej dobrze : pełne skanowanie
SELECT * FROM events 
WHERE value > 100;
```

### LIMIT do ograniczenia wyników

```sql
SELECT * FROM events 
ORDER BY event_date DESC 
LIMIT 100;
```

### Unikać SELECT *

```sql
-- ✅ Dobrze
SELECT event_date, COUNT(*) 
FROM events 
GROUP BY event_date;

-- ❌ Mniej dobrze
SELECT * FROM events;
```

## Kompresja

### Sprawdzić kompresję

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

## Monitorowanie

### Wolne zapytania

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

**Następny krok :** [Zaawansowane funkcje](./05-fonctions-avancees/README.md)

