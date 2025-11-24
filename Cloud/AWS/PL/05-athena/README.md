# 5. Amazon Athena - Zapytania SQL na S3

## 🎯 Cele

- Zrozumieć Amazon Athena i jego użycie
- Tworzyć tabele zewnętrzne wskazujące na S3
- Wykonywać zapytania SQL na plikach S3
- Optymalizować koszty i wydajność
- Integrować z Glue Data Catalog

## 📋 Spis treści

1. [Wprowadzenie do Athena](#wprowadzenie-do-athena)
2. [Tworzyć tabele zewnętrzne](#tworzyć-tabele-zewnętrzne)
3. [Wykonywać zapytania](#wykonywać-zapytania)
4. [Optymalizacja kosztów](#optymalizacja-kosztów)
5. [Integracja z Glue](#integracja-z-glue)
6. [Dobre praktyki](#dobre-praktyki)

---

## Wprowadzenie do Athena

### Czym jest Amazon Athena?

**Amazon Athena** = Usługa zapytań SQL serverless na S3

- **Serverless** : Brak infrastruktury do zarządzania
- **Pay-per-query** : Płacisz tylko za użycie
- **Standard SQL** : Standardowa składnia SQL
- **Bezpośrednio na S3** : Nie potrzeba ładować do bazy danych

### Przypadki użycia dla Data Analyst

- **Eksploracja danych** : Szybko analizować pliki S3
- **Zapytania Data Lake** : Zapytania na data lake
- **Analizy ad-hoc** : Analizy jednorazowe
- **Analiza logów** : Analizować logi przechowywane w S3

### Free Tier Athena

**Darmowe na zawsze :**
- 10 GB danych przeskanowanych/miesiąc
- Poza tym : 5$ za Terabajt przeskanowany

**⚠️ Ważne :** Koszty zależą od ilości przeskanowanych danych. Optymalizować zapytania aby zmniejszyć koszty.

---

## Tworzyć tabele zewnętrzne

### Metoda 1 : Przez edytor Athena

**Krok 1 : Dostęp do Athena**

1. Konsola AWS → Szukać "Athena"
2. Kliknąć "Amazon Athena"
3. Pierwsze użycie : Skonfigurować wynik S3

**Krok 2 : Skonfigurować wynik**

1. "Settings" → "Manage"
2. "Query result location" : `s3://my-bucket/athena-results/`
3. "Save"

**Krok 3 : Utworzyć tabelę**

```sql
-- Tabela dla plików CSV
CREATE EXTERNAL TABLE users (
    id INT,
    name STRING,
    email STRING,
    created_at TIMESTAMP
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
    'serialization.format' = ',',
    'field.delim' = ','
)
STORED AS TEXTFILE
LOCATION 's3://my-bucket/data/users/'
TBLPROPERTIES ('skip.header.line.count'='1');
```

### Metoda 2 : Przez Glue Data Catalog (zalecane)

**Używać tabel utworzonych przez Glue :**

1. Glue → Utworzyć crawler dla S3
2. Crawler automatycznie tworzy tabelę
3. Athena używa bezpośrednio tej tabeli

**Zalety :**
- Automatycznie wykryty schemat
- Nie potrzeba definiować ręcznie
- Wykorzystywane przez inne usługi

### Obsługiwane formaty

**CSV :**
```sql
CREATE EXTERNAL TABLE csv_data (
    col1 STRING,
    col2 INT
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS TEXTFILE
LOCATION 's3://bucket/csv/';
```

**JSON :**
```sql
CREATE EXTERNAL TABLE json_data (
    id INT,
    name STRING
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION 's3://bucket/json/';
```

**Parquet (zalecane) :**
```sql
CREATE EXTERNAL TABLE parquet_data (
    id INT,
    name STRING,
    created_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://bucket/parquet/';
```

---

## Wykonywać zapytania

### Podstawowe zapytania

**Prosty SELECT :**

```sql
SELECT * FROM users LIMIT 10;
```

**Filtrować :**

```sql
SELECT 
    id,
    name,
    email
FROM users
WHERE created_at > DATE '2024-01-01'
ORDER BY created_at DESC;
```

**Agregacje :**

```sql
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS user_count,
    COUNT(DISTINCT email) AS unique_emails
FROM users
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;
```

### Zaawansowane zapytania

**Funkcje okna :**

```sql
SELECT 
    id,
    name,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY DATE_TRUNC('month', created_at) ORDER BY created_at) AS rank
FROM users;
```

**Złączenia :**

```sql
SELECT 
    u.name,
    o.amount,
    o.created_at
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.created_at > DATE '2024-01-01';
```

### Zapytania na partycjach

**Jeśli dane są partycjonowane :**

```sql
-- Tabela partycjonowana według daty
CREATE EXTERNAL TABLE sales (
    id INT,
    product_id INT,
    amount DECIMAL(10,2)
)
PARTITIONED BY (sale_date DATE)
STORED AS PARQUET
LOCATION 's3://bucket/sales/';

-- Dodać partycje
ALTER TABLE sales ADD PARTITION (sale_date='2024-01-01')
LOCATION 's3://bucket/sales/year=2024/month=01/day=01/';

-- Zapytanie z partycją (szybsze i tańsze)
SELECT * FROM sales
WHERE sale_date = DATE '2024-01-01';
```

---

## Optymalizacja kosztów

### Zmniejszyć przeskanowane dane

**1. Używać WHERE aby filtrować wcześnie :**

```sql
-- ❌ Złe : Skanuje wszystko potem filtruje
SELECT * FROM large_table
WHERE date = '2024-01-01';

-- ✅ Dobre : Filtruje od początku (jeśli partycjonowane)
SELECT * FROM large_table
WHERE date = '2024-01-01';
```

**2. Wybierać tylko potrzebne kolumny :**

```sql
-- ❌ Złe : Skanuje wszystkie kolumny
SELECT * FROM large_table;

-- ✅ Dobre : Skanuje tylko potrzebne kolumny
SELECT id, name FROM large_table;
```

**3. Używać LIMIT :**

```sql
-- Ograniczyć liczbę wyników
SELECT * FROM large_table LIMIT 100;
```

### Używać Parquet

**Parquet jest bardziej efektywny niż CSV :**

- **Kompresja** : Mniej przeskanowanych danych
- **Kolumny** : Skanuje tylko potrzebne kolumny
- **Zmniejszone koszty** : Do 90% redukcji

**Konwertować CSV → Parquet z Glue :**

```python
# Job Glue do konwersji
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "csv_data"
)

glueContext.write_dynamic_frame.from_options(
    frame = datasource,
    connection_type = "s3",
    connection_options = {"path": "s3://bucket/parquet/"},
    format = "parquet"
)
```

### Partycjonować dane

**Partycjonować według daty (zalecane) :**

```
s3://bucket/data/
├── year=2024/
│   ├── month=01/
│   │   ├── day=01/
│   │   └── day=02/
│   └── month=02/
```

**Utworzyć tabelę partycjonowaną :**

```sql
CREATE EXTERNAL TABLE partitioned_data (
    id INT,
    name STRING
)
PARTITIONED BY (year INT, month INT, day INT)
STORED AS PARQUET
LOCATION 's3://bucket/data/';
```

---

## Integracja z Glue

### Używać tabel Glue

**Tabele utworzone przez Glue są automatycznie dostępne w Athena :**

1. Glue → Crawler tworzy tabelę
2. Athena → "Tables" → Zobaczyć wszystkie tabele Glue
3. Używać bezpośrednio w zapytaniach

**Zalety :**
- Automatyczny schemat
- Brak ręcznej definicji
- Automatyczna synchronizacja

### Aktualizować partycje

**Jeśli dodane nowe dane :**

```sql
-- Aktualizować partycje
MSCK REPAIR TABLE sales;

-- Lub dodać ręcznie
ALTER TABLE sales ADD PARTITION (sale_date='2024-01-02')
LOCATION 's3://bucket/sales/year=2024/month=01/day=02/';
```

---

## Dobre praktyki

### Wydajność

1. **Używać Parquet** zamiast CSV
2. **Partycjonować dane** według daty/kategorii
3. **Wybierać tylko potrzebne kolumny**
4. **Filtrować wcześnie** z WHERE
5. **Używać LIMIT** do eksploracji

### Koszty

1. **Monitorować przeskanowane dane** w wynikach
2. **Optymalizować zapytania** aby zmniejszyć skanowanie
3. **Używać Parquet** do kompresji
4. **Partycjonować** aby zmniejszyć skanowanie
5. **Buforować** częste wyniki

### Organizacja

1. **Organizować S3** ze spójnymi prefiksami
2. **Nazywać tabele** jasno
3. **Dokumentować schematy**
4. **Używać baz danych** do organizacji

---

## Przykłady praktyczne

### Przykład 1 : Analizować logi

```sql
-- Tabela dla logów
CREATE EXTERNAL TABLE logs (
    timestamp TIMESTAMP,
    level STRING,
    message STRING,
    user_id INT
)
PARTITIONED BY (date DATE)
STORED AS TEXTFILE
LOCATION 's3://bucket/logs/';

-- Zapytanie : Błędy na dzień
SELECT 
    date,
    COUNT(*) AS error_count
FROM logs
WHERE level = 'ERROR'
GROUP BY date
ORDER BY date DESC;
```

### Przykład 2 : Analizować dane CSV

```sql
-- Tabela CSV
CREATE EXTERNAL TABLE sales_csv (
    id INT,
    product_id INT,
    amount DECIMAL(10,2),
    sale_date DATE
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS TEXTFILE
LOCATION 's3://bucket/sales/csv/'
TBLPROPERTIES ('skip.header.line.count'='1');

-- Analiza : Sprzedaż na miesiąc
SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total_sales,
    COUNT(*) AS transaction_count
FROM sales_csv
GROUP BY DATE_TRUNC('month', sale_date)
ORDER BY month;
```

### Przykład 3 : Złączenie wielu tabel

```sql
-- Analizować z złączeniami
SELECT 
    p.name AS product_name,
    c.name AS category_name,
    SUM(s.amount) AS total_sales
FROM sales s
JOIN products p ON s.product_id = p.id
JOIN categories c ON p.category_id = c.id
WHERE s.sale_date >= DATE '2024-01-01'
GROUP BY p.name, c.name
ORDER BY total_sales DESC
LIMIT 10;
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Athena = SQL serverless** na plikach S3
2. **Free Tier : 10 GB/miesiąc** przeskanowanych danych
3. **Parquet** = najbardziej efektywny format
4. **Partycjonować** = zmniejszyć koszty
5. **Integracja Glue** = automatyczne schematy

## 🔗 Następny moduł

Przejdź do modułu [6. AWS Lambda - Serverless Computing](../06-lambda/README.md), aby nauczyć się automatyzować przetwarzanie danych.

