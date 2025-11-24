# 6. Przypadki praktyczne optymalizacji

## 🎯 Cele

- Stosować techniki optymalizacji na rzeczywistych przypadkach
- Analizować problemy wydajnościowe
- Mierzyć wpływ optymalizacji
- Używać Dalibo do identyfikacji i rozwiązywania problemów

## 📋 Spis treści

1. [Przypadek 1 : Wolne zapytanie z skanowaniem sekwencyjnym](#przypadek-1--wolne-zapytanie-z-skanowaniem-sekwencyjnym)
2. [Przypadek 2 : Wolne złączenie na dużej tabeli](#przypadek-2--wolne-złączenie-na-dużej-tabeli)
3. [Przypadek 3 : Wolna agregacja](#przypadek-3--wolna-agregacja)
4. [Przypadek 4 : Podzapytanie skorelowane](#przypadek-4--podzapytanie-skorelowane)
5. [Przypadek 5 : Problem współczynnika trafień cache](#przypadek-5--problem-współczynnika-trafień-cache)
6. [Przypadek 6 : Indeksy nieużywane](#przypadek-6--indeksy-nieużywane)

---

## Przypadek 1 : Wolne zapytanie z skanowaniem sekwencyjnym

### Problem początkowy

**Zapytanie :**
```sql
SELECT * FROM users WHERE email = 'user@example.com';
```

**Plan wykonania :**
```
Seq Scan on users  (cost=0.00..25000.00 rows=1 width=64)
  (actual time=0.123..1500.456 rows=1 loops=1)
  Filter: (email = 'user@example.com'::text)
  Rows Removed by Filter: 999999
Planning Time: 0.234 ms
Execution Time: 1500.678 ms
```

**Zidentyfikowane problemy :**
- 🔴 Skanowanie sekwencyjne na 1 milionie wierszy
- 🔴 Czas wykonania : 1.5 sekundy
- 🔴 999999 wierszy przefiltrowanych

### Analiza z Dalibo

```sql
-- Sprawdzić w pg_stat_statements
SELECT 
    query,
    calls,
    mean_exec_time,
    shared_blks_read,
    shared_blks_hit
FROM pg_stat_statements
WHERE query LIKE '%users WHERE email%'
ORDER BY mean_exec_time DESC;

-- Sprawdzić z pg_qualstats
SELECT 
    left_table,
    left_column,
    operator,
    execution_count
FROM pg_qualstats
WHERE left_table = 'users' AND left_column = 'email';
```

### Rozwiązanie

```sql
-- Utworzyć indeks na email
CREATE INDEX idx_users_email ON users(email);

-- Sprawdzić nowy plan
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'user@example.com';
```

**Zoptymalizowany plan :**
```
Index Scan using idx_users_email on users
  (cost=0.42..8.44 rows=1 width=64)
  (actual time=0.123..0.125 rows=1 loops=1)
  Index Cond: (email = 'user@example.com'::text)
Planning Time: 0.234 ms
Execution Time: 0.125 ms
```

### Wyniki

| Metryka | Przed | Po | Poprawa |
|---------|-------|-----|---------|
| Czas wykonania | 1500ms | 0.125ms | **99.99%** |
| Typ skanowania | Seq Scan | Index Scan | ✅ |
| Wiersze skanowane | 1,000,000 | 1 | ✅ |

---

## Przypadek 2 : Wolne złączenie na dużej tabeli

### Problem początkowy

**Zapytanie :**
```sql
SELECT 
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email;
```

**Plan wykonania :**
```
Hash Join
  (cost=125000.00..250000.00 rows=50000 width=64)
  (actual time=5000.123..15000.456 rows=45000 loops=1)
  Hash Cond: (o.user_id = u.id)
  -> Seq Scan on orders o
      (cost=0.00..100000.00 rows=1000000 width=16)
      (actual time=0.123..5000.456 rows=1000000 loops=1)
  -> Hash
      (cost=25000.00..25000.00 rows=100000 width=48)
      (actual time=2000.123..2000.123 rows=100000 loops=1)
      Buckets: 131072  Batches: 8  Memory Usage: 5120kB
      -> Seq Scan on users u
          (cost=0.00..25000.00 rows=100000 width=48)
          (actual time=0.089..1000.567 rows=100000 loops=1)
          Filter: (created_at > '2024-01-01'::date)
Planning Time: 50.234 ms
Execution Time: 15000.678 ms
```

**Zidentyfikowane problemy :**
- 🔴 Hash Join z 8 batchami (sortowanie na dysku)
- 🔴 Skanowanie sekwencyjne na orders (1 milion wierszy)
- 🔴 Czas wykonania : 15 sekund

### Analiza z Dalibo

```sql
-- Identyfikować brakujące indeksy
SELECT 
    qs.left_table,
    qs.left_column,
    qs.operator,
    COUNT(*) AS execution_count
FROM pg_qualstats qs
WHERE qs.left_table IN ('users', 'orders')
GROUP BY qs.left_table, qs.left_column, qs.operator
ORDER BY execution_count DESC;
```

### Rozwiązanie

```sql
-- Utworzyć indeksy na kluczach złączenia i filtrach
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Indeks złożony dla pełnego zapytania
CREATE INDEX idx_orders_user_id_amount ON orders(user_id, amount);

-- Zwiększyć work_mem do uniknięcia batchów
SET work_mem = '256MB';

-- Sprawdzić nowy plan
EXPLAIN ANALYZE
SELECT 
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email;
```

**Zoptymalizowany plan :**
```
Hash Join
  (cost=5000.00..15000.00 rows=50000 width=64)
  (actual time=200.123..800.456 rows=45000 loops=1)
  Hash Cond: (o.user_id = u.id)
  -> Index Scan using idx_orders_user_id on orders o
      (cost=0.42..8000.00 rows=500000 width=16)
      (actual time=0.123..300.456 rows=500000 loops=1)
  -> Hash
      (cost=2500.00..2500.00 rows=100000 width=48)
      (actual time=100.123..100.123 rows=100000 loops=1)
      Buckets: 131072  Batches: 1  Memory Usage: 5120kB
      -> Index Scan using idx_users_created_at on users u
          (cost=0.42..2500.00 rows=100000 width=48)
          (actual time=0.089..50.567 rows=100000 loops=1)
          Index Cond: (created_at > '2024-01-01'::date)
Planning Time: 5.234 ms
Execution Time: 800.678 ms
```

### Wyniki

| Metryka | Przed | Po | Poprawa |
|---------|-------|-----|---------|
| Czas wykonania | 15000ms | 800ms | **94.7%** |
| Batches Hash Join | 8 | 1 | ✅ |
| Typ skanowania orders | Seq Scan | Index Scan | ✅ |
| Wiersze skanowane | 1,000,000 | 500,000 | ✅ |

---

## Przypadek 3 : Wolna agregacja

### Problem początkowy

**Zapytanie :**
```sql
SELECT 
    status,
    COUNT(*) AS count,
    AVG(amount) AS avg_amount,
    SUM(amount) AS total_amount
FROM orders
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY status
ORDER BY count DESC;
```

**Plan wykonania :**
```
Sort
  (cost=50000.00..50000.00 rows=5 width=32)
  (actual time=10000.123..10000.234 rows=5 loops=1)
  Sort Key: (count(*)) DESC
  Sort Method: quicksort  Memory: 25kB
  -> HashAggregate
      (cost=45000.00..45000.00 rows=5 width=32)
      (actual time=8000.123..8000.456 rows=5 loops=1)
      Group Key: status
      Batches: 1  Memory Usage: 24kB
      -> Seq Scan on orders
          (cost=0.00..40000.00 rows=2000000 width=16)
          (actual time=0.123..5000.456 rows=2000000 loops=1)
          Filter: ((created_at >= '2024-01-01'::date) 
                   AND (created_at <= '2024-12-31'::date))
          Rows Removed by Filter: 0
Planning Time: 10.234 ms
Execution Time: 10000.678 ms
```

**Zidentyfikowane problemy :**
- 🔴 Skanowanie sekwencyjne na 2 milionach wierszy
- 🔴 Czas wykonania : 10 sekund
- 🔴 Brak indeksu na created_at

### Rozwiązanie

```sql
-- Utworzyć indeks na created_at i status
CREATE INDEX idx_orders_created_at_status ON orders(created_at, status);

-- Alternatywa : Indeks częściowy jeśli niektóre status są rzadkie
CREATE INDEX idx_orders_created_at_status_partial 
ON orders(created_at, status) 
WHERE status IN ('pending', 'processing');

-- Sprawdzić nowy plan
EXPLAIN ANALYZE
SELECT 
    status,
    COUNT(*) AS count,
    AVG(amount) AS avg_amount,
    SUM(amount) AS total_amount
FROM orders
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY status
ORDER BY count DESC;
```

**Zoptymalizowany plan :**
```
Sort
  (cost=5000.00..5000.00 rows=5 width=32)
  (actual time=500.123..500.234 rows=5 loops=1)
  Sort Key: (count(*)) DESC
  Sort Method: quicksort  Memory: 25kB
  -> HashAggregate
      (cost=4500.00..4500.00 rows=5 width=32)
      (actual time=400.123..400.456 rows=5 loops=1)
      Group Key: status
      Batches: 1  Memory Usage: 24kB
      -> Index Scan using idx_orders_created_at_status on orders
          (cost=0.42..4000.00 rows=2000000 width=16)
          (actual time=0.123..200.456 rows=2000000 loops=1)
          Index Cond: ((created_at >= '2024-01-01'::date) 
                       AND (created_at <= '2024-12-31'::date))
Planning Time: 5.234 ms
Execution Time: 500.678 ms
```

### Wyniki

| Metryka | Przed | Po | Poprawa |
|---------|-------|-----|---------|
| Czas wykonania | 10000ms | 500ms | **95%** |
| Typ skanowania | Seq Scan | Index Scan | ✅ |
| Wiersze skanowane | 2,000,000 | 2,000,000 | (ta sama liczba, ale indeks) |

---

## Przypadek 4 : Podzapytanie skorelowane

### Problem początkowy

**Zapytanie :**
```sql
SELECT 
    u.id,
    u.name,
    u.email,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count,
    (SELECT MAX(created_at) FROM orders o WHERE o.user_id = u.id) AS last_order_date,
    (SELECT SUM(amount) FROM orders o WHERE o.user_id = u.id) AS total_spent
FROM users u
WHERE u.active = true;
```

**Plan wykonania :**
```
Seq Scan on users u
  (cost=0.00..250000.00 rows=100000 width=64)
  (actual time=0.123..50000.456 rows=100000 loops=1)
  Filter: (active = true)
  SubPlan 1
    -> Aggregate
        (cost=2.50..2.50 rows=1 width=8)
        (actual time=0.100..0.100 rows=1 loops=100000)
        -> Seq Scan on orders o
            (cost=0.00..2.25 rows=10 width=0)
            (actual time=0.050..0.050 rows=5 loops=100000)
            Filter: (user_id = u.id)
  SubPlan 2
    -> Result
        (cost=2.50..2.50 rows=1 width=8)
        (actual time=0.100..0.100 rows=1 loops=100000)
        -> Aggregate
            (cost=2.50..2.50 rows=1 width=8)
            (actual time=0.100..0.100 rows=1 loops=100000)
            -> Seq Scan on orders o
                (cost=0.00..2.25 rows=10 width=0)
                (actual time=0.050..0.050 rows=5 loops=100000)
                Filter: (user_id = u.id)
  SubPlan 3
    -> Aggregate
        (cost=2.50..2.50 rows=1 width=8)
        (actual time=0.100..0.100 rows=1 loops=100000)
        -> Seq Scan on orders o
            (cost=0.00..2.25 rows=10 width=0)
            (actual time=0.050..0.050 rows=5 loops=100000)
            Filter: (user_id = u.id)
Planning Time: 5.234 ms
Execution Time: 50000.678 ms
```

**Zidentyfikowane problemy :**
- 🔴 3 podzapytania skorelowane wykonywane 100,000 razy każde
- 🔴 300,000 skanowań sekwencyjnych na orders
- 🔴 Czas wykonania : 50 sekund

### Rozwiązanie

```sql
-- Zastąpić przez JOIN z agregacją
SELECT 
    u.id,
    u.name,
    u.email,
    COALESCE(o.order_count, 0) AS order_count,
    o.last_order_date,
    COALESCE(o.total_spent, 0) AS total_spent
FROM users u
LEFT JOIN (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        MAX(created_at) AS last_order_date,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id
WHERE u.active = true;

-- Utworzyć indeks do przyspieszenia złączenia
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

**Zoptymalizowany plan :**
```
Hash Right Join
  (cost=5000.00..15000.00 rows=100000 width=64)
  (actual time=200.123..800.456 rows=100000 loops=1)
  Hash Cond: (o.user_id = u.id)
  -> HashAggregate
      (cost=4000.00..4500.00 rows=50000 width=24)
      (actual time=150.123..200.456 rows=50000 loops=1)
      Group Key: orders.user_id
      Batches: 1  Memory Usage: 5120kB
      -> Index Scan using idx_orders_user_id on orders
          (cost=0.42..3000.00 rows=500000 width=16)
          (actual time=0.123..100.456 rows=500000 loops=1)
  -> Hash
      (cost=2000.00..2000.00 rows=100000 width=48)
      (actual time=50.123..50.123 rows=100000 loops=1)
      Buckets: 131072  Batches: 1  Memory Usage: 5120kB
      -> Seq Scan on users u
          (cost=0.00..2000.00 rows=100000 width=48)
          (actual time=0.089..25.567 rows=100000 loops=1)
          Filter: (active = true)
Planning Time: 5.234 ms
Execution Time: 800.678 ms
```

### Wyniki

| Metryka | Przed | Po | Poprawa |
|---------|-------|-----|---------|
| Czas wykonania | 50000ms | 800ms | **98.4%** |
| Skanowania na orders | 300,000 | 1 | ✅ |
| Typ operacji | Podzapytania | Hash Join | ✅ |

---

## Przypadek 5 : Problem współczynnika trafień cache

### Problem początkowy

**Metryki :**
```sql
-- Globalny współczynnik trafień cache
SELECT 
    ROUND(100.0 * SUM(shared_blks_hit) / 
          NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2) AS cache_hit_ratio
FROM pg_stat_statements;
-- Wynik: 75% (cel: > 95%)
```

**Zapytania z wieloma odczytami z dysku :**
```sql
SELECT 
    LEFT(query, 100) AS query_preview,
    shared_blks_read,
    shared_blks_hit,
    ROUND(100.0 * shared_blks_hit / 
          NULLIF(shared_blks_hit + shared_blks_read, 0), 2) AS cache_hit_ratio,
    ROUND((shared_blks_read * 8)::numeric / 1024, 2) AS disk_read_mb
FROM pg_stat_statements
WHERE shared_blks_read > 1000
ORDER BY shared_blks_read DESC
LIMIT 10;
```

### Rozwiązanie

```sql
-- 1. Zwiększyć shared_buffers (w postgresql.conf)
-- shared_buffers = 4GB  (25% RAM dla serwera dedykowanego)

-- 2. Wstępnie załadować często używane tabele
-- Utworzyć funkcję wstępnego ładowania
CREATE OR REPLACE FUNCTION pg_prewarm_table(table_name TEXT)
RETURNS void AS $$
BEGIN
    EXECUTE format('SELECT * FROM %I LIMIT 1', table_name);
END;
$$ LANGUAGE plpgsql;

-- Wstępnie załadować ważne tabele
SELECT pg_prewarm_table('users');
SELECT pg_prewarm_table('orders');
SELECT pg_prewarm_table('products');

-- 3. Używać rozszerzenia pg_prewarm
CREATE EXTENSION IF NOT EXISTS pg_prewarm;

-- Wstępnie załadować pełną tabelę
SELECT pg_prewarm('users');
SELECT pg_prewarm('orders');
```

**Po optymalizacji :**
```sql
-- Sprawdzić poprawę
SELECT 
    ROUND(100.0 * SUM(shared_blks_hit) / 
          NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2) AS cache_hit_ratio
FROM pg_stat_statements;
-- Wynik: 98% ✅
```

### Wyniki

| Metryka | Przed | Po | Poprawa |
|---------|-------|-----|---------|
| Współczynnik trafień cache | 75% | 98% | **+23%** |
| Odczyty z dysku | Wysokie | Niskie | ✅ |
| Czas odpowiedzi | Zmienny | Stabilny | ✅ |

---

## Przypadek 6 : Indeksy nieużywane

### Problem początkowy

**Identyfikacja nieużywanych indeksów :**
```sql
-- Indeksy nigdy nieużywane
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans,
    pg_relation_size(indexrelid) AS size_bytes
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND pg_relation_size(indexrelid) > 1048576  -- > 1MB
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Wynik :**
```
 schemaname | tablename |      indexname       | index_size | index_scans | size_bytes
------------+-----------+----------------------+------------+-------------+------------
 public     | orders    | idx_orders_old_field | 250 MB     |           0 |  262144000
 public     | users     | idx_users_old_email  | 150 MB     |           0 |  157286400
```

**Wpływ :**
- 🔴 400 MB przestrzeni dyskowej zmarnowanej
- 🔴 Spowolnienie INSERT/UPDATE
- 🔴 Niepotrzebna konserwacja

### Rozwiązanie

```sql
-- 1. Sprawdzić z HypoPG czy indeks jest naprawdę niepotrzebny
CREATE EXTENSION IF NOT EXISTS hypopg;

-- 2. Analizować zapytania, które mogłyby użyć indeksu
SELECT 
    query,
    calls,
    mean_exec_time
FROM pg_stat_statements
WHERE query LIKE '%old_field%' OR query LIKE '%old_email%';

-- 3. Jeśli naprawdę niepotrzebny, usunąć indeks
DROP INDEX idx_orders_old_field;
DROP INDEX idx_users_old_email;

-- 4. Sprawdzić zwolnioną przestrzeń
SELECT 
    pg_size_pretty(pg_database_size(current_database())) AS database_size;
```

### Wyniki

| Metryka | Przed | Po | Poprawa |
|---------|-------|-----|---------|
| Przestrzeń indeksów | 400 MB | 0 MB | **-400 MB** |
| Czas INSERT | +10% | Normalny | ✅ |
| Czas UPDATE | +15% | Normalny | ✅ |

---

## 📊 Kluczowe punkty do zapamiętania

1. **Zawsze analizować z EXPLAIN ANALYZE** przed optymalizacją
2. **Używać Dalibo** do automatycznej identyfikacji problemów
3. **Mierzyć wpływ** przed i po każdej optymalizacji
4. **Odpowiednie indeksy** : Najczęstsze rozwiązanie
5. **Unikać podzapytań skorelowanych** : Używać JOIN
6. **Monitorować regularnie** : Problemy ewoluują

## 🔗 Następny moduł

Przejdź do modułu [7. Ćwiczenia](../07-exercices/README.md), aby ćwiczyć z ćwiczeniami prowadzonymi.

