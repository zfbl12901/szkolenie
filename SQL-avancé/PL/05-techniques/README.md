# 5. Techniki optymalizacji

## 🎯 Cele

- Opanować techniki optymalizacji złączeń
- Optymalizować agregacje i podzapytania
- Używać partycjonowania skutecznie
- Wykorzystać równoległość PostgreSQL

## 📋 Spis treści

1. [Optymalizacja złączeń](#optymalizacja-złączeń)
2. [Optymalizacja agregacji](#optymalizacja-agregacji)
3. [Optymalizacja podzapytań](#optymalizacja-podzapytań)
4. [Partycjonowanie](#partycjonowanie)
5. [Równoległość](#równoległość)
6. [Optymalizacja typów danych](#optymalizacja-typów-danych)

---

## Optymalizacja złączeń

### Wybór typu złączenia

PostgreSQL wybiera automatycznie, ale możesz wpływać :

**Nested Loop :**
- ✅ Mała tabela zewnętrzna (< 1000 wierszy)
- ✅ Indeks na kluczu złączenia
- ❌ Duża tabela zewnętrzna

**Hash Join :**
- ✅ Tabele podobnej wielkości
- ✅ Złączenie równościowe
- ✅ Wystarczająco dużo `work_mem`
- ❌ Indeks nie jest konieczny

**Merge Join :**
- ✅ Dane już posortowane
- ✅ Złączenia na posortowanych kluczach
- ❌ Wymaga sortowania jeśli nie posortowane

### Optymalizować z indeksami

```sql
-- Przed : Wolne złączenie
EXPLAIN ANALYZE
SELECT u.*, o.*
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';

-- Utworzyć indeksy na kluczach złączenia
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Po : Zoptymalizowane złączenie
EXPLAIN ANALYZE
SELECT u.*, o.*
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';
```

### Zmniejszyć rozmiar tabel złączenia

```sql
-- Przed : Złączenie na wszystkich wierszach
SELECT u.*, o.*
FROM users u
JOIN orders o ON u.id = o.user_id;

-- Po : Filtrować przed złączeniem
SELECT u.*, o.*
FROM (
    SELECT * FROM users WHERE active = true
) u
JOIN (
    SELECT * FROM orders WHERE status = 'completed'
) o ON u.id = o.user_id;
```

### Kolejność złączeń

Planista wybiera kolejność, ale możesz wpływać :

```sql
-- Używać CTE do wymuszenia kolejności
WITH filtered_users AS (
    SELECT * FROM users WHERE active = true
),
filtered_orders AS (
    SELECT * FROM orders WHERE status = 'completed'
)
SELECT u.*, o.*
FROM filtered_users u
JOIN filtered_orders o ON u.id = o.user_id;
```

### Wiele złączeń

```sql
-- Optymalizować kolejność złączeń
-- PostgreSQL zazwyczaj wybiera dobrą kolejność, ale sprawdź z EXPLAIN

-- Złe : Złączenie na dużej tabeli najpierw
SELECT *
FROM large_table l
JOIN small_table1 s1 ON l.id = s1.large_id
JOIN small_table2 s2 ON l.id = s2.large_id;

-- Lepsze : Filtrować najpierw
SELECT *
FROM large_table l
JOIN (
    SELECT large_id FROM small_table1 WHERE condition = true
) s1 ON l.id = s1.large_id
JOIN (
    SELECT large_id FROM small_table2 WHERE condition = true
) s2 ON l.id = s2.large_id;
```

---

## Optymalizacja agregacji

### Zoptymalizowany GROUP BY

```sql
-- Przed : Agregacja na wszystkich wierszach
SELECT status, COUNT(*), AVG(amount)
FROM orders
GROUP BY status;

-- Po : Filtrować przed agregacją
SELECT status, COUNT(*), AVG(amount)
FROM orders
WHERE created_at > '2024-01-01'
GROUP BY status;

-- Indeks do przyspieszenia
CREATE INDEX idx_orders_status_created ON orders(status, created_at);
```

### Agregacje z HAVING

```sql
-- Filtrować z WHERE przed GROUP BY (bardziej skuteczne)
-- Złe
SELECT status, COUNT(*)
FROM orders
GROUP BY status
HAVING COUNT(*) > 100;

-- Lepsze : Używać podzapytania
SELECT status, cnt
FROM (
    SELECT status, COUNT(*) AS cnt
    FROM orders
    GROUP BY status
) sub
WHERE cnt > 100;
```

### Zoptymalizowany DISTINCT

```sql
-- DISTINCT może być kosztowne
SELECT DISTINCT user_id FROM orders;

-- Czasami GROUP BY jest szybsze
SELECT user_id FROM orders GROUP BY user_id;

-- Z indeksem, oba mogą być szybkie
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

### Agregacje z oknami

```sql
-- Używać funkcji okienkowych do unikania podzapytań
-- Przed : Podzapytanie skorelowane
SELECT 
    o.*,
    (SELECT AVG(amount) FROM orders o2 WHERE o2.user_id = o.user_id) AS avg_user_amount
FROM orders o;

-- Po : Funkcja okienkowa
SELECT 
    o.*,
    AVG(amount) OVER (PARTITION BY user_id) AS avg_user_amount
FROM orders o;
```

---

## Optymalizacja podzapytań

### Podzapytania skorelowane → JOIN

```sql
-- Przed : Podzapytanie skorelowane (wolne)
SELECT 
    u.*,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;

-- Po : JOIN z agregacją (szybsze)
SELECT 
    u.*,
    COALESCE(o.order_count, 0) AS order_count
FROM users u
LEFT JOIN (
    SELECT user_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id;
```

### EXISTS vs IN vs JOIN

```sql
-- EXISTS : Zazwyczaj najszybsze
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);

-- IN : Może być wolne jeśli lista jest duża
SELECT *
FROM users u
WHERE u.id IN (
    SELECT user_id FROM orders
);

-- JOIN : Dobry kompromis
SELECT DISTINCT u.*
FROM users u
JOIN orders o ON u.id = o.user_id;
```

### Podzapytania w SELECT

```sql
-- Unikać podzapytań w SELECT jeśli możliwe
-- Przed : Podzapytanie wykonywane dla każdego wiersza
SELECT 
    u.*,
    (SELECT MAX(created_at) FROM orders WHERE user_id = u.id) AS last_order_date
FROM users u;

-- Po : JOIN z agregacją
SELECT 
    u.*,
    o.last_order_date
FROM users u
LEFT JOIN (
    SELECT user_id, MAX(created_at) AS last_order_date
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id;
```

### CTE (Common Table Expressions)

```sql
-- CTE do poprawy czytelności i czasami wydajności
WITH active_users AS (
    SELECT * FROM users WHERE active = true
),
recent_orders AS (
    SELECT * FROM orders WHERE created_at > '2024-01-01'
)
SELECT 
    u.*,
    COUNT(o.id) AS order_count
FROM active_users u
LEFT JOIN recent_orders o ON u.id = o.user_id
GROUP BY u.id;
```

---

## Partycjonowanie

### Partycjonowanie według zakresu (Range)

```sql
-- Utworzyć tabelę partycjonowaną
CREATE TABLE orders (
    id SERIAL,
    user_id INTEGER,
    amount DECIMAL,
    created_at DATE
) PARTITION BY RANGE (created_at);

-- Utworzyć partycje
CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

CREATE TABLE orders_2024_q3 PARTITION OF orders
    FOR VALUES FROM ('2024-07-01') TO ('2024-10-01');

CREATE TABLE orders_2024_q4 PARTITION OF orders
    FOR VALUES FROM ('2024-10-01') TO ('2025-01-01');
```

**Zalety :**
- ✅ Partition pruning (tylko odpowiednie partycje są skanowane)
- ✅ Konserwacja według partycji (VACUUM, ANALYZE)
- ✅ Szybkie usuwanie całych partycji

### Partycjonowanie według listy (List)

```sql
-- Partycjonowanie według regionu
CREATE TABLE users (
    id SERIAL,
    name TEXT,
    region TEXT
) PARTITION BY LIST (region);

CREATE TABLE users_europe PARTITION OF users
    FOR VALUES IN ('FR', 'DE', 'UK', 'IT');

CREATE TABLE users_america PARTITION OF users
    FOR VALUES IN ('US', 'CA', 'MX');

CREATE TABLE users_asia PARTITION OF users
    FOR VALUES IN ('JP', 'CN', 'IN');
```

### Partycjonowanie według hash

```sql
-- Partycjonowanie według hash (dla równomiernej dystrybucji)
CREATE TABLE events (
    id SERIAL,
    user_id INTEGER,
    event_type TEXT,
    created_at TIMESTAMP
) PARTITION BY HASH (user_id);

CREATE TABLE events_0 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE events_1 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE events_2 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE events_3 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

### Indeksy na tabelach partycjonowanych

```sql
-- Utworzyć indeks na tabeli partycjonowanej (utworzony na wszystkich partycjach)
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Lub utworzyć indeksy specyficzne według partycji
CREATE INDEX idx_orders_2024_q1_user_id ON orders_2024_q1(user_id);
```

### Konserwacja partycji

```sql
-- Sprawdzić partycje
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
  AND tablename LIKE 'orders_%'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Usunąć partycję (bardzo szybko)
DROP TABLE orders_2024_q1;  -- Usuwa partycję i jej dane

-- Odłączyć partycję (zachować dane)
ALTER TABLE orders DETACH PARTITION orders_2024_q1;
```

---

## Równoległość

### Konfiguracja równoległości

```sql
-- Sprawdzić konfigurację
SHOW max_parallel_workers_per_gather;
SHOW max_parallel_workers;
SHOW max_worker_processes;

-- Zmodyfikować (w postgresql.conf)
-- max_parallel_workers_per_gather = 4
-- max_parallel_workers = 8
-- max_worker_processes = 8
```

### Kiedy równoległość jest używana

PostgreSQL używa równoległości dla :
- ✅ Skanowań sekwencyjnych dużych tabel
- ✅ Złączeń na dużych tabelach
- ✅ Agregacji na dużych tabelach
- ❌ Małych tabel (< 8MB domyślnie)
- ❌ Zapytań z blokadami

### Wymusić równoległość

```sql
-- Zwiększyć min_parallel_table_scan_size do wymuszenia równoległości
SET min_parallel_table_scan_size = 0;  -- Zawsze rozważać równoległość

-- Zobaczyć plan z równoległością
EXPLAIN ANALYZE
SELECT COUNT(*) FROM large_table WHERE condition = 'value';
```

**Typowy wynik :**
```
Finalize Aggregate
  -> Gather
      Workers Planned: 4
      -> Partial Aggregate
          -> Parallel Seq Scan on large_table
```

### Optymalizować dla równoległości

```sql
-- Tabele z wieloma kolumnami : zmniejszyć work_mem na worker
SET work_mem = '64MB';

-- Tabele partycjonowane : równoległość według partycji
-- Każda partycja może być skanowana równolegle
```

---

## Optymalizacja typów danych

### Wybrać odpowiedni typ

```sql
-- Unikać TEXT dla wartości ograniczonych
-- Przed
CREATE TABLE users (
    id SERIAL,
    status TEXT  -- 'active', 'inactive', 'pending'
);

-- Po : Używać ENUM lub VARCHAR
CREATE TYPE user_status AS ENUM ('active', 'inactive', 'pending');
CREATE TABLE users (
    id SERIAL,
    status user_status
);

-- Lub VARCHAR z ograniczeniem
CREATE TABLE users (
    id SERIAL,
    status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'pending'))
);
```

### Typy numeryczne

```sql
-- Używać najmniejszego możliwego typu
-- Przed
CREATE TABLE products (
    id BIGINT,  -- Jeśli nigdy > 2 miliardy
    price DECIMAL(10,2)
);

-- Po : Dostosować według potrzeb
CREATE TABLE products (
    id INTEGER,  -- Wystarczające dla < 2 miliardy
    price NUMERIC(10,2)  -- NUMERIC = DECIMAL
);
```

### Typy daty/godziny

```sql
-- Używać TIMESTAMP WITH TIME ZONE dla dat/godzin
CREATE TABLE events (
    id SERIAL,
    created_at TIMESTAMPTZ,  -- Przechowuje z timezone
    event_date DATE  -- Dla dat tylko
);

-- Indeks na datach
CREATE INDEX idx_events_created_at ON events(created_at);
```

### JSON vs kolumny normalne

```sql
-- JSON : Elastyczne ale wolniejsze
CREATE TABLE products (
    id SERIAL,
    metadata JSONB
);

-- Kolumny normalne : Szybsze jeśli struktura stała
CREATE TABLE products (
    id SERIAL,
    brand TEXT,
    category TEXT,
    tags TEXT[]
);

-- Indeks GIN dla JSONB
CREATE INDEX idx_products_metadata_gin ON products USING gin(metadata);
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Indeksy na kluczach złączenia** : Niezbędne dla szybkich złączeń
2. **Filtrować przed agregacją** : Zmniejszyć rozmiar danych
3. **Unikać podzapytań skorelowanych** : Używać JOIN zamiast
4. **Partycjonować duże tabele** : Poprawia wydajność i konserwację
5. **Równoległość** : Automatyczna, ale konfigurowalna
6. **Typy danych** : Wybrać najbardziej odpowiedni

## 🔗 Następny moduł

Przejdź do modułu [6. Przypadki praktyczne](../06-cas-pratiques/README.md), aby zobaczyć konkretne przykłady optymalizacji.

