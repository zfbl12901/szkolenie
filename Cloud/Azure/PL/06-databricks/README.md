# 6. Azure Databricks - Analiza Big Data

## 🎯 Cele

- Zrozumieć Azure Databricks
- Utworzyć obszar roboczy Databricks
- Używać notebooków Python/SQL
- Przetwarzać dane ze Spark
- Integrować z innymi usługami Azure

## 📋 Spis treści

1. [Wprowadzenie do Databricks](#wprowadzenie-do-databricks)
2. [Utworzyć obszar roboczy Databricks](#utworzyć-obszar-roboczy-databricks)
3. [Utworzyć klaster](#utworzyć-klaster)
4. [Notebooki Python/SQL](#notebooki-pythonsql)
5. [Przetwarzanie danych ze Spark](#przetwarzanie-danych-ze-spark)
6. [Integracja z innymi usługami](#integracja-z-innymi-usługami)

---

## Wprowadzenie do Databricks

### Czym jest Azure Databricks?

**Azure Databricks** = Platforma Big Data oparta na Apache Spark

- **Apache Spark** : Silnik przetwarzania rozproszonego
- **Notebooki** : Python, SQL, Scala, R
- **Zarządzane** : Microsoft zarządza infrastrukturą
- **Skalowalne** : Klastry auto-scaling

### Przypadki użycia dla Data Analyst

- **Przetwarzanie Big Data** : Przetwarzać duże ilości
- **ETL** : Złożone przekształcenia
- **Machine Learning** : Zintegrowany MLlib
- **Data Science** : Interaktywne notebooki

### Databricks Free Tier

**Darmowe z kredytem Azure :**
- Używać 200$ darmowego kredytu (30 dni)
- Po tym : normalne rozliczanie

**⚠️ Ważne :** Databricks może być kosztowne. Monitorować koszty uważnie.

---

## Utworzyć obszar roboczy Databricks

### Krok 1 : Dostęp do Databricks

1. Portal Azure → Szukać "Azure Databricks"
2. Kliknąć "Azure Databricks"
3. Kliknąć "Create"

### Krok 2 : Podstawowa konfiguracja

**Podstawowe informacje :**
- **Subscription** : Wybrać subskrypcję
- **Resource group** : Utworzyć lub użyć istniejącego
- **Workspace name** : `my-databricks-workspace`
- **Region** : Wybrać region
- **Pricing tier** : Standard (lub Premium)

**Networking :**
- **Virtual network** : Utworzyć nowy lub użyć istniejącego
- **Public IP** : ✅ Włączyć (dla łatwego dostępu)

### Krok 3 : Utworzyć obszar roboczy

1. Kliknąć "Review + create"
2. Sprawdzić konfigurację
3. Kliknąć "Create"
4. Czekać na utworzenie (5-10 minut)

**⚠️ Ważne :** Zanotować URL obszaru roboczego.

### Krok 4 : Otworzyć Databricks

1. Po utworzeniu, kliknąć "Launch Workspace"
2. Interfejs web Databricks
3. Zalogować się z Azure AD

---

## Utworzyć klaster

### Krok 1 : Dostęp do klastrów

1. Databricks Workspace → "Compute"
2. Kliknąć "Create Cluster"

### Krok 2 : Konfiguracja klastra

**Podstawowa konfiguracja :**
- **Cluster name** : `my-cluster`
- **Cluster mode** : Standard (lub Single Node dla testów)
- **Databricks runtime version** : Latest LTS (zalecane)
- **Python version** : 3.11

**Typ węzła :**
- **Worker type** : Standard_DS3_v2 (do rozpoczęcia)
- **Driver type** : Standard_DS3_v2
- **Min workers** : 0 (aby oszczędzić)
- **Max workers** : 2 (do rozpoczęcia)

**⚠️ Ważne :** Min workers = 0 umożliwia auto-termination gdy nieaktywny.

### Krok 3 : Opcje zaawansowane

**Auto-termination :**
- ✅ Włączyć (zatrzymuje klaster po nieaktywności)
- **Terminate after** : 30 minut

**Tagi :**
- Dodać tagi do organizacji

### Krok 4 : Utworzyć klaster

1. Kliknąć "Create Cluster"
2. Czekać na uruchomienie (3-5 minut)
3. Klaster gotowy gdy status = "Running"

**⚠️ Ważne :** Klaster zużywa zasoby nawet nieaktywny. Zatrzymać gdy nieużywany.

---

## Notebooki Python/SQL

### Utworzyć notebook

**Krok 1 : Utworzyć notebook**

1. Databricks Workspace → "Workspace"
2. Klik prawy → "Create" → "Notebook"
3. Nazwa : `data-processing`
4. Język : Python (lub SQL)
5. Klaster : Dołączyć do utworzonego klastra

### Krok 2 : Używać notebook

**Komórki Python :**

```python
# Komórka 1 : Importować biblioteki
import pandas as pd
from pyspark.sql import SparkSession

# Komórka 2 : Utworzyć sesję Spark
spark = SparkSession.builder.appName("DataProcessing").getOrCreate()

# Komórka 3 : Czytać dane
df = spark.read.csv("dbfs:/FileStore/data/users.csv", header=True, inferSchema=True)

# Komórka 4 : Wyświetlić dane
df.show()

# Komórka 5 : Przekształcić
df_filtered = df.filter(df["status"] == "active")
df_filtered.show()
```

**Komórki SQL :**

```sql
-- Komórka SQL : Utworzyć widok tymczasowy
CREATE OR REPLACE TEMPORARY VIEW users AS
SELECT * FROM csv.`dbfs:/FileStore/data/users.csv`

-- Zapytanie SQL
SELECT 
    YEAR(created_at) AS year,
    COUNT(*) AS user_count
FROM users
GROUP BY YEAR(created_at)
ORDER BY year;
```

### Wykonać notebook

- **Run cell** : Wykonać komórkę
- **Run all** : Wykonać wszystkie komórki
- **Run all above** : Wykonać wszystkie komórki powyżej

---

## Przetwarzanie danych ze Spark

### Czytać dane

**Z Data Lake Storage :**

```python
# Czytać CSV
df = spark.read.csv(
    "abfss://container@account.dfs.core.windows.net/data/users.csv",
    header=True,
    inferSchema=True
)

# Czytać Parquet
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/data/users.parquet"
)

# Czytać JSON
df = spark.read.json(
    "abfss://container@account.dfs.core.windows.net/data/users.json"
)
```

**Z Azure Blob Storage :**

```python
# Skonfigurować dostęp
spark.conf.set(
    "fs.azure.account.key.accountname.blob.core.windows.net",
    "your-account-key"
)

# Czytać
df = spark.read.csv(
    "wasbs://container@accountname.blob.core.windows.net/data/users.csv",
    header=True
)
```

### Przekształcać dane

**Filtrować :**

```python
df_filtered = df.filter(df["age"] > 18)
```

**Wybierać kolumny :**

```python
df_selected = df.select("id", "name", "email")
```

**Agregacje :**

```python
df_aggregated = df.groupBy("category").agg({
    "amount": "sum",
    "id": "count"
})
```

**Łączyć :**

```python
df_joined = df1.join(df2, df1.id == df2.user_id, "inner")
```

### Zapisywać dane

**Do Data Lake Storage :**

```python
# Zapisać w Parquet
df.write.mode("overwrite").parquet(
    "abfss://container@account.dfs.core.windows.net/processed/users.parquet"
)

# Zapisać w CSV
df.write.mode("overwrite").csv(
    "abfss://container@account.dfs.core.windows.net/processed/users.csv"
)
```

---

## Integracja z innymi usługami

### Databricks + Data Lake Storage

**Bezpośredni dostęp :**

```python
# Skonfigurować dostęp
spark.conf.set(
    "fs.azure.account.auth.type.account.dfs.core.windows.net",
    "OAuth"
)
spark.conf.set(
    "fs.azure.account.oauth.provider.type.account.dfs.core.windows.net",
    "org.apache.hadoop.fs.azurebfs.oauth2.MsiTokenProvider"
)

# Czytać
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/data/users.parquet"
)
```

### Databricks + Azure SQL Database

**Czytać z SQL Database :**

```python
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:sqlserver://server.database.windows.net:1433;database=db") \
    .option("dbtable", "users") \
    .option("user", "sqladmin") \
    .option("password", "password") \
    .load()
```

**Zapisać do SQL Database :**

```python
df.write \
    .format("jdbc") \
    .option("url", "jdbc:sqlserver://server.database.windows.net:1433;database=db") \
    .option("dbtable", "users_processed") \
    .option("user", "sqladmin") \
    .option("password", "password") \
    .mode("overwrite") \
    .save()
```

### Databricks + Data Factory

**Pipeline Data Factory :**
1. Źródło : Azure Blob Storage
2. Działanie : Databricks Notebook
3. Sink : Azure SQL Database

**Konfiguracja :**
- Notebook path : `/Workspace/path/to/notebook`
- Parameters : Przekazać parametry

---

## Dobre praktyki

### Wydajność

1. **Używać cache** aby ponownie używać danych
2. **Partycjonować** dane aby poprawić wydajność
3. **Optymalizować przekształcenia** aby zmniejszyć czas
4. **Używać odpowiedniej liczby workers**

### Koszty

1. **Zatrzymywać klastry** gdy nieużywane
2. **Używać auto-termination** aby oszczędzić
3. **Monitorować koszty** w Azure Cost Management
4. **Używać mniejszych klastrów** do rozpoczęcia

### Organizacja

1. **Organizować notebooki** w folderach
2. **Nazywać jasno** notebooki i klastry
3. **Dokumentować** kod
4. **Wersjonować** z Git

### Bezpieczeństwo

1. **Używać Azure AD** do uwierzytelniania
2. **Ograniczać dostęp** z RBAC
3. **Szyfrować dane** w tranzycie i w spoczynku
4. **Audytować** dostęp

---

## Przykłady praktyczne

### Przykład 1 : Kompletny pipeline ETL

**Notebook Databricks :**

```python
# 1. Czytać z Data Lake
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/raw/users.parquet"
)

# 2. Przekształcić
df_processed = df \
    .filter(df["status"] == "active") \
    .select("id", "name", "email", "created_at") \
    .withColumn("year", year(col("created_at")))

# 3. Zapisać do Data Lake
df_processed.write.mode("overwrite").parquet(
    "abfss://container@account.dfs.core.windows.net/processed/users.parquet"
)
```

### Przykład 2 : Analiza ze Spark SQL

```python
# Utworzyć widok tymczasowy
df.createOrReplaceTempView("users")

# Zapytanie SQL
result = spark.sql("""
    SELECT 
        YEAR(created_at) AS year,
        COUNT(*) AS user_count,
        COUNT(DISTINCT email) AS unique_emails
    FROM users
    GROUP BY YEAR(created_at)
    ORDER BY year
""")

result.show()
```

### Przykład 3 : Integracja z Data Factory

1. Utworzyć notebook Databricks
2. W Data Factory, dodać działanie "Databricks Notebook"
3. Skonfigurować notebook
4. Wykonać pipeline

---

## 📊 Kluczowe punkty do zapamiętania

1. **Databricks = Big Data** z Apache Spark
2. **Notebooki** Python/SQL do rozwoju
3. **Klastry auto-scaling** dla wydajności
4. **Natywna integracja** z usługami Azure
5. **Płatne** : Monitorować koszty

## 🔗 Następny moduł

Przejdź do modułu [7. Projekty praktyczne](../07-projets/README.md), aby tworzyć kompletne projekty z Azure.

