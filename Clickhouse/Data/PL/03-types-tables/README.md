# 3. Typy danych i tabele

## 🎯 Cele

- Zrozumieć typy danych ClickHouse
- Tworzyć zoptymalizowane tabele
- Wybierać odpowiedni silnik tabeli
- Konfigurować partycjonowanie

## Typy danych

### Typy całkowite

```sql
UInt8, UInt16, UInt32, UInt64  -- Liczby całkowite bez znaku
Int8, Int16, Int32, Int64      -- Liczby całkowite ze znakiem
```

### Typy dziesiętne

```sql
Float32, Float64               -- Liczby zmiennoprzecinkowe
Decimal32, Decimal64, Decimal128  -- Dokładne liczby dziesiętne
```

### Typy łańcuchowe

```sql
String                        -- Łańcuch znaków
FixedString(N)                -- Łańcuch o stałej długości
```

### Typy daty/czasu

```sql
Date                          -- Data (YYYY-MM-DD)
DateTime                      -- Data i czas
DateTime64                    -- Data i czas z precyzją
```

## Tworzenie tabel

### Tabela MergeTree (zalecana)

```sql
CREATE TABLE events
(
    id UInt64,
    event_date Date,
    event_time DateTime,
    user_id UInt32,
    event_type String,
    value Float64
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, user_id);
```

### Tabela Memory

```sql
CREATE TABLE temp_data
(
    id UInt64,
    name String
)
ENGINE = Memory;
```

## Partycjonowanie

### Według daty (zalecane)

```sql
PARTITION BY toYYYYMM(event_date)
```

### Według hash

```sql
PARTITION BY intHash32(user_id) % 10
```

## Silniki tabel

- **MergeTree** : Do danych trwałych, dużych wolumenów
- **Memory** : Do danych tymczasowych
- **Log** : Do małych wolumenów, logów

---

**Następny krok :** [Wydajność i optymalizacja](./04-performance/README.md)

