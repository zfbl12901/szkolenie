# 7. Ćwiczenia praktyczne

## 🎯 Cele

- Stosować zdobytą wiedzę
- Rozwiązywać problemy wydajnościowe
- Analizować i optymalizować rzeczywiste zapytania
- Używać Dalibo do analizy

## 📋 Struktura ćwiczeń

Ćwiczenia są zorganizowane według poziomu trudności :
- 🟢 **Początkujący** : Podstawowe koncepcje
- 🟡 **Średnio zaawansowany** : Zaawansowane techniki
- 🔴 **Zaawansowany** : Złożone optymalizacje

---

## Ćwiczenie 1 : Analiza prostego zapytania (🟢 Początkujący)

### Kontekst

Masz tabelę `products` z 1 milionem wierszy i wolne zapytanie :

```sql
SELECT * FROM products WHERE category = 'electronics';
```

### Zadania

1. **Analizować zapytanie** z `EXPLAIN ANALYZE`
2. **Identyfikować problem** w planie wykonania
3. **Zaproponować rozwiązanie** z indeksem
4. **Sprawdzić poprawę** z `EXPLAIN ANALYZE`

### Dane testowe

```sql
-- Utworzyć tabelę
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    category TEXT,
    price DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Wstawić dane testowe (1 milion wierszy)
INSERT INTO products (name, category, price)
SELECT 
    'Product ' || i,
    CASE (i % 10)
        WHEN 0 THEN 'electronics'
        WHEN 1 THEN 'clothing'
        WHEN 2 THEN 'books'
        WHEN 3 THEN 'food'
        WHEN 4 THEN 'toys'
        WHEN 5 THEN 'electronics'
        WHEN 6 THEN 'furniture'
        WHEN 7 THEN 'electronics'
        WHEN 8 THEN 'sports'
        ELSE 'other'
    END,
    (RANDOM() * 1000)::DECIMAL(10,2)
FROM generate_series(1, 1000000) i;

-- Analizować tabelę
ANALYZE products;
```

### Oczekiwane rozwiązanie

<details>
<summary>Kliknij, aby zobaczyć rozwiązanie</summary>

```sql
-- 1. Analizować zapytanie
EXPLAIN ANALYZE
SELECT * FROM products WHERE category = 'electronics';

-- Zidentyfikowany problem : Seq Scan na 1 milionie wierszy

-- 2. Utworzyć indeks
CREATE INDEX idx_products_category ON products(category);

-- 3. Analizować ponownie
EXPLAIN ANALYZE
SELECT * FROM products WHERE category = 'electronics';

-- Oczekiwany wynik : Index Scan z czasem wykonania < 100ms
```
</details>

---

## Ćwiczenie 2 : Optymalizacja złączenia (🟡 Średnio zaawansowany)

### Kontekst

Masz dwie tabele `users` i `orders` z wolnym złączeniem :

```sql
SELECT 
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email
HAVING COUNT(o.id) > 5;
```

### Zadania

1. **Analizować zapytanie** z `EXPLAIN ANALYZE`
2. **Identyfikować problemy** (skanowania, złączenia, agregacje)
3. **Utworzyć potrzebne indeksy**
4. **Optymalizować zapytanie** (HAVING → WHERE jeśli możliwe)
5. **Zmierzyć poprawę**

### Dane testowe

```sql
-- Utworzyć tabele
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    amount DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Wstawić dane
INSERT INTO users (name, email, created_at)
SELECT 
    'User ' || i,
    'user' || i || '@example.com',
    NOW() - (RANDOM() * 365 || ' days')::INTERVAL
FROM generate_series(1, 100000) i;

INSERT INTO orders (user_id, amount, created_at)
SELECT 
    (RANDOM() * 100000)::INTEGER + 1,
    (RANDOM() * 1000)::DECIMAL(10,2),
    NOW() - (RANDOM() * 365 || ' days')::INTERVAL
FROM generate_series(1, 1000000) i;

ANALYZE users, orders;
```

### Oczekiwane rozwiązanie

<details>
<summary>Kliknij, aby zobaczyć rozwiązanie</summary>

```sql
-- 1. Analizować zapytanie
EXPLAIN ANALYZE
SELECT 
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email
HAVING COUNT(o.id) > 5;

-- Zidentyfikowane problemy :
-- - Brak indeksu na users.created_at
-- - Brak indeksu na orders.user_id
-- - HAVING może być zoptymalizowane

-- 2. Utworzyć indeksy
CREATE INDEX idx_users_created_at ON users(created_at);
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 3. Optymalizować zapytanie (filtrować w podzapytaniu)
SELECT 
    u.name,
    u.email,
    o.order_count,
    o.total_amount
FROM users u
JOIN (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        SUM(amount) AS total_amount
    FROM orders
    GROUP BY user_id
    HAVING COUNT(*) > 5
) o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';

-- 4. Analizować ponownie
EXPLAIN ANALYZE [zoptymalizowane zapytanie];

-- Oczekiwany wynik : Czas wykonania zmniejszony o 80%+
```
</details>

---

## Ćwiczenie 3 : Podzapytanie skorelowane (🟡 Średnio zaawansowany)

### Kontekst

Zapytanie z podzapytaniami skorelowanymi bardzo wolne :

```sql
SELECT 
    u.id,
    u.name,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count,
    (SELECT MAX(created_at) FROM orders o WHERE o.user_id = u.id) AS last_order_date,
    (SELECT AVG(amount) FROM orders o WHERE o.user_id = u.id) AS avg_order_amount
FROM users u
WHERE u.active = true;
```

### Zadania

1. **Analizować zapytanie** i identyfikować podzapytania skorelowane
2. **Konwertować na JOIN** z agregacją
3. **Utworzyć potrzebne indeksy**
4. **Porównać wydajność**

### Oczekiwane rozwiązanie

<details>
<summary>Kliknij, aby zobaczyć rozwiązanie</summary>

```sql
-- 1. Analizować
EXPLAIN ANALYZE [oryginalne zapytanie];
-- Problem : 3 podzapytania wykonywane dla każdego wiersza

-- 2. Konwertować na JOIN
SELECT 
    u.id,
    u.name,
    COALESCE(o.order_count, 0) AS order_count,
    o.last_order_date,
    COALESCE(o.avg_order_amount, 0) AS avg_order_amount
FROM users u
LEFT JOIN (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        MAX(created_at) AS last_order_date,
        AVG(amount) AS avg_order_amount
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id
WHERE u.active = true;

-- 3. Utworzyć indeks
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 4. Porównać
-- Oczekiwany wynik : Poprawa o 95%+
```
</details>

---

## Ćwiczenie 4 : Analiza z Dalibo (🔴 Zaawansowany)

### Kontekst

Musisz przeanalizować wydajność bazy danych produkcyjnej (symulowanej) i zidentyfikować główne problemy.

### Zadania

1. **Aktywować pg_stat_statements** i pg_qualstats
2. **Wykonać zapytania testowe** do generowania statystyk
3. **Identyfikować wolne zapytania** z pg_stat_statements
4. **Identyfikować brakujące indeksy** z pg_qualstats
5. **Generować raport** z rekomendacjami

### Dane testowe

```sql
-- Używać tabel z ćwiczenia 2
-- Wykonać różne zapytania do generowania statystyk

-- Zapytania testowe
SELECT * FROM users WHERE email = 'user50000@example.com';
SELECT * FROM orders WHERE user_id = 12345;
SELECT * FROM users WHERE created_at > '2024-06-01';
SELECT COUNT(*) FROM orders WHERE amount > 500;
```

### Oczekiwane rozwiązanie

<details>
<summary>Kliknij, aby zobaczyć rozwiązanie</summary>

```sql
-- 1. Aktywować rozszerzenia
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pg_qualstats;

-- 2. Wykonać zapytania testowe
[zapytania testowe]

-- 3. Identyfikować wolne zapytania
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    mean_exec_time,
    total_exec_time,
    (total_exec_time / SUM(total_exec_time) OVER ()) * 100 AS percent_total_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- 4. Identyfikować brakujące indeksy
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
ORDER BY execution_count DESC;

-- 5. Generować rekomendacje
SELECT 
    'CREATE INDEX idx_' || left_table || '_' || left_column || 
    ' ON ' || left_schema || '.' || left_table || 
    ' (' || left_column || ');' AS recommendation,
    execution_count AS priority
FROM [zapytanie pg_qualstats powyżej]
ORDER BY execution_count DESC
LIMIT 5;
```
</details>

---

## Ćwiczenie 5 : Optymalizacja kompletna (🔴 Zaawansowany)

### Kontekst

Masz aplikację e-commerce z obniżoną wydajnością. Przeanalizuj i zoptymalizuj kompletny system.

### Schemat bazy danych

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    email TEXT,
    name TEXT,
    created_at TIMESTAMP,
    active BOOLEAN
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    category TEXT,
    price DECIMAL(10,2),
    stock INTEGER
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    created_at TIMESTAMP,
    status TEXT,
    total_amount DECIMAL(10,2)
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER,
    price DECIMAL(10,2)
);
```

### Zapytania do optymalizacji

1. **Zapytanie dashboardu** :
```sql
SELECT 
    c.name,
    c.email,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent,
    MAX(o.created_at) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE c.active = true
GROUP BY c.id, c.name, c.email
HAVING COUNT(o.id) > 0
ORDER BY total_spent DESC
LIMIT 100;
```

2. **Zapytanie popularnych produktów** :
```sql
SELECT 
    p.name,
    p.category,
    SUM(oi.quantity) AS total_sold,
    SUM(oi.quantity * oi.price) AS total_revenue
FROM products p
JOIN order_items oi ON p.id = oi.product_id
JOIN orders o ON oi.order_id = o.id
WHERE o.created_at > '2024-01-01'
  AND o.status = 'completed'
GROUP BY p.id, p.name, p.category
ORDER BY total_sold DESC
LIMIT 50;
```

3. **Zapytanie wyszukiwania** :
```sql
SELECT 
    p.*,
    (SELECT COUNT(*) FROM order_items oi WHERE oi.product_id = p.id) AS times_ordered
FROM products p
WHERE p.category = 'electronics'
  AND p.price BETWEEN 100 AND 500
ORDER BY times_ordered DESC;
```

### Zadania

1. **Analizować każde zapytanie** z EXPLAIN ANALYZE
2. **Identyfikować wszystkie problemy**
3. **Utworzyć plan optymalizacji**
4. **Zaimplementować optymalizacje**
5. **Zmierzyć globalny wpływ**

### Oczekiwane rozwiązanie

<details>
<summary>Kliknij, aby zobaczyć rozwiązanie</summary>

```sql
-- 1. Analizować wszystkie zapytania
EXPLAIN ANALYZE [każde zapytanie];

-- 2. Utworzyć potrzebne indeksy
CREATE INDEX idx_customers_active ON customers(active);
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_created_at_status ON orders(created_at, status);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_products_category_price ON products(category, price);

-- 3. Optymalizować zapytanie 1 (HAVING → WHERE)
SELECT 
    c.name,
    c.email,
    o.order_count,
    o.total_spent,
    o.last_order_date
FROM customers c
JOIN (
    SELECT 
        customer_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent,
        MAX(created_at) AS last_order_date
    FROM orders
    GROUP BY customer_id
) o ON c.id = o.customer_id
WHERE c.active = true
ORDER BY o.total_spent DESC
LIMIT 100;

-- 4. Optymalizować zapytanie 3 (podzapytanie → JOIN)
SELECT 
    p.*,
    COALESCE(oi.times_ordered, 0) AS times_ordered
FROM products p
LEFT JOIN (
    SELECT product_id, COUNT(*) AS times_ordered
    FROM order_items
    GROUP BY product_id
) oi ON p.id = oi.product_id
WHERE p.category = 'electronics'
  AND p.price BETWEEN 100 AND 500
ORDER BY oi.times_ordered DESC NULLS LAST;

-- 5. Zmierzyć wpływ
-- Porównać czasy wykonania przed/po
```
</details>

---

## Ćwiczenie 6 : Monitoring i alerty (🔴 Zaawansowany)

### Kontekst

Utwórz system monitoringu do automatycznego wykrywania problemów wydajnościowych.

### Zadania

1. **Utworzyć widok** konsolidujący kluczowe metryki
2. **Utworzyć funkcję** wykrywania alertów
3. **Utworzyć raport** automatyczny
4. **Przetestować system** z rzeczywistymi danymi

### Oczekiwane rozwiązanie

<details>
<summary>Kliknij, aby zobaczyć rozwiązanie</summary>

```sql
-- 1. Skonsolidowany widok
CREATE OR REPLACE VIEW v_performance_metrics AS
SELECT 
    'slow_queries' AS metric_type,
    COUNT(*) AS count,
    AVG(mean_exec_time) AS avg_value,
    MAX(mean_exec_time) AS max_value
FROM pg_stat_statements
WHERE mean_exec_time > 1000

UNION ALL

SELECT 
    'cache_hit_ratio',
    NULL,
    ROUND(100.0 * SUM(shared_blks_hit) / 
          NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2),
    NULL
FROM pg_stat_statements

UNION ALL

SELECT 
    'idle_in_transaction',
    COUNT(*),
    NULL,
    NULL
FROM pg_stat_activity
WHERE state = 'idle in transaction';

-- 2. Funkcja alertów
CREATE OR REPLACE FUNCTION check_performance_alerts()
RETURNS TABLE(alert_type TEXT, message TEXT, severity TEXT) AS $$
BEGIN
    -- Alerty na wolne zapytania
    RETURN QUERY
    SELECT 
        'SLOW_QUERY'::TEXT,
        format('Query with mean_exec_time: %s ms', ROUND(mean_exec_time::numeric, 2)),
        CASE WHEN mean_exec_time > 5000 THEN 'CRITICAL' ELSE 'WARNING' END
    FROM pg_stat_statements
    WHERE mean_exec_time > 1000
    ORDER BY mean_exec_time DESC
    LIMIT 5;

    -- Alert na współczynnik trafień cache
    RETURN QUERY
    SELECT 
        'LOW_CACHE_HIT'::TEXT,
        format('Cache hit ratio: %s%%', 
               ROUND(100.0 * SUM(shared_blks_hit) / 
                     NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2)),
        CASE 
            WHEN 100.0 * SUM(shared_blks_hit) / 
                 NULLIF(SUM(shared_blks_hit + shared_blks_read), 0) < 90 
            THEN 'CRITICAL'
            WHEN 100.0 * SUM(shared_blks_hit) / 
                 NULLIF(SUM(shared_blks_hit + shared_blks_read), 0) < 95 
            THEN 'WARNING'
            ELSE 'INFO'
        END
    FROM pg_stat_statements;

    -- Alert na połączenia idle in transaction
    RETURN QUERY
    SELECT 
        'IDLE_IN_TRANSACTION'::TEXT,
        format('%s connections idle in transaction', COUNT(*)),
        CASE WHEN COUNT(*) > 10 THEN 'CRITICAL' ELSE 'WARNING' END
    FROM pg_stat_activity
    WHERE state = 'idle in transaction';
END;
$$ LANGUAGE plpgsql;

-- 3. Użycie
SELECT * FROM check_performance_alerts();
```
</details>

---

## 📝 Wskazówki do ćwiczeń

1. **Zawsze zaczynać od EXPLAIN ANALYZE** : Rozumieć przed optymalizacją
2. **Mierzyć przed i po** : Kwantyfikować poprawę
3. **Testować z realistycznymi danymi** : Używać objętości podobnych do produkcji
4. **Dokumentować decyzje** : Notować dlaczego wybrałeś taką optymalizację
5. **Sprawdzać efekty uboczne** : Indeks poprawia SELECT ale spowalnia INSERT/UPDATE

## 🔗 Powrót do modułów

- [Moduł 1 : Podstawy](../01-fondamentaux/README.md)
- [Moduł 2 : Plany wykonania](../02-plans-execution/README.md)
- [Moduł 3 : Dalibo](../03-dalibo/README.md)
- [Moduł 4 : Wskaźniki](../04-indicateurs/README.md)
- [Moduł 5 : Techniki](../05-techniques/README.md)
- [Moduł 6 : Przypadki praktyczne](../06-cas-pratiques/README.md)

