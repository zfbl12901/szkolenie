# 2. Analiza planów wykonania

## 🎯 Cele

- Opanować EXPLAIN i EXPLAIN ANALYZE
- Interpretować różne typy operacji
- Rozumieć koszty i czasy wykonania
- Identyfikować problemy wydajnościowe w planach

## 📋 Spis treści

1. [EXPLAIN i EXPLAIN ANALYZE](#explain-i-explain-analyze)
2. [Typy operacji](#typy-operacji)
3. [Interpretacja kosztów](#interpretacja-kosztów)
4. [Sygnały alarmowe](#sygnały-alarmowe)
5. [Dobre praktyki](#dobre-praktyki)

---

## EXPLAIN i EXPLAIN ANALYZE

### EXPLAIN (bez wykonania)

Wyświetla szacowany plan wykonania **bez wykonywania zapytania** :

```sql
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';
```

**Wynik :**
```
Seq Scan on users  (cost=0.00..25.00 rows=1 width=64)
  Filter: (email = 'user@example.com'::text)
```

### EXPLAIN ANALYZE (z wykonaniem)

Wykonuje zapytanie i wyświetla **rzeczywiste czasy** :

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'user@example.com';
```

**Wynik :**
```
Seq Scan on users  (cost=0.00..25.00 rows=1 width=64) 
  (actual time=0.123..15.456 rows=1 loops=1)
  Filter: (email = 'user@example.com'::text)
  Rows Removed by Filter: 9999
Planning Time: 0.234 ms
Execution Time: 15.678 ms
```

### Przydatne opcje

```sql
-- Format JSON (dla narzędzi zewnętrznych)
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) 
SELECT * FROM users WHERE email = 'user@example.com';

-- Wyświetlić bufory
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM users WHERE email = 'user@example.com';

-- Wyświetlić parametry planowania
EXPLAIN (ANALYZE, VERBOSE, SETTINGS) 
SELECT * FROM users WHERE email = 'user@example.com';

-- Format YAML
EXPLAIN (ANALYZE, FORMAT YAML) 
SELECT * FROM users WHERE email = 'user@example.com';
```

### Interpretacja metryk

**Szacowane koszty :**
- `cost=0.00..25.00` : Koszt startowy..Koszt całkowity
- `rows=1` : Szacowana liczba wierszy
- `width=64` : Średni rozmiar wiersza w bajtach

**Rzeczywiste czasy (ANALYZE) :**
- `actual time=0.123..15.456` : Czas startowy..Czas całkowity (ms)
- `rows=1` : Rzeczywista liczba zwróconych wierszy
- `loops=1` : Liczba wykonań tej operacji
- `Planning Time` : Czas planowania
- `Execution Time` : Całkowity czas wykonania

**Bufory (z BUFFERS) :**
- `shared hit=15` : Strony odczytane z cache współdzielonego
- `shared read=3` : Strony odczytane z dysku
- `shared written=0` : Strony zapisane
- `temp read/written` : Strony tymczasowe

---

## Typy operacji

### Seq Scan (Skanowanie sekwencyjne)

**Kiedy używane :**
- Brak odpowiedniego indeksu
- Mała tabela (< 10% tabeli)
- Indeks nie selektywny

**Przykład :**
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE status = 'inactive';
```

**Interpretacja :**
- ✅ Akceptowalne dla małych tabel
- ⚠️ Problematyczne dla dużych tabel
- 🔍 **Działanie** : Utworzyć indeks jeśli tabela jest duża

### Index Scan

**Kiedy używane :**
- Indeks dostępny i selektywny
- Bezpośredni dostęp przez indeks

**Przykład :**
```sql
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'user@example.com';
```

**Typowy wynik :**
```
Index Scan using idx_users_email on users  
  (cost=0.42..8.44 rows=1 width=64)
  (actual time=0.123..0.125 rows=1 loops=1)
  Index Cond: (email = 'user@example.com'::text)
```

**Interpretacja :**
- ✅ Dobra wydajność
- ✅ Bezpośredni dostęp do wierszy

### Index Only Scan

**Kiedy używane :**
- Wszystkie potrzebne kolumny są w indeksie
- Nie ma potrzeby dostępu do tabeli

**Przykład :**
```sql
-- Indeks na (id, email)
CREATE INDEX idx_users_id_email ON users(id, email);

EXPLAIN ANALYZE 
SELECT id, email FROM users WHERE id BETWEEN 1 AND 100;
```

**Typowy wynik :**
```
Index Only Scan using idx_users_id_email on users
  (cost=0.42..5.44 rows=100 width=64)
  (actual time=0.123..0.456 rows=100 loops=1)
  Index Cond: ((id >= 1) AND (id <= 100))
  Heap Fetches: 0
```

**Interpretacja :**
- ✅ Optymalna wydajność
- ✅ `Heap Fetches: 0` = brak dostępu do tabeli

### Bitmap Index Scan + Bitmap Heap Scan

**Kiedy używane :**
- Wiele warunków z kilkoma indeksami
- Zwraca wiele wierszy

**Przykład :**
```sql
EXPLAIN ANALYZE 
SELECT * FROM users 
WHERE status = 'active' AND created_at > '2024-01-01';
```

**Typowy wynik :**
```
Bitmap Heap Scan on users
  (cost=4.44..25.67 rows=50 width=64)
  (actual time=0.234..1.456 rows=45 loops=1)
  Recheck Cond: ((status = 'active'::text) AND (created_at > '2024-01-01'::date))
  Heap Blocks: exact=12
  -> Bitmap Index Scan on idx_users_status
      (cost=0.00..4.43 rows=50 width=0)
      (actual time=0.123..0.123 rows=45 loops=1)
      Index Cond: (status = 'active'::text)
```

**Interpretacja :**
- ✅ Skuteczne dla wielu warunków
- ⚠️ `Recheck Cond` = dodatkowa weryfikacja

### Nested Loop

**Kiedy używane :**
- Małe tabele lub ograniczone wyniki
- Mała tabela zewnętrzna

**Przykład :**
```sql
EXPLAIN ANALYZE 
SELECT u.*, o.* 
FROM users u 
JOIN orders o ON u.id = o.user_id 
WHERE u.id = 123;
```

**Typowy wynik :**
```
Nested Loop
  (cost=0.85..25.67 rows=10 width=128)
  (actual time=0.123..2.456 rows=8 loops=1)
  -> Index Scan using idx_users_id on users
      (cost=0.42..8.44 rows=1 width=64)
      (actual time=0.089..0.090 rows=1 loops=1)
      Index Cond: (id = 123)
  -> Index Scan using idx_orders_user_id on orders
      (cost=0.42..17.23 rows=10 width=64)
      (actual time=0.234..2.345 rows=8 loops=1)
      Index Cond: (user_id = 123)
```

**Interpretacja :**
- ✅ Skuteczne dla małych pętli
- ⚠️ Może być wolne jeśli pętla zewnętrzna jest duża

### Hash Join

**Kiedy używane :**
- Tabele podobnej wielkości
- Brak indeksu na kluczu złączenia
- Prosta równość

**Przykład :**
```sql
EXPLAIN ANALYZE 
SELECT u.*, o.* 
FROM users u 
JOIN orders o ON u.id = o.user_id;
```

**Typowy wynik :**
```
Hash Join
  (cost=125.67..456.78 rows=10000 width=128)
  (actual time=2.345..15.678 rows=9876 loops=1)
  Hash Cond: (o.user_id = u.id)
  -> Seq Scan on orders o
      (cost=0.00..234.56 rows=10000 width=64)
      (actual time=0.123..5.678 rows=10000 loops=1)
  -> Hash
      (cost=123.45..123.45 rows=1000 width=64)
      (actual time=1.234..1.234 rows=1000 loops=1)
      Buckets: 1024  Batches: 1  Memory Usage: 64kB
      -> Seq Scan on users u
          (cost=0.00..123.45 rows=1000 width=64)
          (actual time=0.089..0.567 rows=1000 loops=1)
```

**Interpretacja :**
- ✅ Skuteczne dla złączeń równościowych
- ⚠️ Wymaga pamięci (`work_mem`)
- 🔍 **Działanie** : Zwiększyć `work_mem` jeśli "Batches > 1"

### Merge Join

**Kiedy używane :**
- Dane już posortowane
- Złączenia na posortowanych kluczach
- Operatory porównania (<, >, <=, >=)

**Przykład :**
```sql
EXPLAIN ANALYZE 
SELECT u.*, o.* 
FROM users u 
JOIN orders o ON u.id = o.user_id 
ORDER BY u.id;
```

**Interpretacja :**
- ✅ Skuteczne jeśli dane są posortowane
- ⚠️ Wymaga sortowania jeśli dane nie są posortowane

### Sort

**Kiedy używane :**
- ORDER BY
- GROUP BY (czasami)
- Operacje wymagające sortowania

**Przykład :**
```sql
EXPLAIN ANALYZE 
SELECT * FROM users ORDER BY created_at DESC LIMIT 100;
```

**Typowy wynik :**
```
Limit
  (cost=234.56..256.78 rows=100 width=64)
  (actual time=12.345..15.678 rows=100 loops=1)
  -> Sort
      (cost=234.56..256.78 rows=10000 width=64)
      (actual time=12.345..15.234 rows=100 loops=1)
      Sort Key: created_at DESC
      Sort Method: top-N heapsort  Memory: 32kB
      -> Seq Scan on users
          (cost=0.00..123.45 rows=10000 width=64)
          (actual time=0.089..5.678 rows=10000 loops=1)
```

**Interpretacja :**
- ⚠️ `Sort Method: external merge` = sortowanie na dysku (wolne)
- ✅ `Sort Method: quicksort` = sortowanie w pamięci (szybkie)
- 🔍 **Działanie** : Zwiększyć `work_mem` jeśli sortowanie na dysku

### Aggregate

**Kiedy używane :**
- Funkcje agregujące (COUNT, SUM, AVG, etc.)
- GROUP BY

**Przykład :**
```sql
EXPLAIN ANALYZE 
SELECT status, COUNT(*) 
FROM users 
GROUP BY status;
```

**Typowy wynik :**
```
HashAggregate
  (cost=123.45..145.67 rows=5 width=12)
  (actual time=2.345..2.456 rows=5 loops=1)
  Group Key: status
  Batches: 1  Memory Usage: 24kB
  -> Seq Scan on users
      (cost=0.00..98.76 rows=10000 width=4)
      (actual time=0.089..1.234 rows=10000 loops=1)
```

**Interpretacja :**
- ✅ `HashAggregate` = skuteczne
- ⚠️ `GroupAggregate` = może być wolne
- 🔍 **Działanie** : Zwiększyć `work_mem` jeśli "Batches > 1"

---

## Interpretacja kosztów

### Struktura kosztów

```
cost=0.00..25.00
  ↑      ↑
  |      └─ Koszt całkowity
  └─ Koszt startowy
```

**Koszt startowy :** Koszt przed zwróceniem pierwszego wiersza
**Koszt całkowity :** Koszt dla zwrócenia wszystkich wierszy

### Porównanie kosztów

**Ogólna zasada :**
- Koszt < 100 : Bardzo szybko
- Koszt 100-1000 : Szybko
- Koszt 1000-10000 : Umiarkowanie
- Koszt > 10000 : Potencjalnie wolno

**⚠️ Ważne :** Koszty są względne i zależą od konfiguracji.

### Różnica między szacunkiem a rzeczywistością

**Porównać :**
- `rows` (szacowane) vs `rows` (rzeczywiste w ANALYZE)
- `cost` (szacowane) vs `actual time` (rzeczywiste)

**Przykład problematyczny :**
```
Seq Scan on users
  (cost=0.00..25.00 rows=1 width=64)
  (actual time=0.123..1500.456 rows=100000 loops=1)
```

**Problem :** Szacowanie bardzo nieprawidłowe (1 wiersz szacowany, 100000 rzeczywistych)
**Działanie :** Wykonać `ANALYZE users;`

---

## Sygnały alarmowe

### 🔴 Alarmy krytyczne

1. **Seq Scan na dużej tabeli**
   ```
   Seq Scan on large_table (cost=0.00..50000.00 rows=1000000)
   ```
   **Działanie :** Utworzyć odpowiedni indeks

2. **Sortowanie na dysku**
   ```
   Sort Method: external merge  Disk: 50000kB
   ```
   **Działanie :** Zwiększyć `work_mem`

3. **Bardzo nieprawidłowe szacowanie**
   ```
   rows=1 (szacowane) vs rows=100000 (rzeczywiste)
   ```
   **Działanie :** Wykonać `ANALYZE`

4. **Hash Join z wieloma batchami**
   ```
   Hash Join
     Batches: 16  Memory Usage: 512kB
   ```
   **Działanie :** Zwiększyć `work_mem`

5. **Nested Loop z dużą pętlą zewnętrzną**
   ```
   Nested Loop (loops=100000)
   ```
   **Działanie :** Sprawdzić indeksy lub zmienić typ złączenia

### 🟡 Alarmy umiarkowane

1. **Index Scan z wieloma Heap Fetches**
   ```
   Index Only Scan
     Heap Fetches: 50000
   ```
   **Działanie :** Sprawdzić widoczność krotek

2. **Bitmap Heap Scan z wieloma rechecks**
   ```
   Rows Removed by Filter: 50000
   ```
   **Działanie :** Poprawić selektywność indeksu

3. **Wysoki czas planowania**
   ```
   Planning Time: 500.234 ms
   ```
   **Działanie :** Uprościć zapytanie lub zwiększyć `plan_cache_mode`

---

## Dobre praktyki

### 1. Zawsze używać EXPLAIN ANALYZE dla wolnych zapytań

```sql
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT ...;
```

### 2. Porównywać szacunki i rzeczywistość

Sprawdzić czy `rows` (szacowane) ≈ `rows` (rzeczywiste)

### 3. Monitorować bufory

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

- `shared hit` wysoki = dobrze (cache)
- `shared read` wysoki = można poprawić (I/O dysku)

### 4. Identyfikować kosztowne operacje

Szukać operacji z :
- `actual time` wysokim
- `loops` wysokim
- `rows` znacznie wyższym niż szacunek

### 5. Używać narzędzi wizualizacji

- **pgAdmin** : Wizualizacja graficzna planów
- **explain.dalibo.com** : Analiza online
- **pev** : PostgreSQL Explain Visualizer

---

## 📊 Kluczowe punkty do zapamiętania

1. **EXPLAIN** = szacunek, **EXPLAIN ANALYZE** = rzeczywistość
2. **Seq Scan** na dużej tabeli = potencjalny problem
3. **Sortowanie na dysku** = zwiększyć `work_mem`
4. **Nieprawidłowe szacowanie** = wykonać `ANALYZE`
5. **Zawsze porównywać** szacunek vs rzeczywistość

## 🔗 Następny moduł

Przejdź do modułu [3. Dalibo - Narzędzie analizy](../03-dalibo/README.md), aby nauczyć się używać Dalibo do analizy wydajności.

