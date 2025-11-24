# 5. Azure Synapse Analytics - Hurtownia danych

## 🎯 Cele

- Zrozumieć Azure Synapse Analytics
- Utworzyć obszar roboczy Synapse
- Ładować dane
- Wykonywać zaawansowane zapytania SQL
- Integrować z PowerBI

## 📋 Spis treści

1. [Wprowadzenie do Synapse](#wprowadzenie-do-synapse)
2. [Utworzyć obszar roboczy Synapse](#utworzyć-obszar-roboczy-synapse)
3. [Ładować dane](#ładować-dane)
4. [Zaawansowane zapytania SQL](#zaawansowane-zapytania-sql)
5. [Integracja z PowerBI](#integracja-z-powerbi)
6. [Dobre praktyki](#dobre-praktyki)

---

## Wprowadzenie do Synapse

### Czym jest Azure Synapse Analytics?

**Azure Synapse Analytics** = Ujednolicona platforma analytics

- **Data Warehouse** : Przechowywanie i analiza danych
- **Big Data** : Przetwarzanie dużych ilości
- **SQL** : Standardowe zapytania SQL
- **Spark** : Przetwarzanie rozproszone
- **Integracja** : Ze wszystkimi usługami Azure

### Komponenty Synapse

1. **SQL Pool** : Hurtownia danych SQL (dawniej SQL Data Warehouse)
2. **Spark Pool** : Klastry Spark dla Big Data
3. **Synapse Studio** : Ujednolicony interfejs web
4. **Pipelines** : Zintegrowane ETL
5. **Notebooks** : Python, SQL, Scala

### Synapse Free Tier

**Darmowe z kredytem Azure :**
- Używać 200$ darmowego kredytu (30 dni)
- Po tym : normalne rozliczanie

**⚠️ Ważne :** Synapse może być kosztowne. Monitorować koszty uważnie.

---

## Utworzyć obszar roboczy Synapse

### Krok 1 : Dostęp do Synapse

1. Portal Azure → Szukać "Azure Synapse Analytics"
2. Kliknąć "Azure Synapse Analytics"
3. Kliknąć "Create"

### Krok 2 : Podstawowa konfiguracja

**Podstawowe informacje :**
- **Subscription** : Wybrać subskrypcję
- **Resource group** : Utworzyć lub użyć istniejącego
- **Workspace name** : `my-synapse-workspace`
- **Region** : Wybrać region
- **Data Lake Storage Gen2** : Utworzyć nowy lub użyć istniejącego

**Administrator SQL :**
- **SQL admin name** : `sqladmin`
- **Password** : Silne hasło

### Krok 3 : Konfiguracja SQL Pool

**SQL Pool :**
- **Create a SQL pool** : ✅ Tak (do rozpoczęcia)
- **Performance level** : DW100c (najtańsze)
- **Lub** : Utworzyć później (Serverless SQL)

**⚠️ Ważne :** Serverless SQL = pay-per-query, bardziej ekonomiczne do rozpoczęcia.

### Krok 4 : Utworzyć obszar roboczy

1. Kliknąć "Review + create"
2. Sprawdzić konfigurację
3. Kliknąć "Create"
4. Czekać na utworzenie (5-10 minut)

**⚠️ Ważne :** Zanotować credentials SQL.

---

## Ładować dane

### Metoda 1 : COPY z Data Lake Storage

**Najszybsze dla dużych ilości :**

```sql
-- Utworzyć tabelę
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at DATETIME2
)
WITH (
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
);

-- Ładować z Data Lake Storage
COPY INTO users
FROM 'https://mystorageaccount.dfs.core.windows.net/data-lake/raw/users.csv'
WITH (
    FILE_TYPE = 'CSV',
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
);
```

### Metoda 2 : Przez Synapse Pipelines

**Zintegrowany pipeline :**

1. Synapse Studio → "Integrate" → "Pipelines"
2. Utworzyć nowy pipeline
3. Dodać działanie "Copy Data"
4. Źródło : Azure Blob Storage lub Data Lake
5. Sink : SQL Pool
6. Wykonać pipeline

### Metoda 3 : INSERT (małe ilości)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Metoda 4 : Przez PolyBase (Tabele zewnętrzne)

**Utworzyć tabelę zewnętrzną :**

```sql
-- Utworzyć credential
CREATE DATABASE SCOPED CREDENTIAL BlobCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
SECRET = 'your-sas-token';

-- Utworzyć zewnętrzne źródło danych
CREATE EXTERNAL DATA SOURCE BlobStorage
WITH (
    TYPE = HADOOP,
    LOCATION = 'wasbs://container@account.blob.core.windows.net',
    CREDENTIAL = BlobCredential
);

-- Utworzyć zewnętrzny format pliku
CREATE EXTERNAL FILE FORMAT CSVFormat
WITH (
    FORMAT_TYPE = DELIMITEDTEXT,
    FORMAT_OPTIONS (FIELD_TERMINATOR = ',')
);

-- Utworzyć tabelę zewnętrzną
CREATE EXTERNAL TABLE users_external (
    id INT,
    name VARCHAR(100),
    email VARCHAR(100)
)
WITH (
    LOCATION = 'raw/users.csv',
    DATA_SOURCE = BlobStorage,
    FILE_FORMAT = CSVFormat
);

-- Ładować do tabeli wewnętrznej
INSERT INTO users
SELECT * FROM users_external;
```

---

## Zaawansowane zapytania SQL

### Podstawowe zapytania

**Prosty SELECT :**

```sql
SELECT TOP 100 * FROM users;
```

**Agregacje :**

```sql
SELECT 
    YEAR(created_at) AS year,
    MONTH(created_at) AS month,
    COUNT(*) AS user_count
FROM users
GROUP BY YEAR(created_at), MONTH(created_at)
ORDER BY year, month;
```

### Funkcje okna

**ROW_NUMBER :**

```sql
SELECT 
    id,
    name,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY YEAR(created_at) ORDER BY created_at) AS rank
FROM users;
```

**LAG/LEAD :**

```sql
SELECT 
    date,
    sales,
    LAG(sales, 1) OVER (ORDER BY date) AS previous_sales,
    LEAD(sales, 1) OVER (ORDER BY date) AS next_sales
FROM daily_sales;
```

### Dystrybucja i wydajność

**Klucze dystrybucji :**

```sql
-- Dystrybucja HASH (dla złączeń)
CREATE TABLE users (
    id INT,
    name VARCHAR(100)
)
WITH (
    DISTRIBUTION = HASH(id),
    CLUSTERED COLUMNSTORE INDEX
);

-- Dystrybucja ROUND_ROBIN (domyślnie)
CREATE TABLE logs (
    id INT,
    message VARCHAR(MAX)
)
WITH (
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
);
```

**Clustered Columnstore Index :**
- Zoptymalizowane dla analytics
- Wysoka kompresja
- Szybkie zapytania na dużych tabelach

---

## Integracja z PowerBI

### Bezpośrednie połączenie

**Krok 1 : W PowerBI Desktop**

1. "Get Data" → "Azure" → "Azure Synapse Analytics SQL"
2. Wprowadzić informacje :
   - **Server** : `my-synapse-workspace-ondemand.sql.azuresynapse.net` (Serverless)
   - **Database** : Nazwa bazy
   - **Data connectivity mode** : DirectQuery (zalecane)

**Krok 2 : Uwierzytelnianie**

- **Authentication method** : Database
- **Username** : `sqladmin`
- **Password** : Twoje hasło

**Krok 3 : Wybrać tabele**

- Wybrać tabele lub widoki
- Kliknąć "Load"

### Tworzyć widoki dla PowerBI

**Zoptymalizowany widok :**

```sql
CREATE VIEW vw_user_analytics AS
SELECT 
    u.id,
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name, u.email;
```

**Używać widoku w PowerBI :**
- Prostsze dla użytkowników
- Centralizowana logika biznesowa
- Zoptymalizowana wydajność

---

## Dobre praktyki

### Wydajność

1. **Używać Columnstore Index** dla analytics
2. **Wybrać odpowiednie klucze dystrybucji**
3. **Partycjonować** duże tabele
4. **Optymalizować zapytania** z EXPLAIN

### Koszty

1. **Używać Serverless SQL** do rozpoczęcia (pay-per-query)
2. **Wstrzymać SQL Pool** gdy nieużywany
3. **Monitorować koszty** w Azure Cost Management
4. **Używać odpowiednich rozmiarów** pool

### Organizacja

1. **Tworzyć schematy** aby organizować
2. **Nazywać jasno** tabele i widoki
3. **Dokumentować** schematy
4. **Używać widoków** aby uprościć

### Bezpieczeństwo

1. **Używać Azure AD** do uwierzytelniania
2. **Ograniczać dostęp** z regułami firewall
3. **Szyfrować dane** (włączone domyślnie)
4. **Audytować dostęp**

---

## Przykłady praktyczne

### Przykład 1 : Kompletny pipeline Data Lake → Synapse

**Pipeline Synapse :**
1. Źródło : Data Lake Storage (Parquet)
2. Działanie : Copy Data
3. Sink : SQL Pool
4. Trigger : Schedule (codziennie)

### Przykład 2 : Złożone zapytania analityczne

```sql
-- Analiza sprzedaży z funkcjami okna
WITH monthly_sales AS (
    SELECT 
        YEAR(sale_date) AS year,
        MONTH(sale_date) AS month,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY YEAR(sale_date), MONTH(sale_date)
)
SELECT 
    year,
    month,
    total_sales,
    LAG(total_sales, 1) OVER (ORDER BY year, month) AS previous_month,
    (total_sales - LAG(total_sales, 1) OVER (ORDER BY year, month)) / 
        LAG(total_sales, 1) OVER (ORDER BY year, month) * 100 AS growth_percent
FROM monthly_sales
ORDER BY year, month;
```

### Przykład 3 : Eksport do PowerBI

1. Utworzyć widok analityczny
2. Połączyć PowerBI z widokiem
3. Tworzyć wizualizacje
4. Opublikować raport

---

## 📊 Kluczowe punkty do zapamiętania

1. **Synapse = Ujednolicona platforma** analytics
2. **SQL Pool** dla hurtowni danych
3. **Serverless SQL** dla pay-per-query
4. **Natywna integracja PowerBI**
5. **Skalowalne** od kilku GB do kilku PB

## 🔗 Następny moduł

Przejdź do modułu [6. Azure Databricks - Analiza Big Data](../06-databricks/README.md), aby nauczyć się używać Databricks dla Big Data.

