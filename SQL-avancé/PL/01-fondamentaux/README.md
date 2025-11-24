# 1. Podstawy optymalizacji PostgreSQL

## 🎯 Cele

- Zrozumieć architekturę PostgreSQL i planistę zapytań
- Opanować różne typy indeksów i ich optymalne wykorzystanie
- Zrozumieć rolę statystyk w optymalizacji

## 📋 Spis treści

1. [Architektura PostgreSQL](#architektura-postgresql)
2. [Planista zapytań](#planista-zapytań)
3. [Typy indeksów](#typy-indeksów)
4. [Statystyki i ANALYZE](#statystyki-i-analyze)

---

## Architektura PostgreSQL

### Kluczowe komponenty

PostgreSQL wykorzystuje architekturę wieloprocesową z kilkoma ważnymi komponentami :

- **Postmaster** : Główny proces zarządzający połączeniami
- **Backend processes** : Jeden proces na połączenie klienta
- **Planista (Planner)** : Optymalizuje zapytania SQL
- **Wykonawca (Executor)** : Wykonuje plany zapytań

### Przepływ wykonania zapytania

```
Zapytanie SQL
    ↓
Parser (analiza składniowa)
    ↓
Rewriter (przepisywanie widoków/reguł)
    ↓
Planner (generowanie planu wykonania)
    ↓
Executor (wykonanie planu)
    ↓
Wynik
```

### Planista zapytań

Planista jest odpowiedzialny za :
- Wybór najlepszego planu wykonania
- Szacowanie kosztów każdej operacji
- Wykorzystanie statystyk bazy danych
- Optymalizację złączeń, sortowania, agregacji

**Czynniki wpływające na planistę :**
- Statystyki tabel (`pg_stat_user_tables`)
- Statystyki kolumn (`pg_stats`)
- Konfiguracja kosztów (`random_page_cost`, `seq_page_cost`, itp.)
- Parametry pamięci (`work_mem`, `shared_buffers`)

---

## Planista zapytań

### Parametry kosztów

PostgreSQL wykorzystuje system kosztów do porównywania planów :

```sql
-- Zobaczyć aktualne parametry kosztów
SHOW random_page_cost;
SHOW seq_page_cost;
SHOW cpu_tuple_cost;
SHOW cpu_index_tuple_cost;
```

**Ważne parametry :**
- `seq_page_cost` : Koszt odczytu sekwencyjnego (domyślnie: 1.0)
- `random_page_cost` : Koszt odczytu losowego (domyślnie: 4.0)
- `cpu_tuple_cost` : Koszt przetworzenia wiersza (domyślnie: 0.01)
- `cpu_index_tuple_cost` : Koszt wykorzystania indeksu (domyślnie: 0.005)

### Szacowanie kosztów

Planista szacuje :
- **Liczbę wierszy** : Na podstawie statystyk
- **Koszt I/O** : Odczyt/zapis dysku
- **Koszt CPU** : Przetwarzanie danych
- **Koszt całkowity** : Suma kosztów

**Ważne ograniczenie :** Szacowania mogą być niedokładne, jeśli statystyki są przestarzałe.

---

## Typy indeksów

### Indeks B-tree (domyślny)

**Wykorzystanie :**
- Równość (`=`)
- Porównania (`<`, `>`, `<=`, `>=`)
- BETWEEN, IN
- LIKE z ustalonym prefiksem

**Przykład :**
```sql
CREATE INDEX idx_user_email ON users(email);
-- Używany dla: WHERE email = 'user@example.com'
```

### Indeks Hash

**Wykorzystanie :**
- Tylko dla równości (`=`)
- Szybszy niż B-tree dla prostej równości
- Nie obsługuje porównań

**Przykład :**
```sql
CREATE INDEX idx_user_id_hash ON users USING hash(id);
-- Używany dla: WHERE id = 123
```

### Indeks GIN (Generalized Inverted Index)

**Wykorzystanie :**
- Złożone typy danych (tablice, JSONB, full-text)
- Zaawansowane operatory wyszukiwania

**Przykład :**
```sql
CREATE INDEX idx_product_tags_gin ON products USING gin(tags);
-- Używany dla: WHERE tags @> ARRAY['electronics']
```

### Indeks GiST (Generalized Search Tree)

**Wykorzystanie :**
- Typy geometryczne
- Wyszukiwanie pełnotekstowe
- Typy niestandardowe

**Przykład :**
```sql
CREATE INDEX idx_location_gist ON places USING gist(location);
-- Używany dla: WHERE location <-> point(0,0) < 1000
```

### Indeks BRIN (Block Range Index)

**Wykorzystanie :**
- Duże tabele z posortowanymi danymi
- Bardzo kompaktowy (mało miejsca)
- Skuteczny dla zakresów wartości

**Przykład :**
```sql
CREATE INDEX idx_orders_date_brin ON orders USING brin(order_date);
-- Używany dla: WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'
```

### Indeks częściowy

**Wykorzystanie :**
- Zmniejszenie rozmiaru indeksu
- Poprawa wydajności dla określonych warunków

**Przykład :**
```sql
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
-- Indeks tylko dla aktywnych użytkowników
```

### Indeks złożony

**Wykorzystanie :**
- Wiele kolumn
- Kolejność kolumn jest ważna

**Przykład :**
```sql
CREATE INDEX idx_user_name_email ON users(last_name, first_name, email);
-- Używany dla: WHERE last_name = 'Doe' AND first_name = 'John'
```

**Ważna zasada :** Indeks może być użyty, jeśli zapytanie wykorzystuje kolumny w kolejności indeksu, zaczynając od pierwszej.

---

## Statystyki i ANALYZE

### Dlaczego statystyki są istotne

Planista wykorzystuje statystyki do :
- Szacowania liczby zwracanych wierszy
- Wyboru między różnymi planami wykonania
- Określenia kolejności złączeń

### Zbieranie statystyk

```sql
-- Analizować konkretną tabelę
ANALYZE table_name;

-- Analizować wszystkie tabele
ANALYZE;

-- Analizować z poziomem szczegółowości
ANALYZE VERBOSE table_name;
```

**Kiedy wykonać ANALYZE :**
- Po znaczących modyfikacjach (INSERT, UPDATE, DELETE)
- Po utworzeniu indeksu
- Automatycznie przez autovacuum (konfigurowalne)

### Konfiguracja autovacuum

```sql
-- Zobaczyć aktualną konfigurację
SHOW autovacuum;
SHOW autovacuum_analyze_scale_factor;
SHOW autovacuum_analyze_threshold;

-- Zmodyfikować dla konkretnej tabeli
ALTER TABLE large_table SET (
    autovacuum_analyze_scale_factor = 0.05,
    autovacuum_analyze_threshold = 10000
);
```

### Konsultowanie statystyk

```sql
-- Statystyki tabel
SELECT 
    schemaname,
    tablename,
    n_tup_ins AS inserts,
    n_tup_upd AS updates,
    n_tup_del AS deletes,
    n_live_tup AS live_rows,
    n_dead_tup AS dead_rows,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;

-- Statystyki kolumn
SELECT 
    schemaname,
    tablename,
    attname AS column_name,
    n_distinct,
    correlation,
    most_common_vals
FROM pg_stats
WHERE tablename = 'your_table';
```

### Statystyki rozszerzone

PostgreSQL 10+ obsługuje statystyki rozszerzone :

```sql
-- Utworzyć statystyki na wielu kolumnach
CREATE STATISTICS stats_user_name_email 
ON users(last_name, first_name);

-- Analizować, aby zebrać statystyki
ANALYZE users;

-- Konsultować statystyki rozszerzone
SELECT * FROM pg_statistic_ext;
```

**Użyteczność :** Poprawia szacowania dla zapytań z wieloma skorelowanymi kolumnami.

---

## 📊 Kluczowe punkty do zapamiętania

1. **Planista zależy od statystyk** : Przestarzałe statystyki = złe plany
2. **Wybór odpowiedniego typu indeksu** : Każdy typ ma swoje zalety
3. **Regularne ANALYZE** : Istotne dla utrzymania dobrych wydajności
4. **Zrozumienie kosztów** : Pomaga interpretować plany wykonania

## 🔗 Następny moduł

Przejdź do modułu [2. Analiza planów wykonania](../02-plans-execution/README.md), aby nauczyć się interpretować plany wykonania.

