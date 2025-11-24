# 3. Dalibo - Narzędzie analizy wydajności

## 🎯 Cele

- Zrozumieć ekosystem Dalibo
- Zainstalować i skonfigurować narzędzia Dalibo
- Używać pg_stat_statements do analizy
- Generować i interpretować raporty wydajności
- Używać automatycznych rekomendacji

## 📋 Spis treści

1. [Prezentacja Dalibo](#prezentacja-dalibo)
2. [Instalacja i konfiguracja](#instalacja-i-konfiguracja)
3. [pg_stat_statements](#pg_stat_statements)
4. [pg_qualstats](#pg_qualstats)
5. [pg_stat_monitor](#pg_stat_monitor)
6. [Raporty i wizualizacje](#raporty-i-wizualizacje)
7. [Rekomendacje automatyczne](#rekomendacje-automatyczne)

---

## Prezentacja Dalibo

### Ekosystem Dalibo

Dalibo oferuje zestaw narzędzi open-source do analizy wydajności PostgreSQL :

- **pg_stat_statements** : Statystyki zapytań SQL
- **pg_qualstats** : Statystyki predykatów (WHERE, JOIN)
- **pg_stat_monitor** : Zaawansowany monitoring z agregacją czasową
- **pg_wait_sampling** : Analiza oczekiwań
- **HypoPG** : Test indeksów hipotetycznych
- **pgBadger** : Analiza logów PostgreSQL
- **pg_activity** : Monitoring w czasie rzeczywistym

### Zalety

✅ **Open-source** : Darmowe i modyfikowalne
✅ **Kompletne** : Obejmuje wszystkie aspekty wydajności
✅ **Zintegrowane** : Działa z natywnym PostgreSQL
✅ **Społeczność** : Aktywne wsparcie i dokumentacja

---

## Instalacja i konfiguracja

### Wymagania wstępne

- PostgreSQL 12+ (niektóre narzędzia wymagają określonych wersji)
- Dostęp superużytkownika do instalacji rozszerzeń
- Kompilator C dla niektórych rozszerzeń

### Instalacja pg_stat_statements

**PostgreSQL 9.2+ :** Włączone domyślnie

```sql
-- Aktywować rozszerzenie
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Sprawdzić instalację
SELECT * FROM pg_extension WHERE extname = 'pg_stat_statements';
```

**Konfiguracja w postgresql.conf :**

```ini
# Załadować rozszerzenie przy starcie
shared_preload_libraries = 'pg_stat_statements'

# Liczba unikalnych zapytań do śledzenia (domyślnie: 10000)
pg_stat_statements.max = 10000

# Maksymalny rozmiar zapytania przechowywanego (domyślnie: 1024)
pg_stat_statements.track = all
pg_stat_statements.track_utility = on
pg_stat_statements.save = on
```

**Uruchomić ponownie PostgreSQL po modyfikacji**

### Instalacja pg_qualstats

**Pobieranie i kompilacja :**

```bash
# Sklonować repozytorium
git clone https://github.com/dalibo/pg_qualstats.git
cd pg_qualstats

# Skompilować i zainstalować
make
sudo make install
```

**Aktywacja :**

```sql
-- Dodać do shared_preload_libraries
-- W postgresql.conf:
-- shared_preload_libraries = 'pg_stat_statements,pg_qualstats'

-- Utworzyć rozszerzenie
CREATE EXTENSION IF NOT EXISTS pg_qualstats;

-- Sprawdzić
SELECT * FROM pg_extension WHERE extname = 'pg_qualstats';
```

### Instalacja pg_stat_monitor

**Dla PostgreSQL 12+ :**

```bash
# Instalacja przez menedżer pakietów (przykład Ubuntu)
sudo apt-get install postgresql-14-pg-stat-monitor

# Lub kompilacja ze źródeł
git clone https://github.com/percona/pg_stat_monitor.git
cd pg_stat_monitor
make
sudo make install
```

**Aktywacja :**

```sql
-- W postgresql.conf:
-- shared_preload_libraries = 'pg_stat_monitor'

CREATE EXTENSION IF NOT EXISTS pg_stat_monitor;
```

---

## pg_stat_statements

### Przegląd

`pg_stat_statements` zbiera statystyki o wszystkich wykonywanych zapytaniach SQL.

### Najbardziej kosztowne zapytania

```sql
-- Top 10 zapytań według całkowitego czasu
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time,
    stddev_exec_time,
    rows,
    100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

### Najczęstsze zapytania

```sql
-- Top 10 zapytań według liczby wywołań
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    (total_exec_time / sum(total_exec_time) OVER ()) * 100 AS percent_total_time
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;
```

### Zapytania z wysokim I/O

```sql
-- Zapytania z wieloma odczytami z dysku
SELECT 
    query,
    calls,
    shared_blks_read,
    shared_blks_hit,
    shared_blks_dirtied,
    shared_blks_written,
    temp_blks_read,
    temp_blks_written
FROM pg_stat_statements
WHERE shared_blks_read > 1000
ORDER BY shared_blks_read DESC
LIMIT 10;
```

### Szczegółowa analiza zapytania

```sql
-- Pełne statystyki dla konkretnego zapytania
SELECT 
    query,
    calls,
    total_exec_time,
    min_exec_time,
    max_exec_time,
    mean_exec_time,
    stddev_exec_time,
    rows,
    shared_blks_hit,
    shared_blks_read,
    shared_blks_dirtied,
    shared_blks_written,
    temp_blks_read,
    temp_blks_written,
    blk_read_time,
    blk_write_time
FROM pg_stat_statements
WHERE query LIKE '%SELECT * FROM users%'
ORDER BY total_exec_time DESC;
```

### Resetowanie statystyk

```sql
-- Resetować wszystkie statystyki
SELECT pg_stat_statements_reset();

-- Resetować dla konkretnej bazy
SELECT pg_stat_statements_reset(userid, dbid, queryid);
```

### Normalizacja zapytań

`pg_stat_statements` normalizuje zapytania, zastępując wartości przez `$1`, `$2`, etc.

**Przykład :**
```sql
-- Oryginalne zapytanie
SELECT * FROM users WHERE id = 123;

-- Znormalizowane w pg_stat_statements
SELECT * FROM users WHERE id = $1;
```

**Zaleta :** Grupuje podobne zapytania z różnymi parametrami.

---

## pg_qualstats

### Przegląd

`pg_qualstats` zbiera statystyki o **predykatach** (warunki WHERE, JOIN) w celu identyfikacji brakujących indeksów.

### Statystyki predykatów

```sql
-- Top najczęściej używanych predykatów
SELECT 
    left_schema,
    left_table,
    left_column,
    operator,
    count(*) AS execution_count,
    n_distinct,
    most_common_vals
FROM pg_qualstats
GROUP BY left_schema, left_table, left_column, operator
ORDER BY execution_count DESC
LIMIT 20;
```

### Identyfikacja brakujących indeksów

```sql
-- Predykaty bez odpowiadającego indeksu
SELECT 
    qs.left_schema,
    qs.left_table,
    qs.left_column,
    qs.operator,
    qs.execution_count,
    pg_size_pretty(pg_relation_size(qs.left_schema||'.'||qs.left_table)) AS table_size
FROM pg_qualstats qs
WHERE NOT EXISTS (
    SELECT 1
    FROM pg_index i
    JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
    WHERE i.indrelid = (qs.left_schema||'.'||qs.left_table)::regclass
    AND a.attname = qs.left_column
)
ORDER BY qs.execution_count DESC
LIMIT 20;
```

### Rekomendacje indeksów

```sql
-- Generować polecenia CREATE INDEX
SELECT 
    'CREATE INDEX idx_' || 
    left_table || '_' || 
    left_column || 
    ' ON ' || left_schema || '.' || left_table || 
    ' (' || left_column || ');' AS create_index_command,
    execution_count,
    n_distinct
FROM (
    SELECT 
        qs.left_schema,
        qs.left_table,
        qs.left_column,
        COUNT(*) AS execution_count,
        COUNT(DISTINCT qs.most_common_vals) AS n_distinct
    FROM pg_qualstats qs
    WHERE NOT EXISTS (
        SELECT 1
        FROM pg_index i
        JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
        WHERE i.indrelid = (qs.left_schema||'.'||qs.left_table)::regclass
        AND a.attname = qs.left_column
    )
    GROUP BY qs.left_schema, qs.left_table, qs.left_column
) AS missing_indexes
ORDER BY execution_count DESC
LIMIT 10;
```

### Resetowanie statystyk

```sql
-- Resetować pg_qualstats
SELECT pg_qualstats_reset();
```

---

## pg_stat_monitor

### Przegląd

`pg_stat_monitor` oferuje zaawansowany monitoring z agregacją czasową i analizą bucketów.

### Konfiguracja

```sql
-- Zobaczyć konfigurację
SELECT * FROM pg_stat_monitor_settings;

-- Zmodyfikować konfigurację
ALTER SYSTEM SET pg_stat_monitor.pgsm_max_buckets = 10;
SELECT pg_reload_conf();
```

### Zapytania według bucketa (okres)

```sql
-- Zapytania pogrupowane według okresu
SELECT 
    bucket,
    bucket_start_time,
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time
FROM pg_stat_monitor
ORDER BY bucket DESC, total_exec_time DESC
LIMIT 20;
```

### Analiza błędów

```sql
-- Zapytania z błędami
SELECT 
    query,
    calls,
    errors,
    error_count,
    error_code
FROM pg_stat_monitor
WHERE errors > 0
ORDER BY errors DESC;
```

### Analiza planów

```sql
-- Najczęściej używane plany wykonania
SELECT 
    query,
    planid,
    calls,
    mean_exec_time,
    plans
FROM pg_stat_monitor
WHERE plans IS NOT NULL
ORDER BY calls DESC
LIMIT 10;
```

---

## Raporty i wizualizacje

### pgBadger - Analiza logów

**Instalacja :**

```bash
# Ubuntu/Debian
sudo apt-get install pgbadger

# Lub przez Perl CPAN
cpanm pgbadger
```

**Generowanie raportu :**

```bash
# Generować raport HTML
pgbadger /var/log/postgresql/postgresql-*.log -o report.html

# Z opcjami zaawansowanymi
pgbadger \
  --prefix '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h' \
  --outdir /var/www/pgbadger \
  /var/log/postgresql/postgresql-*.log
```

**Konfiguracja PostgreSQL dla pgBadger :**

```ini
# W postgresql.conf
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 0
log_autovacuum_min_duration = 0
log_error_verbosity = default
log_min_duration_statement = 1000  # Log zapytania > 1s
```

### pg_activity - Monitoring czasu rzeczywistego

**Instalacja :**

```bash
pip install pg_activity
```

**Użycie :**

```bash
# Proste połączenie
pg_activity -U postgres -d mydb

# Z opcjami
pg_activity -U postgres -d mydb --refresh 2 --no-database-size
```

### Wizualizacja z Metabase/Grafana

**Integracja z Grafana :**

1. Zainstalować plugin PostgreSQL
2. Utworzyć dashboardy z widokami systemowymi
3. Monitorować metryki w czasie rzeczywistym

**Przydatne zapytania dla Grafana :**

```sql
-- Średni czas wykonania na minutę
SELECT 
    date_trunc('minute', now()) AS time,
    AVG(mean_exec_time) AS avg_exec_time
FROM pg_stat_statements
GROUP BY time;
```

---

## Rekomendacje automatyczne

### Podstawowy skrypt rekomendacji

```sql
-- Wolne zapytania bez odpowiedniego indeksu
WITH slow_queries AS (
    SELECT 
        query,
        calls,
        mean_exec_time,
        total_exec_time
    FROM pg_stat_statements
    WHERE mean_exec_time > 100  -- > 100ms
    ORDER BY total_exec_time DESC
    LIMIT 10
),
missing_indexes AS (
    SELECT 
        qs.left_schema,
        qs.left_table,
        qs.left_column,
        qs.operator,
        COUNT(*) AS execution_count
    FROM pg_qualstats qs
    WHERE NOT EXISTS (
        SELECT 1
        FROM pg_index i
        JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
        WHERE i.indrelid = (qs.left_schema||'.'||qs.left_table)::regclass
        AND a.attname = qs.left_column
    )
    GROUP BY qs.left_schema, qs.left_table, qs.left_column, qs.operator
)
SELECT 
    'MISSING INDEX' AS recommendation_type,
    'CREATE INDEX idx_' || left_table || '_' || left_column || 
    ' ON ' || left_schema || '.' || left_table || 
    ' (' || left_column || ');' AS recommendation,
    execution_count AS priority_score
FROM missing_indexes
ORDER BY execution_count DESC
LIMIT 10;
```

### Używać HypoPG do testowania indeksów

**Instalacja :**

```sql
CREATE EXTENSION IF NOT EXISTS hypopg;
```

**Testować indeks hipotetyczny :**

```sql
-- Utworzyć indeks hipotetyczny
SELECT * FROM hypopg_create_index('CREATE INDEX ON users(email)');

-- Zobaczyć indeksy hipotetyczne
SELECT * FROM hypopg_list_indexes();

-- Testować plan z indeksem hipotetycznym
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Usunąć indeks hipotetyczny
SELECT hypopg_drop_index(oid) FROM hypopg_list_indexes();
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **pg_stat_statements** : Niezbędne do identyfikacji wolnych zapytań
2. **pg_qualstats** : Identyfikuje brakujące indeksy automatycznie
3. **pg_stat_monitor** : Zaawansowany monitoring z agregacją czasową
4. **pgBadger** : Kompletna analiza logów PostgreSQL
5. **HypoPG** : Testuje indeksy przed ich utworzeniem

## 🔗 Następny moduł

Przejdź do modułu [4. Wskaźniki wydajności](../04-indicateurs/README.md), aby nauczyć się interpretować kluczowe wskaźniki wydajności.

