# 4. Amazon Redshift - Data Warehouse

## 🎯 Cele

- Zrozumieć Amazon Redshift i jego rolę
- Utworzyć klaster Redshift (darmowy 2 miesiące)
- Ładować dane do Redshift
- Optymalizować zapytania Redshift
- Integrować z S3 i innymi usługami

## 📋 Spis treści

1. [Wprowadzenie do Redshift](#wprowadzenie-do-redshift)
2. [Utworzyć klaster Redshift](#utworzyć-klaster-redshift)
3. [Ładować dane](#ładować-dane)
4. [Zaawansowane zapytania SQL](#zaawansowane-zapytania-sql)
5. [Optymalizacja](#optymalizacja)
6. [Integracja z innymi usługami](#integracja-z-innymi-usługami)

---

## Wprowadzenie do Redshift

### Czym jest Amazon Redshift?

**Amazon Redshift** = Zarządzany data warehouse w chmurze

- **OLAP** : Zoptymalizowany do analizy (nie transakcje)
- **Kolumnowy** : Przechowywanie zorientowane na kolumny
- **Masowo równoległy** : Przetwarzanie rozproszone
- **Skalowalny** : Od kilku GB do kilku PB

### Przypadki użycia dla Data Analyst

- **Data Warehouse** : Centralizować dane
- **Analytics** : Złożone zapytania na dużych wolumenach
- **Business Intelligence** : Pulpity nawigacyjne i raporty
- **Data Mining** : Dogłębne analizy

### Free Tier Redshift

**Darmowy 2 miesiące :**
- 750 godzin/miesiąc klastra `dc2.large`
- 32 GB przechowywania na węzeł
- Po 2 miesiącach : normalne rozliczanie

**⚠️ Ważne :** Zatrzymać klaster gdy nieużywany, aby uniknąć kosztów.

---

## Utworzyć klaster Redshift

### Krok 1 : Dostęp do Redshift

1. Konsola AWS → Szukać "Redshift"
2. Kliknąć "Amazon Redshift"
3. "Create cluster"

### Krok 2 : Konfiguracja klastra

**Podstawowa konfiguracja :**

1. **Cluster identifier** : `data-analyst-cluster`
2. **Node type** : `dc2.large` (darmowy 2 miesiące)
3. **Number of nodes** : 1 (wystarczające do rozpoczęcia)
4. **Database name** : `analytics` (domyślnie : `dev`)
5. **Database port** : 5439 (domyślnie)
6. **Master username** : `admin` (lub inny)
7. **Master password** : Silne hasło

**Konfiguracja sieci :**

1. **VPC** : Wybrać istniejący VPC
2. **Subnet group** : Utworzyć lub użyć istniejącego
3. **Publicly accessible** : ✅ Tak (dla łatwego dostępu)
4. **Availability zone** : Wybrać strefę

**Bezpieczeństwo :**

1. **VPC security groups** : Utworzyć grupę bezpieczeństwa
   - Zezwolić port 5439 z Twojego IP
2. **Encryption** : Włączyć (zalecane)

### Krok 3 : Utworzyć klaster

1. Kliknąć "Create cluster"
2. Czekać 5-10 minut (tworzenie)
3. Klaster gotowy gdy status = "Available"

**⚠️ Ważne :** Zanotować endpoint klastra (np. `data-analyst-cluster.xxxxx.eu-west-3.redshift.amazonaws.com:5439`)

---

## Ładować dane

### Metoda 1 : COPY z S3 (zalecane)

**Najszybsze dla dużych ilości :**

```sql
-- Utworzyć tabelę
CREATE TABLE users (
    id INTEGER,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP
);

-- Ładować z S3
COPY users
FROM 's3://my-bucket/data/users.csv'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
IGNOREHEADER 1;
```

**Konfiguracja roli IAM :**

1. IAM → "Roles" → "Create role"
2. Typ : "Redshift"
3. Dołączyć politykę : `AmazonS3ReadOnlyAccess`
4. Nazwa : `RedshiftS3Role`
5. Skopiować ARN dla COPY

### Metoda 2 : INSERT (małe ilości)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Metoda 3 : INSERT z zapytania

```sql
INSERT INTO users_aggregated
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS user_count
FROM users
GROUP BY DATE_TRUNC('month', created_at);
```

### Obsługiwane formaty

- **CSV** : Pliki CSV
- **JSON** : Pliki JSON
- **Parquet** : Format zoptymalizowany (zalecane)
- **Avro** : Format Avro

---

## Zaawansowane zapytania SQL

### Funkcje analityczne

**Funkcje okna :**

```sql
-- ROW_NUMBER
SELECT 
    id,
    name,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY created_at) AS rank
FROM products;

-- LAG/LEAD
SELECT 
    date,
    sales,
    LAG(sales, 1) OVER (ORDER BY date) AS previous_sales,
    LEAD(sales, 1) OVER (ORDER BY date) AS next_sales
FROM daily_sales;

-- RANK
SELECT 
    user_id,
    total_spent,
    RANK() OVER (ORDER BY total_spent DESC) AS spending_rank
FROM user_totals;
```

### Złożone agregacje

```sql
-- GROUP BY z ROLLUP
SELECT 
    category,
    region,
    SUM(amount) AS total
FROM sales
GROUP BY ROLLUP(category, region);

-- GROUP BY z CUBE
SELECT 
    category,
    region,
    SUM(amount) AS total
FROM sales
GROUP BY CUBE(category, region);
```

### Zoptymalizowane złączenia

```sql
-- Złączenie z kluczem dystrybucji
SELECT 
    u.name,
    o.amount,
    o.created_at
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';
```

---

## Optymalizacja

### Klucze dystrybucji

**Wybrać odpowiedni klucz dystrybucji :**

```sql
-- Dystrybucja według klucza (dla złączeń)
CREATE TABLE users (
    id INTEGER DISTKEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Dystrybucja ALL (dla małych tabel)
CREATE TABLE categories (
    id INTEGER,
    name VARCHAR(100)
) DISTSTYLE ALL;

-- Dystrybucja EVEN (domyślnie)
CREATE TABLE logs (
    id INTEGER,
    message TEXT
) DISTSTYLE EVEN;
```

### Klucze sortowania

**Poprawić wydajność zapytań :**

```sql
-- Prosty klucz sortowania
CREATE TABLE orders (
    id INTEGER,
    user_id INTEGER,
    created_at TIMESTAMP,
    amount DECIMAL(10,2)
) SORTKEY (created_at);

-- Złożony klucz sortowania
CREATE TABLE sales (
    date DATE,
    region VARCHAR(50),
    amount DECIMAL(10,2)
) SORTKEY (date, region);
```

### Kompresja

**Zmniejszyć przestrzeń przechowywania :**

```sql
-- Automatyczna kompresja
CREATE TABLE users (
    id INTEGER,
    name VARCHAR(100) ENCODE lzo,
    email VARCHAR(100) ENCODE lzo,
    created_at TIMESTAMP ENCODE delta
);
```

### ANALYZE

**Aktualizować statystyki :**

```sql
-- Analizować tabelę
ANALYZE users;

-- Analizować wszystkie tabele
ANALYZE;
```

---

## Integracja z innymi usługami

### Redshift + S3

**Unload do S3 :**

```sql
UNLOAD ('SELECT * FROM users WHERE created_at > ''2024-01-01''')
TO 's3://my-bucket/exports/users/'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
PARALLEL OFF;
```

### Redshift + Glue

**Glue może ładować do Redshift :**

```python
# W jobie Glue
glueContext.write_dynamic_frame.from_jdbc_conf(
    frame = transformed_data,
    catalog_connection = "redshift-connection",
    connection_options = {
        "dbtable": "users",
        "database": "analytics"
    }
)
```

### Redshift + QuickSight

**Połączyć QuickSight z Redshift :**

1. QuickSight → "Data sources"
2. "Redshift"
3. Wprowadzić informacje połączenia
4. Wybrać tabele
5. Tworzyć wizualizacje

---

## Dobre praktyki

### Wydajność

1. **Używać COPY** zamiast INSERT dla dużych ilości
2. **Wybrać odpowiednie klucze dystrybucji**
3. **Używać kluczy sortowania** dla częstych zapytań
4. **Kompresować kolumny** aby oszczędzić przestrzeń
5. **VACUUM regularnie** aby zoptymalizować

### Koszty

1. **Zatrzymać klaster** gdy nieużywany
2. **Używać odpowiedniego typu węzła** według potrzeb
3. **Monitorować użycie** przechowywania
4. **Czyścić dane** niepotrzebne

### Bezpieczeństwo

1. **Szyfrować dane** w tranzycie i w spoczynku
2. **Używać VPC** aby izolować klaster
3. **Ograniczać dostęp** z grupami bezpieczeństwa
4. **Audytować dostęp** z CloudTrail

---

## Przykłady praktyczne

### Przykład 1 : Kompletny pipeline S3 → Redshift

```sql
-- 1. Utworzyć tabelę
CREATE TABLE sales (
    id INTEGER,
    product_id INTEGER,
    amount DECIMAL(10,2),
    sale_date DATE
) DISTKEY(product_id) SORTKEY(sale_date);

-- 2. Ładować z S3
COPY sales
FROM 's3://my-bucket/data/sales/'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
IGNOREHEADER 1;

-- 3. Analizować
ANALYZE sales;

-- 4. Zapytania analityczne
SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total_sales
FROM sales
GROUP BY DATE_TRUNC('month', sale_date)
ORDER BY month;
```

### Przykład 2 : Agregacje z funkcjami okna

```sql
-- Top 10 produktów na miesiąc
SELECT 
    product_id,
    month,
    total_sales,
    RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) AS rank
FROM (
    SELECT 
        product_id,
        DATE_TRUNC('month', sale_date) AS month,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY product_id, DATE_TRUNC('month', sale_date)
) monthly_sales
WHERE RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) <= 10;
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Redshift = Data warehouse** dla analytics
2. **Free Tier : 2 miesiące** darmowe (750 godzin)
3. **COPY z S3** = najszybsza metoda
4. **Klucze dystrybucji i sortowania** = klucze wydajności
5. **Zatrzymać klaster** gdy nieużywany

## 🔗 Następny moduł

Przejdź do modułu [5. Amazon Athena - Zapytania SQL na S3](../05-athena/README.md), aby nauczyć się bezpośrednio odpytywać pliki S3.

