# 7. Projekty praktyczne AWS

## 🎯 Cele

- Stosować zdobytą wiedzę
- Tworzyć kompletne pipeline'y ETL
- Budować Data Lake na AWS
- Tworzyć projekty dla portfolio
- Integrować wiele usług AWS

## 📋 Spis treści

1. [Projekt 1 : Pipeline ETL S3 → Parquet](#projekt-1--pipeline-etl-s3---parquet)
2. [Projekt 2 : Data Lake na AWS](#projekt-2--data-lake-na-aws)
3. [Projekt 3 : Analytics z Athena](#projekt-3--analytics-z-athena)
4. [Projekt 4 : Kompletny zautomatyzowany pipeline](#projekt-4--kompletny-zautomatyzowany-pipeline)
5. [Dobre praktyki dla portfolio](#dobre-praktyki-dla-portfolio)

---

## Projekt 1 : Pipeline ETL S3 → Parquet

### Cel

Utworzyć pipeline ETL który przekształca pliki CSV z S3 w zoptymalizowany format Parquet.

### Architektura

```
S3 (raw/) → Glue Crawler → Data Catalog → Glue Job → S3 (processed/parquet/)
```

### Kroki

#### 1. Przygotować dane

**Utworzyć bucket S3 :**
- Nazwa : `data-analyst-project-1`
- Utworzyć folder `raw/`
- Przesłać plik CSV testowy

**Przykład danych CSV :**
```csv
id,name,email,created_at,status
1,John Doe,john@example.com,2024-01-01,active
2,Jane Smith,jane@example.com,2024-01-02,inactive
```

#### 2. Utworzyć Crawler Glue

1. Glue → "Crawlers" → "Add crawler"
2. Nazwa : `csv-crawler`
3. Źródło danych : `s3://data-analyst-project-1/raw/`
4. Rola IAM : Utworzyć rolę z dostępem S3
5. Baza danych : `project1_db`
6. Wykonać crawler

#### 3. Utworzyć Job Glue

1. Glue → "ETL jobs" → "Add job"
2. Nazwa : `csv-to-parquet-job`
3. Typ : Spark
4. Skrypt :

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
    database = "project1_db",
    table_name = "raw_data"
)

# Filtrować aktywne dane
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# Zapisać w Parquet
glueContext.write_dynamic_frame.from_options(
    frame = filtered,
    connection_type = "s3",
    connection_options = {
        "path": "s3://data-analyst-project-1/processed/parquet/"
    },
    format = "parquet"
)

job.commit()
```

#### 4. Wykonać job

1. Wybrać job
2. "Run job"
3. Sprawdzić logi
4. Sprawdzić pliki Parquet w S3

### Wynik

- Pliki CSV przekształcone w Parquet
- Dane przefiltrowane (tylko aktywne)
- Gotowe do analytics z Athena

---

## Projekt 2 : Data Lake na AWS

### Cel

Utworzyć kompletny Data Lake z ingerencją, przekształcaniem i analytics.

### Architektura

```
Źródła → S3 (Raw) → Glue (Transform) → S3 (Processed) → Athena (Analytics)
                ↓
            Lambda (Trigger)
```

### Kroki

#### 1. Struktura S3

```
data-lake-bucket/
├── raw/
│   ├── users/
│   ├── orders/
│   └── products/
├── processed/
│   ├── users/
│   ├── orders/
│   └── products/
└── analytics/
    └── results/
```

#### 2. Crawlery dla każdego źródła

**Utworzyć 3 crawlery :**
- `users-crawler` → `s3://bucket/raw/users/`
- `orders-crawler` → `s3://bucket/raw/orders/`
- `products-crawler` → `s3://bucket/raw/products/`

#### 3. Joby ETL do przekształcania

**Job dla users :**
```python
# users-etl-job
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_lake_db",
    table_name = "users"
)

# Czyścić i przekształcać
cleaned = Filter.apply(
    frame = datasource,
    f = lambda x: x["email"] is not None
)

glueContext.write_dynamic_frame.from_options(
    frame = cleaned,
    connection_type = "s3",
    connection_options = {"path": "s3://bucket/processed/users/"},
    format = "parquet"
)
```

#### 4. Tabele Athena do analytics

```sql
-- Tabela users
CREATE EXTERNAL TABLE users_processed (
    id INT,
    name STRING,
    email STRING,
    created_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://bucket/processed/users/';

-- Tabela orders
CREATE EXTERNAL TABLE orders_processed (
    id INT,
    user_id INT,
    amount DECIMAL(10,2),
    created_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://bucket/processed/orders/';

-- Zapytanie analityczne
SELECT 
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM users_processed u
LEFT JOIN orders_processed o ON u.id = o.user_id
GROUP BY u.name
ORDER BY total_spent DESC;
```

#### 5. Automatyzacja z Lambda

**Lambda wyzwalana przez przesłanie S3 :**

```python
import boto3

glue = boto3.client('glue')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Określić który job wykonać według prefiksu
    if 'users' in key:
        job_name = 'users-etl-job'
    elif 'orders' in key:
        job_name = 'orders-etl-job'
    else:
        job_name = 'products-etl-job'
    
    # Wyzwolić job
    glue.start_job_run(JobName=job_name)
    
    return {'statusCode': 200}
```

### Wynik

- Funkcjonalny Data Lake
- Zautomatyzowany pipeline
- Analytics z Athena
- Kompletny projekt dla portfolio

---

## Projekt 3 : Analytics z Athena

### Cel

Utworzyć kompletny system analytics z zapytaniami SQL na danych S3.

### Kroki

#### 1. Przygotować dane

**Przesłać pliki Parquet do S3 :**
- `s3://analytics-bucket/sales/year=2024/month=01/`
- `s3://analytics-bucket/sales/year=2024/month=02/`

#### 2. Utworzyć tabele partycjonowane

```sql
CREATE EXTERNAL TABLE sales (
    id INT,
    product_id INT,
    amount DECIMAL(10,2),
    sale_date TIMESTAMP
)
PARTITIONED BY (year INT, month INT)
STORED AS PARQUET
LOCATION 's3://analytics-bucket/sales/';

-- Dodać partycje
ALTER TABLE sales ADD PARTITION (year=2024, month=1)
LOCATION 's3://analytics-bucket/sales/year=2024/month=01/';

ALTER TABLE sales ADD PARTITION (year=2024, month=2)
LOCATION 's3://analytics-bucket/sales/year=2024/month=02/';
```

#### 3. Zapytania analityczne

**Sprzedaż na miesiąc :**
```sql
SELECT 
    year,
    month,
    SUM(amount) AS total_sales,
    COUNT(*) AS transaction_count,
    AVG(amount) AS avg_transaction
FROM sales
WHERE year = 2024
GROUP BY year, month
ORDER BY year, month;
```

**Top produkty :**
```sql
SELECT 
    product_id,
    SUM(amount) AS total_revenue,
    COUNT(*) AS sales_count
FROM sales
WHERE year = 2024
GROUP BY product_id
ORDER BY total_revenue DESC
LIMIT 10;
```

**Trendy :**
```sql
SELECT 
    DATE_TRUNC('week', sale_date) AS week,
    SUM(amount) AS weekly_sales,
    LAG(SUM(amount), 1) OVER (ORDER BY DATE_TRUNC('week', sale_date)) AS previous_week
FROM sales
WHERE year = 2024
GROUP BY DATE_TRUNC('week', sale_date)
ORDER BY week;
```

#### 4. Zapisać wyniki

**Utworzyć tabelę dla wyników :**
```sql
CREATE EXTERNAL TABLE analytics_results (
    metric_name STRING,
    metric_value DECIMAL(10,2),
    calculated_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://analytics-bucket/results/';
```

---

## Projekt 4 : Kompletny zautomatyzowany pipeline

### Cel

Utworzyć kompletnie zautomatyzowany pipeline ETL z wieloma usługami AWS.

### Kompletna architektura

```
Plik CSV przesłany → S3 (raw/)
    ↓ (Event)
Lambda (Walidacja)
    ↓
S3 (validated/)
    ↓ (Event)
Glue Job (Przekształć CSV → Parquet)
    ↓
S3 (processed/parquet/)
    ↓
Glue Crawler (Aktualizuj Catalog)
    ↓
Athena (Analytics)
    ↓
S3 (results/)
```

### Implementacja

#### 1. Lambda walidacji

```python
import boto3
import csv

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Pobrać i walidować
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    reader = csv.DictReader(content.splitlines())
    
    valid_rows = []
    for row in reader:
        if row.get('email') and '@' in row['email']:
            valid_rows.append(row)
    
    # Przesłać zwalidowane dane
    if valid_rows:
        validated_key = key.replace('raw/', 'validated/')
        # Konwertować w CSV i przesłać
        # ...
    
    return {'statusCode': 200}
```

#### 2. Glue Job przekształcania

```python
# Przekształć zwalidowany CSV w Parquet
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "pipeline_db",
    table_name = "validated_data"
)

# Przekształcić
transformed = Map.apply(
    frame = datasource,
    f = lambda x: {
        'id': x['id'],
        'name': x['name'].upper(),
        'email': x['email'].lower(),
        'created_at': x['created_at']
    }
)

# Zapisać w Parquet
glueContext.write_dynamic_frame.from_options(
    frame = transformed,
    connection_type = "s3",
    connection_options = {"path": "s3://bucket/processed/"},
    format = "parquet"
)
```

#### 3. Workflow Glue

**Utworzyć workflow :**
1. Wyzwalacz : Nowy plik w `validated/`
2. Akcja : Wykonać job Glue
3. Następna akcja : Aktualizować crawler

### Wynik

- Kompletnie zautomatyzowany pipeline
- Automatyczna walidacja
- Automatyczne przekształcanie
- Analytics dostępne natychmiast

---

## Dobre praktyki dla portfolio

### Dokumentacja

**Utworzyć README dla każdego projektu :**

```markdown
# Projekt : Pipeline ETL AWS

## Opis
Zautomatyzowany pipeline ETL do przekształcania danych CSV w Parquet.

## Architektura
- S3 : Przechowywanie
- Glue : Przekształcanie
- Athena : Analytics

## Wyniki
- Redukcja kosztów o 60%
- Czas przetwarzania zmniejszony o 80%
```

### Wizualizacje

**Tworzyć diagramy :**
- Architektura systemu
- Przepływ danych
- Schemat danych

**Narzędzia :**
- Draw.io
- Lucidchart
- Diagramy ASCII w README

### Metryki

**Uwzględniać metryki :**
- Czas wykonania przed/po
- Koszty przed/po
- Wolumen przetworzonych danych
- Wydajność zapytań

### Kod

**Dobre praktyki :**
- Kod skomentowany
- Zmienne środowiskowe do konfiguracji
- Obsługa błędów
- Logowanie

### GitHub

**Utworzyć repozytorium :**
- README z dokumentacją
- Skrypty Lambda
- Skrypty Glue
- Konfiguracja
- Diagramy

---

## 📊 Kluczowe punkty do zapamiętania

1. **Projekty praktyczne** : Niezbędne dla portfolio
2. **Dokumentacja** : Wyjaśniać architekturę i wyniki
3. **Metryki** : Pokazywać wpływ (wydajność, koszty)
4. **Czysty kod** : Skomentowany i zorganizowany
5. **GitHub** : Dzielić się projektami

## 🔗 Zasoby

- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Solutions](https://aws.amazon.com/solutions/)
- [GitHub AWS Examples](https://github.com/aws-samples)

---

**Gratulacje !** Ukończyłeś formację AWS dla Data Analyst. Możesz teraz tworzyć kompletne projekty na AWS używając wyłącznie darmowych zasobów.

