# 3. AWS Glue - ETL Serverless

## 🎯 Cele

- Zrozumieć AWS Glue i jego rolę w ETL
- Tworzyć crawlery do odkrywania danych
- Tworzyć joby ETL z Glue
- Przekształcać dane z PySpark
- Integrować Glue z S3 i innymi usługami

## 📋 Spis treści

1. [Wprowadzenie do AWS Glue](#wprowadzenie-do-aws-glue)
2. [Utworzyć Data Catalog](#utworzyć-data-catalog)
3. [Crawlery - Odkrywać dane](#crawlery---odkrywać-dane)
4. [Utworzyć job ETL](#utworzyć-job-etl)
5. [Przekształcanie danych](#przekształcanie-danych)
6. [Orkiestracja i planowanie](#orkiestracja-i-planowanie)

---

## Wprowadzenie do AWS Glue

### Czym jest AWS Glue?

**AWS Glue** = Zarządzana usługa ETL serverless

- **ETL** : Extract, Transform, Load
- **Serverless** : Brak serwerów do zarządzania
- **Zarządzane** : AWS zarządza infrastrukturą
- **Skalowalne** : Automatycznie dostosowuje się

### Komponenty Glue

1. **Data Catalog** : Katalog metadanych
2. **Crawlery** : Automatycznie odkrywają schematy
3. **ETL Jobs** : Skrypty przekształcania (Python/PySpark)
4. **Triggers** : Automatyczne wyzwalanie
5. **Workflows** : Orkiestracja wielu jobów

### Free Tier Glue

**Darmowe na zawsze :**
- 10 000 obiektów/miesiąc w Data Catalog
- 1 milion zapytań/miesiąc do Data Catalog
- 0.44$ za DPU-godzinę (pierwszy milion darmowy)

**⚠️ Ważne :** Joby Glue zużywają DPU (Data Processing Units). Monitorować koszty.

---

## Utworzyć Data Catalog

### Czym jest Data Catalog?

**Data Catalog** = Scentralizowany katalog metadanych

- Schematy danych
- Lokalizacje (S3, bazy danych)
- Typy danych
- Partycje

### Struktura Data Catalog

- **Databases** : Grupy tabel
- **Tables** : Metadane danych
- **Partitions** : Organizacja danych

### Utworzyć bazę danych

1. Konsola AWS → Glue → "Databases"
2. "Add database"
3. Nazwa : `data_analyst_db`
4. Opis (opcjonalne)
5. "Create"

**Użycie :**
- Organizować tabele według projektu
- Przykład : `raw_data_db`, `processed_data_db`

---

## Crawlery - Odkrywać dane

### Czym jest Crawler?

**Crawler** = Usługa skanująca dane i automatycznie tworząca tabele

- Analizuje pliki w S3
- Automatycznie wykrywa schemat
- Tworzy tabele w Data Catalog
- Obsługuje : CSV, JSON, Parquet, etc.

### Utworzyć Crawler

**Krok 1 : Podstawowa konfiguracja**

1. Glue → "Crawlers" → "Add crawler"
2. Nazwa : `s3-csv-crawler`
3. Opis (opcjonalne)

**Krok 2 : Źródło danych**

1. "Add a data source"
2. Typ : "S3"
3. Ścieżka S3 : `s3://my-bucket/raw/`
4. Uwzględnić podfoldery (opcjonalne)

**Krok 3 : Rola IAM**

1. Utworzyć nową rolę lub użyć istniejącej
2. Nazwa : `AWSGlueServiceRole-default`
3. Uprawnienia : Dostęp S3 i Glue

**Krok 4 : Wyjście**

1. Baza danych : `data_analyst_db`
2. Prefiks tabel (opcjonalne)

**Krok 5 : Wykonać**

1. "Run crawler now" lub zaplanować
2. Czekać na zakończenie (kilka minut)
3. Sprawdzić utworzone tabele

### Wynik Crawlera

**Automatycznie utworzona tabela :**
- Wykryte kolumny
- Wnioskowane typy danych
- Lokalizacja S3
- Format pliku

**Przykład utworzonej tabeli :**
```
Table: raw_data
Columns:
  - id (bigint)
  - name (string)
  - created_at (timestamp)
Location: s3://my-bucket/raw/
Format: csv
```

---

## Utworzyć job ETL

### Typy jobów Glue

1. **Spark** : Joby PySpark (zalecane)
2. **Python shell** : Proste skrypty Python
3. **Ray** : Zaawansowane przetwarzanie rozproszone

### Utworzyć job Spark

**Krok 1 : Konfiguracja**

1. Glue → "ETL jobs" → "Add job"
2. Nazwa : `transform-csv-job`
3. Rola IAM : `AWSGlueServiceRole-default`
4. Typ : "Spark"
5. Wersja Glue : "4.0" (zalecane)
6. DPU : 2 (minimum, regulowane)

**Krok 2 : Źródło danych**

1. "Data source" : Wybrać tabelę z Data Catalog
2. Lub : Bezpośrednia ścieżka S3

**Krok 3 : Destynacja**

1. "Data target" : S3
2. Format : Parquet (zalecane dla analytics)
3. Ścieżka : `s3://my-bucket/processed/`

**Krok 4 : Skrypt**

1. Wygenerować skrypt automatyczny
2. Lub : Napisać skrypt niestandardowy

### Podstawowy skrypt ETL

**Automatycznie wygenerowany skrypt :**

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Czytać z Data Catalog
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_data"
)

# Przekształcić (przykład : filtrować)
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# Zapisać do S3
glueContext.write_dynamic_frame.from_options(
    frame = filtered,
    connection_type = "s3",
    connection_options = {"path": "s3://my-bucket/processed/"},
    format = "parquet"
)

job.commit()
```

---

## Przekształcanie danych

### Częste przekształcenia

#### 1. Filtrować wiersze

```python
from awsglue.transforms import Filter

filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["age"] > 18
)
```

#### 2. Wybierać kolumny

```python
from awsglue.transforms import SelectFields

selected = SelectFields.apply(
    frame = datasource,
    paths = ["id", "name", "email"]
)
```

#### 3. Zmieniać nazwy kolumn

```python
from awsglue.transforms import RenameField

renamed = RenameField.apply(
    frame = datasource,
    old_name = "old_column",
    new_name = "new_column"
)
```

#### 4. Łączyć dane

```python
joined = Join.apply(
    frame1 = datasource1,
    frame2 = datasource2,
    keys1 = ["id"],
    keys2 = ["user_id"]
)
```

#### 5. Agregacje

```python
# Konwertować na DataFrame Spark dla agregacji
df = datasource.toDF()

aggregated = df.groupBy("category").agg({
    "amount": "sum",
    "id": "count"
})

# Konwertować z powrotem na DynamicFrame
from awsglue.dynamicframe import DynamicFrame
result = DynamicFrame.fromDF(aggregated, glueContext, "result")
```

### Kompletny przykład : Przekształcenie CSV → Parquet

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# 1. Czytać z S3 (przez Data Catalog)
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_data"
)

# 2. Filtrować dane
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# 3. Wybierać kolumny
selected = SelectFields.apply(
    frame = filtered,
    paths = ["id", "name", "email", "created_at"]
)

# 4. Konwertować na DataFrame dla zaawansowanych przekształceń
df = selected.toDF()

# 5. Dodać kolumnę obliczoną
from pyspark.sql.functions import col, year
df = df.withColumn("year", year(col("created_at")))

# 6. Konwertować z powrotem na DynamicFrame
from awsglue.dynamicframe import DynamicFrame
result = DynamicFrame.fromDF(df, glueContext, "result")

# 7. Zapisać do S3 w Parquet (partycjonowane według roku)
glueContext.write_dynamic_frame.from_options(
    frame = result,
    connection_type = "s3",
    connection_options = {
        "path": "s3://my-bucket/processed/",
        "partitionKeys": ["year"]
    },
    format = "parquet"
)

job.commit()
```

---

## Orkiestracja i planowanie

### Wyzwolić job ręcznie

1. Glue → "ETL jobs"
2. Wybrać job
3. "Run job"
4. Zobaczyć logi w czasie rzeczywistym

### Zaplanować job (Trigger)

**Utworzyć trigger :**

1. Glue → "Triggers" → "Add trigger"
2. Nazwa : `daily-etl-trigger`
3. Typ : "Scheduled"
4. Częstotliwość : "Cron expression"
   - Przykład : `cron(0 2 * * ? *)` = Codziennie o 2h
5. Akcje : Wybrać job do wykonania
6. "Add"

**Typy triggerów :**
- **On-demand** : Ręczne wyzwalanie
- **Scheduled** : Zaplanowane (cron)
- **Event-driven** : Wyzwalane przez zdarzenie (np. nowy plik S3)

### Workflows (złożona orkiestracja)

**Utworzyć workflow :**

1. Glue → "Workflows" → "Add workflow"
2. Nazwa : `etl-pipeline-workflow`
3. Dodać kroki :
   - Crawler → Job ETL → Inny Job
4. Zdefiniować zależności
5. Wyzwolić workflow

**Przykład workflow :**
```
1. Crawler S3 → Odkrywa nowe pliki
2. Job ETL 1 → Przekształca surowe dane
3. Job ETL 2 → Agreguje dane
4. Job ETL 3 → Ładuje do Redshift
```

---

## Dobre praktyki

### Wydajność

1. **Używać Parquet** zamiast CSV (szybsze)
2. **Partycjonować dane** (poprawia wydajność)
3. **Dostosować DPU** według rozmiaru danych
4. **Używać cache Spark** do ponownego użycia danych

### Koszty

1. **Monitorować DPU-godziny** używane
2. **Optymalizować skrypty** do zmniejszenia czasu wykonania
3. **Używać odpowiednich klas S3** (Standard-IA dla archiwów)
4. **Zatrzymywać joby** które szybko kończą się niepowodzeniem

### Organizacja

1. **Nazywać joby** spójnie
2. **Dokumentować przekształcenia**
3. **Wersjonować skrypty** (Git)
4. **Testować lokalnie** przed wdrożeniem

---

## Przykłady praktyczne

### Przykład 1 : Przekształcić CSV → Parquet

```python
# Czytać CSV z S3
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_csv_data"
)

# Zapisać w Parquet
glueContext.write_dynamic_frame.from_options(
    frame = datasource,
    connection_type = "s3",
    connection_options = {"path": "s3://my-bucket/parquet/"},
    format = "parquet"
)
```

### Przykład 2 : Czyścić i walidować

```python
# Filtrować nieprawidłowe wiersze
cleaned = Filter.apply(
    frame = datasource,
    f = lambda x: x["email"] is not None and "@" in x["email"]
)

# Usunąć duplikaty (przez DataFrame)
df = cleaned.toDF()
df = df.dropDuplicates(["id"])

result = DynamicFrame.fromDF(df, glueContext, "result")
```

### Przykład 3 : Łączyć wiele źródeł

```python
# Czytać dwie tabele
users = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "users"
)

orders = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "orders"
)

# Łączyć
joined = Join.apply(
    frame1 = users,
    frame2 = orders,
    keys1 = ["id"],
    keys2 = ["user_id"]
)
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Glue = ETL serverless** zarządzane przez AWS
2. **Crawlery** automatycznie odkrywają schematy
3. **Joby ETL** używają PySpark do przekształceń
4. **Data Catalog** centralizuje metadane
5. **Triggers** umożliwiają automatyzację

## 🔗 Następny moduł

Przejdź do modułu [4. Amazon Redshift - Data Warehouse](../04-redshift/README.md), aby nauczyć się używać Redshift do analizy danych.

