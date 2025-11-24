# 4. Azure SQL Database - Baza danych

## 🎯 Cele

- Zrozumieć Azure SQL Database
- Utworzyć bazę SQL Database (darmowo do 32 GB)
- Migrować dane
- Optymalizować zapytania
- Integrować z PowerBI

## 📋 Spis treści

1. [Wprowadzenie do SQL Database](#wprowadzenie-do-sql-database)
2. [Utworzyć bazę SQL Database](#utworzyć-bazę-sql-database)
3. [Połączyć się z bazą](#połączyć-się-z-bazą)
4. [Ładować dane](#ładować-dane)
5. [Zapytania SQL](#zapytania-sql)
6. [Integracja z PowerBI](#integracja-z-powerbi)

---

## Wprowadzenie do SQL Database

### Czym jest Azure SQL Database?

**Azure SQL Database** = Zarządzana baza danych SQL w chmurze

- **Zgodne z SQL Server** : Standardowa składnia SQL
- **Zarządzane** : Microsoft zarządza infrastrukturą
- **Skalowalne** : Od kilku GB do kilku TB
- **Wysoka dostępność** : 99.99% dostępności

### Przypadki użycia dla Data Analyst

- **Data Warehouse** : Centralizować dane
- **Analytics** : Złożone zapytania
- **Business Intelligence** : Źródło dla PowerBI
- **Data Integration** : Centralny punkt dla ETL

### SQL Database Free Tier

**Darmowe 12 miesięcy :**
- **Basic tier** : Do 32 GB
- **DTU** : 5 DTU (Database Transaction Units)
- **Backup** : Automatyczny (7 dni)

**⚠️ Ważne :** Po 12 miesiącach, normalne rozliczanie. Monitorować koszty.

---

## Utworzyć bazę SQL Database

### Krok 1 : Dostęp do SQL Database

1. Portal Azure → Szukać "SQL databases"
2. Kliknąć "SQL databases"
3. Kliknąć "Create"

### Krok 2 : Podstawowa konfiguracja

**Podstawowe informacje :**
- **Subscription** : Wybrać subskrypcję
- **Resource group** : Utworzyć lub użyć istniejącego
- **Database name** : `analytics-db`
- **Server** : Utworzyć nowy serwer lub użyć istniejącego

**Utworzyć serwer SQL :**
- **Server name** : `my-sql-server-xxxxx` (unikalne globalnie)
- **Location** : Wybrać region
- **Authentication method** : SQL authentication (lub Azure AD)
- **Server admin login** : `sqladmin` (lub inny)
- **Password** : Silne hasło
- **Allow Azure services** : ✅ Tak (dla Data Factory)

### Krok 3 : Konfiguracja bazy

**Compute + storage :**
- **Service tier** : Basic (dla Free Tier)
- **Compute tier** : Serverless (lub Provisioned)
- **Storage** : 2 GB (darmowe, rozszerzalne do 32 GB)

**⚠️ Ważne :** Basic tier = 5 DTU, wystarczające do rozpoczęcia.

### Krok 4 : Konfiguracja sieci

**Networking :**
- **Public endpoint** : ✅ Włączyć
- **Firewall rules** :
  - ✅ Allow Azure services and resources
  - Dodać Twoje IP dla dostępu lokalnego

### Krok 5 : Utworzyć bazę

1. Kliknąć "Review + create"
2. Sprawdzić konfigurację
3. Kliknąć "Create"
4. Czekać na utworzenie (2-3 minuty)

**⚠️ Ważne :** Zanotować nazwę serwera i credentials.

---

## Połączyć się z bazą

### Przez portal Azure (Query Editor)

1. SQL Database → "Query editor"
2. Wprowadzić credentials
3. Wykonywać zapytania SQL

### Przez SQL Server Management Studio (SSMS)

**Pobrać SSMS :**
- https://aka.ms/ssmsfullsetup

**Połączenie :**
- **Server name** : `my-sql-server-xxxxx.database.windows.net`
- **Authentication** : SQL Server Authentication
- **Login** : `sqladmin`
- **Password** : Twoje hasło

### Przez Azure Data Studio

**Pobrać Azure Data Studio :**
- https://aka.ms/azuredatastudio

**Zalety :**
- Darmowe i open-source
- Nowoczesny interfejs
- Wsparcie notebooków
- Integracja Git

### Przez Python (pyodbc)

```python
import pyodbc

# Połączenie
server = 'my-sql-server-xxxxx.database.windows.net'
database = 'analytics-db'
username = 'sqladmin'
password = 'your-password'
driver = '{ODBC Driver 17 for SQL Server}'

conn = pyodbc.connect(
    f'DRIVER={driver};SERVER={server};DATABASE={database};UID={username};PWD={password}'
)

# Wykonać zapytanie
cursor = conn.cursor()
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()
for row in rows:
    print(row)
```

---

## Ładować dane

### Metoda 1 : INSERT (małe ilości)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Metoda 2 : BULK INSERT z Blob Storage

**Wymagania wstępne :**
- Utworzyć klucz SAS dla Blob Storage
- Utworzyć credential w SQL Database

**Przykład :**

```sql
-- Utworzyć credential
CREATE DATABASE SCOPED CREDENTIAL BlobCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
SECRET = 'your-sas-token';

-- Utworzyć zewnętrzne źródło danych
CREATE EXTERNAL DATA SOURCE BlobStorage
WITH (
    TYPE = BLOB_STORAGE,
    LOCATION = 'https://mystorageaccount.blob.core.windows.net',
    CREDENTIAL = BlobCredential
);

-- Importować z Blob Storage
BULK INSERT users
FROM 'raw-data/users.csv'
WITH (
    DATA_SOURCE = 'BlobStorage',
    FORMAT = 'CSV',
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
);
```

### Metoda 3 : Przez Data Factory

**Pipeline :**
1. Źródło : Azure Blob Storage (CSV)
2. Działanie : Copy Data
3. Sink : Azure SQL Database

**Konfiguracja :**
- Źródło : `raw-data/users.csv`
- Sink : Tabela `users` w SQL Database
- Mapowanie : Kolumny automatyczne lub ręczne

### Metoda 4 : Przez Python (pandas)

```python
import pandas as pd
import pyodbc

# Czytać plik CSV
df = pd.read_csv('users.csv')

# Połączenie
conn = pyodbc.connect(connection_string)

# Zapisać w SQL Database
df.to_sql('users', conn, if_exists='append', index=False)
```

---

## Zapytania SQL

### Podstawowe zapytania

**Prosty SELECT :**

```sql
SELECT * FROM users LIMIT 10;
```

**Filtrować :**

```sql
SELECT id, name, email
FROM users
WHERE created_at > '2024-01-01'
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
WHERE o.created_at > '2024-01-01';
```

**CTE (Common Table Expressions) :**

```sql
WITH monthly_users AS (
    SELECT 
        DATE_TRUNC('month', created_at) AS month,
        COUNT(*) AS user_count
    FROM users
    GROUP BY DATE_TRUNC('month', created_at)
)
SELECT 
    month,
    user_count,
    LAG(user_count, 1) OVER (ORDER BY month) AS previous_month
FROM monthly_users;
```

---

## Integracja z PowerBI

### Bezpośrednie połączenie

**Krok 1 : W PowerBI Desktop**

1. "Get Data" → "Azure" → "Azure SQL Database"
2. Wprowadzić informacje :
   - **Server** : `my-sql-server-xxxxx.database.windows.net`
   - **Database** : `analytics-db`
   - **Data connectivity mode** : Import (lub DirectQuery)

**Krok 2 : Uwierzytelnianie**

- **Authentication method** : Database
- **Username** : `sqladmin`
- **Password** : Twoje hasło

**Krok 3 : Wybrać tabele**

- Wybrać tabele do importu
- Kliknąć "Load"

### DirectQuery vs Import

**Import :**
- ✅ Szybkie dla wizualizacji
- ✅ Działa offline
- ❌ Dane statyczne (wymaga odświeżenia)

**DirectQuery :**
- ✅ Dane w czasie rzeczywistym
- ✅ Brak limitu rozmiaru
- ❌ Wolniejsze (zapytania przy każdej interakcji)

### Tworzyć wizualizacje

**Przykład :**
1. Importować tabelę `users`
2. Utworzyć wykres : Liczba użytkowników na miesiąc
3. Dodać filtry
4. Opublikować na PowerBI Service

---

## Dobre praktyki

### Wydajność

1. **Tworzyć indeksy** na często używanych kolumnach
2. **Optymalizować zapytania** z EXPLAIN
3. **Używać widoków** aby uprościć
4. **Partycjonować** duże tabele

### Koszty

1. **Monitorować użycie** w Azure Cost Management
2. **Używać Basic tier** do rozpoczęcia
3. **Zatrzymać bazę** jeśli nieużywana (Serverless)
4. **Czyścić** niepotrzebne dane

### Bezpieczeństwo

1. **Używać Azure AD** do uwierzytelniania
2. **Ograniczać dostęp** z regułami firewall
3. **Szyfrować dane** (włączone domyślnie)
4. **Audytować dostęp** z SQL Auditing

### Organizacja

1. **Nazywać jasno** tabele i kolumny
2. **Dokumentować** schematy
3. **Używać schematów** aby organizować
4. **Wersjonować** skrypty SQL (Git)

---

## Przykłady praktyczne

### Przykład 1 : Kompletny pipeline Blob → SQL Database

**Przez Data Factory :**
1. Źródło : Azure Blob Storage (CSV)
2. Działanie : Copy Data
3. Sink : Azure SQL Database
4. Trigger : Schedule (codziennie)

### Przykład 2 : Zapytania analityczne

```sql
-- Top 10 użytkowników według wydatków
SELECT TOP 10
    u.name,
    SUM(o.amount) AS total_spent,
    COUNT(o.id) AS order_count
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= DATEADD(month, -3, GETDATE())
GROUP BY u.name
ORDER BY total_spent DESC;
```

### Przykład 3 : Eksport do PowerBI

1. Utworzyć widok dla PowerBI
2. Połączyć PowerBI z widokiem
3. Tworzyć wizualizacje
4. Opublikować raport

---

## 📊 Kluczowe punkty do zapamiętania

1. **SQL Database = Baza SQL w chmurze** zarządzana przez Microsoft
2. **Free Tier : 32 GB** przez 12 miesięcy (Basic tier)
3. **Zgodne z SQL Server** : Standardowa składnia
4. **Integracja PowerBI** : Bezpośrednie połączenie
5. **Skalowalne** : Od Basic do Premium

## 🔗 Następny moduł

Przejdź do modułu [5. Azure Synapse Analytics - Hurtownia danych](../05-synapse/README.md), aby nauczyć się używać Synapse do analizy danych.

