# 4. Wskaźniki wydajności

## 🎯 Cele

- Identyfikować kluczowe metryki do monitorowania
- Interpretować wskaźniki Dalibo
- Definiować progi alarmowe
- Rozumieć korelacje między wskaźnikami

## 📋 Spis treści

1. [Metryki systemowe](#metryki-systemowe)
2. [Metryki zapytań](#metryki-zapytań)
3. [Metryki indeksów](#metryki-indeksów)
4. [Metryki I/O](#metryki-io)
5. [Progi alarmowe](#progi-alarmowe)
6. [Pulpit nawigacyjny wskaźników](#pulpit-nawigacyjny-wskaźników)

---

## Metryki systemowe

### Wykorzystanie CPU

```sql
-- Zobaczyć wykorzystanie CPU według procesu
SELECT 
    pid,
    usename,
    application_name,
    state,
    query_start,
    state_change,
    wait_event_type,
    wait_event,
    query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY query_start;
```

**Kluczowe wskaźniki :**
- **Wysokie CPU** : Zapytania w trakcie wykonywania
- **wait_event_type = CPU** : Proces oczekujący na CPU

### Wykorzystanie pamięci

```sql
-- Używana pamięć współdzielona
SELECT 
    setting AS shared_buffers,
    pg_size_pretty(setting::bigint * 8192) AS shared_buffers_size
FROM pg_settings
WHERE name = 'shared_buffers';

-- Używana pamięć robocza
SELECT 
    setting AS work_mem,
    pg_size_pretty(setting::bigint * 1024) AS work_mem_size
FROM pg_settings
WHERE name = 'work_mem';

-- Statystyki pamięci
SELECT 
    name,
    setting,
    unit,
    short_desc
FROM pg_settings
WHERE name IN ('shared_buffers', 'work_mem', 'maintenance_work_mem', 'effective_cache_size')
ORDER BY name;
```

**Kluczowe wskaźniki :**
- **shared_buffers** : Cache współdzielony (zalecane: 25% RAM)
- **work_mem** : Pamięć na operację sortowania/hash
- **effective_cache_size** : Szacunek cache OS (zalecane: 50-75% RAM)

### Aktywne połączenia

```sql
-- Liczba połączeń według stanu
SELECT 
    state,
    COUNT(*) AS count,
    COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () AS percent
FROM pg_stat_activity
GROUP BY state
ORDER BY count DESC;

-- Połączenia według bazy danych
SELECT 
    datname,
    COUNT(*) AS connections,
    COUNT(*) FILTER (WHERE state = 'active') AS active,
    COUNT(*) FILTER (WHERE state = 'idle') AS idle,
    COUNT(*) FILTER (WHERE state = 'idle in transaction') AS idle_in_transaction
FROM pg_stat_activity
WHERE datname IS NOT NULL
GROUP BY datname
ORDER BY connections DESC;

-- Limit połączeń
SELECT 
    setting AS max_connections,
    (SELECT COUNT(*) FROM pg_stat_activity) AS current_connections,
    ROUND(100.0 * (SELECT COUNT(*) FROM pg_stat_activity) / setting::numeric, 2) AS percent_used
FROM pg_settings
WHERE name = 'max_connections';
```

**Kluczowe wskaźniki :**
- **max_connections** : Skonfigurowany limit
- **idle in transaction** : Blokujące połączenia (alarm jeśli > 5%)
- **active** : Zapytania w trakcie

### Blokady (Locks)

```sql
-- Blokady oczekujące
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

**Kluczowe wskaźniki :**
- **Blokady oczekujące** : Aktywne blokady (alarm jeśli > 0)
- **Czas trwania blokad** : Czas oczekiwania (alarm jeśli > 1s)

---

## Metryki zapytań

### Czas wykonania

```sql
-- Średni czas wykonania według zapytania (top 20)
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    ROUND(total_exec_time::numeric, 2) AS total_time_ms,
    ROUND(mean_exec_time::numeric, 2) AS mean_time_ms,
    ROUND(max_exec_time::numeric, 2) AS max_time_ms,
    ROUND(stddev_exec_time::numeric, 2) AS stddev_time_ms,
    ROUND((total_exec_time / SUM(total_exec_time) OVER ()) * 100, 2) AS percent_total_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

**Kluczowe wskaźniki :**
- **mean_exec_time** : Średni czas (alarm jeśli > 1000ms)
- **max_exec_time** : Maksymalny czas (alarm jeśli > 5000ms)
- **stddev_exec_time** : Zmienność (alarm jeśli stddev > mean)

### Częstotliwość wykonania

```sql
-- Najczęstsze zapytania
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    ROUND(mean_exec_time::numeric, 2) AS mean_time_ms,
    ROUND((calls * mean_exec_time)::numeric, 2) AS total_time_ms,
    ROUND((calls::numeric / (SELECT SUM(calls) FROM pg_stat_statements)) * 100, 2) AS percent_calls
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 20;
```

**Kluczowe wskaźniki :**
- **calls** : Liczba wywołań (identyfikować zapytania N+1)
- **percent_calls** : Procent całkowity (alarm jeśli > 50% dla jednego zapytania)

### Skuteczność cache

```sql
-- Współczynnik trafień cache według zapytania
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    shared_blks_hit,
    shared_blks_read,
    ROUND(100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0), 2) AS cache_hit_ratio,
    ROUND((shared_blks_read * 8)::numeric / 1024, 2) AS disk_read_mb
FROM pg_stat_statements
WHERE shared_blks_hit + shared_blks_read > 0
ORDER BY shared_blks_read DESC
LIMIT 20;
```

**Kluczowe wskaźniki :**
- **cache_hit_ratio** : Współczynnik trafień (cel: > 95%)
- **disk_read_mb** : Odczyty z dysku (alarm jeśli > 100MB)

### Zapytania z I/O tymczasowym

```sql
-- Zapytania używające plików tymczasowych
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    temp_blks_read,
    temp_blks_written,
    ROUND((temp_blks_read + temp_blks_written) * 8.0 / 1024, 2) AS temp_mb,
    ROUND(mean_exec_time::numeric, 2) AS mean_time_ms
FROM pg_stat_statements
WHERE temp_blks_read > 0 OR temp_blks_written > 0
ORDER BY (temp_blks_read + temp_blks_written) DESC
LIMIT 20;
```

**Kluczowe wskaźniki :**
- **temp_blks_read/written** : I/O tymczasowe (alarm jeśli > 0)
- **Działanie** : Zwiększyć `work_mem` jeśli obecne

---

## Metryki indeksów

### Wykorzystanie indeksów

```sql
-- Statystyki wykorzystania indeksów
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS index_scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS tuples_fetched,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC, pg_relation_size(indexrelid) DESC
LIMIT 20;
```

**Kluczowe wskaźniki :**
- **idx_scan = 0** : Indeks nieużywany (kandydat do usunięcia)
- **index_size** : Rozmiar indeksu (koszt utrzymania)

### Brakujące indeksy (przez pg_qualstats)

```sql
-- Częste predykaty bez indeksu
SELECT 
    qs.left_schema,
    qs.left_table,
    qs.left_column,
    qs.operator,
    COUNT(*) AS execution_count,
    pg_size_pretty(pg_relation_size((qs.left_schema||'.'||qs.left_table)::regclass)) AS table_size
FROM pg_qualstats qs
WHERE NOT EXISTS (
    SELECT 1
    FROM pg_index i
    JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
    WHERE i.indrelid = (qs.left_schema||'.'||qs.left_table)::regclass
    AND a.attname = qs.left_column
)
GROUP BY qs.left_schema, qs.left_table, qs.left_column, qs.operator
ORDER BY execution_count DESC
LIMIT 20;
```

**Kluczowe wskaźniki :**
- **execution_count** : Częstotliwość użycia (priorytet)
- **table_size** : Rozmiar tabeli (wpływ indeksu)

### Bloat indeksów

```sql
-- Wykrywanie bloatu indeksów
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans,
    CASE 
        WHEN idx_scan = 0 THEN 'UNUSED'
        WHEN idx_scan < 100 THEN 'LOW'
        ELSE 'OK'
    END AS usage_status
FROM pg_stat_user_indexes
WHERE pg_relation_size(indexrelid) > 1048576  -- > 1MB
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Kluczowe wskaźniki :**
- **UNUSED** : Indeks nigdy nieużywany (kandydat do usunięcia)
- **LOW** : Indeks rzadko używany (do oceny)

---

## Metryki I/O

### Statystyki I/O według tabeli

```sql
-- I/O według tabeli
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins AS inserts,
    n_tup_upd AS updates,
    n_tup_del AS deletes,
    n_live_tup AS live_rows,
    n_dead_tup AS dead_rows,
    ROUND(n_dead_tup::numeric / NULLIF(n_live_tup + n_dead_tup, 0) * 100, 2) AS dead_tuple_percent,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY seq_scan DESC
LIMIT 20;
```

**Kluczowe wskaźniki :**
- **seq_scan wysoki** : Wiele skanowań sekwencyjnych (utworzyć indeksy)
- **dead_tuple_percent** : Procent martwych krotek (alarm jeśli > 10%)
- **last_vacuum** : Ostatni vacuum (alarm jeśli > 7 dni)

### Statystyki I/O globalne

```sql
-- Statystyki I/O globalne
SELECT 
    datname,
    blks_read,
    blks_hit,
    ROUND(100.0 * blks_hit / NULLIF(blks_hit + blks_read, 0), 2) AS cache_hit_ratio,
    tup_returned,
    tup_fetched,
    tup_inserted,
    tup_updated,
    tup_deleted
FROM pg_stat_database
WHERE datname NOT IN ('template0', 'template1', 'postgres')
ORDER BY blks_read DESC;
```

**Kluczowe wskaźniki :**
- **cache_hit_ratio** : Globalny współczynnik trafień (cel: > 95%)
- **blks_read** : Odczyty z dysku (alarm jeśli wysoki)

---

## Progi alarmowe

### Tabela zalecanych progów

| Metryka | Próg alarmowy | Próg krytyczny | Działanie |
|---------|---------------|----------------|-----------|
| **Średni czas wykonania** | > 1000ms | > 5000ms | Analizować plan, utworzyć indeks |
| **Współczynnik trafień cache** | < 95% | < 90% | Zwiększyć shared_buffers |
| **Martwe krotki** | > 10% | > 20% | Wykonać VACUUM |
| **I/O tymczasowe** | > 0 | > 100MB | Zwiększyć work_mem |
| **Połączenia idle in transaction** | > 5% | > 10% | Identyfikować i poprawić |
| **Blokady oczekujące** | > 0 | > 10 | Analizować blokady |
| **Indeksy nieużywane** | Rozmiar > 100MB | Rozmiar > 1GB | Ocenić usunięcie |
| **Ostatni VACUUM** | > 7 dni | > 30 dni | Skonfigurować autovacuum |

### Zapytanie monitoringu globalnego

```sql
-- Pulpit nawigacyjny zdrowia PostgreSQL
WITH metrics AS (
    SELECT 
        (SELECT COUNT(*) FROM pg_stat_activity WHERE state = 'active') AS active_queries,
        (SELECT COUNT(*) FROM pg_stat_activity WHERE state = 'idle in transaction') AS idle_in_transaction,
        (SELECT COUNT(*) FROM pg_locks WHERE NOT granted) AS waiting_locks,
        (SELECT AVG(mean_exec_time) FROM pg_stat_statements) AS avg_query_time,
        (SELECT SUM(shared_blks_read) FROM pg_stat_statements) AS total_disk_reads,
        (SELECT SUM(shared_blks_hit) FROM pg_stat_statements) AS total_cache_hits,
        (SELECT COUNT(*) FROM pg_stat_user_indexes WHERE idx_scan = 0) AS unused_indexes,
        (SELECT SUM(n_dead_tup)::numeric / NULLIF(SUM(n_live_tup + n_dead_tup), 0) * 100 
         FROM pg_stat_user_tables) AS dead_tuple_percent
)
SELECT 
    active_queries,
    idle_in_transaction,
    waiting_locks,
    ROUND(avg_query_time::numeric, 2) AS avg_query_time_ms,
    ROUND(100.0 * total_cache_hits / NULLIF(total_cache_hits + total_disk_reads, 0), 2) AS cache_hit_ratio,
    unused_indexes,
    ROUND(dead_tuple_percent::numeric, 2) AS dead_tuple_percent,
    CASE 
        WHEN idle_in_transaction > (SELECT setting::int FROM pg_settings WHERE name = 'max_connections') * 0.1 
        THEN 'ALERT'
        WHEN waiting_locks > 0 THEN 'WARNING'
        WHEN avg_query_time > 1000 THEN 'WARNING'
        WHEN dead_tuple_percent > 10 THEN 'WARNING'
        ELSE 'OK'
    END AS health_status
FROM metrics;
```

---

## Pulpit nawigacyjny wskaźników

### Skonsolidowany widok dla Dalibo

```sql
-- Skonsolidowany widok kluczowych wskaźników
CREATE OR REPLACE VIEW v_performance_dashboard AS
SELECT 
    'QUERIES' AS category,
    COUNT(*) AS metric_count,
    ROUND(AVG(mean_exec_time)::numeric, 2) AS avg_value,
    ROUND(MAX(mean_exec_time)::numeric, 2) AS max_value,
    'ms' AS unit
FROM pg_stat_statements
WHERE mean_exec_time > 100

UNION ALL

SELECT 
    'CACHE_HIT',
    NULL,
    ROUND(100.0 * SUM(shared_blks_hit) / NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2),
    NULL,
    '%'
FROM pg_stat_statements

UNION ALL

SELECT 
    'IDLE_IN_TRANSACTION',
    COUNT(*),
    NULL,
    NULL,
    'count'
FROM pg_stat_activity
WHERE state = 'idle in transaction'

UNION ALL

SELECT 
    'WAITING_LOCKS',
    COUNT(*),
    NULL,
    NULL,
    'count'
FROM pg_locks
WHERE NOT granted

UNION ALL

SELECT 
    'UNUSED_INDEXES',
    COUNT(*),
    NULL,
    NULL,
    'count'
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND pg_relation_size(indexrelid) > 1048576;  -- > 1MB

-- Użycie
SELECT * FROM v_performance_dashboard;
```

### Eksport dla monitoringu zewnętrznego

```sql
-- Eksport JSON dla narzędzi monitoringu
SELECT json_build_object(
    'timestamp', NOW(),
    'metrics', json_build_object(
        'active_queries', (SELECT COUNT(*) FROM pg_stat_activity WHERE state = 'active'),
        'cache_hit_ratio', (
            SELECT ROUND(100.0 * SUM(shared_blks_hit) / NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2)
            FROM pg_stat_statements
        ),
        'avg_query_time_ms', (
            SELECT ROUND(AVG(mean_exec_time)::numeric, 2)
            FROM pg_stat_statements
        ),
        'unused_indexes', (
            SELECT COUNT(*) FROM pg_stat_user_indexes WHERE idx_scan = 0
        )
    )
);
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Monitorować regularnie** : Metryki zmieniają się z czasem
2. **Progi kontekstowe** : Dostosować według środowiska
3. **Korelacje** : Analizować kilka metryk razem
4. **Tendencje** : Monitorować ewolucję w czasie
5. **Działania priorytetowe** : Działać na krytycznych metrykach najpierw

## 🔗 Następny moduł

Przejdź do modułu [5. Techniki optymalizacji](../05-techniques/README.md), aby nauczyć się praktycznych technik optymalizacji.

