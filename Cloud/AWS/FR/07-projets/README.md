# 7. Projets pratiques AWS

## 🎯 Objectifs

- Appliquer les connaissances acquises
- Créer des pipelines ETL complets
- Construire un Data Lake sur AWS
- Créer des projets pour votre portfolio
- Intégrer plusieurs services AWS

## 📋 Table des matières

1. [Projet 1 : Pipeline ETL S3 → Parquet](#projet-1--pipeline-etl-s3---parquet)
2. [Projet 2 : Data Lake sur AWS](#projet-2--data-lake-sur-aws)
3. [Projet 3 : Analytics avec Athena](#projet-3--analytics-avec-athena)
4. [Projet 4 : Pipeline automatisé complet](#projet-4--pipeline-automatisé-complet)
5. [Bonnes pratiques pour portfolio](#bonnes-pratiques-pour-portfolio)

---

## Projet 1 : Pipeline ETL S3 → Parquet

### Objectif

Créer un pipeline ETL qui transforme des fichiers CSV depuis S3 en format Parquet optimisé.

### Architecture

```
S3 (raw/) → Glue Crawler → Data Catalog → Glue Job → S3 (processed/parquet/)
```

### Étapes

#### 1. Préparer les données

**Créer un bucket S3 :**
- Nom : `data-analyst-project-1`
- Créer un dossier `raw/`
- Uploader un fichier CSV de test

**Exemple de données CSV :**
```csv
id,name,email,created_at,status
1,John Doe,john@example.com,2024-01-01,active
2,Jane Smith,jane@example.com,2024-01-02,inactive
```

#### 2. Créer un Crawler Glue

1. Glue → "Crawlers" → "Add crawler"
2. Nom : `csv-crawler`
3. Data source : `s3://data-analyst-project-1/raw/`
4. IAM Role : Créer un rôle avec accès S3
5. Database : `project1_db`
6. Exécuter le crawler

#### 3. Créer un Job Glue

1. Glue → "ETL jobs" → "Add job"
2. Nom : `csv-to-parquet-job`
3. Type : Spark
4. Script :

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

# Lire depuis Data Catalog
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "project1_db",
    table_name = "raw_data"
)

# Filtrer les données actives
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# Écrire en Parquet
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

#### 4. Exécuter le job

1. Sélectionner le job
2. "Run job"
3. Vérifier les logs
4. Vérifier les fichiers Parquet dans S3

### Résultat

- Fichiers CSV transformés en Parquet
- Données filtrées (seulement actives)
- Prêt pour analytics avec Athena

---

## Projet 2 : Data Lake sur AWS

### Objectif

Créer un Data Lake complet avec ingestion, transformation et analytics.

### Architecture

```
Sources → S3 (Raw) → Glue (Transform) → S3 (Processed) → Athena (Analytics)
                ↓
            Lambda (Trigger)
```

### Étapes

#### 1. Structure S3

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

#### 2. Crawlers pour chaque source

**Créer 3 crawlers :**
- `users-crawler` → `s3://bucket/raw/users/`
- `orders-crawler` → `s3://bucket/raw/orders/`
- `products-crawler` → `s3://bucket/raw/products/`

#### 3. Jobs ETL pour transformation

**Job pour users :**
```python
# users-etl-job
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_lake_db",
    table_name = "users"
)

# Nettoyer et transformer
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

#### 4. Tables Athena pour analytics

```sql
-- Table users
CREATE EXTERNAL TABLE users_processed (
    id INT,
    name STRING,
    email STRING,
    created_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://bucket/processed/users/';

-- Table orders
CREATE EXTERNAL TABLE orders_processed (
    id INT,
    user_id INT,
    amount DECIMAL(10,2),
    created_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://bucket/processed/orders/';

-- Requête analytique
SELECT 
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM users_processed u
LEFT JOIN orders_processed o ON u.id = o.user_id
GROUP BY u.name
ORDER BY total_spent DESC;
```

#### 5. Automatisation avec Lambda

**Lambda déclenché par upload S3 :**

```python
import boto3

glue = boto3.client('glue')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Déterminer quel job exécuter selon le préfixe
    if 'users' in key:
        job_name = 'users-etl-job'
    elif 'orders' in key:
        job_name = 'orders-etl-job'
    else:
        job_name = 'products-etl-job'
    
    # Déclencher le job
    glue.start_job_run(JobName=job_name)
    
    return {'statusCode': 200}
```

### Résultat

- Data Lake fonctionnel
- Pipeline automatisé
- Analytics avec Athena
- Projet complet pour portfolio

---

## Projet 3 : Analytics avec Athena

### Objectif

Créer un système d'analytics complet avec requêtes SQL sur données S3.

### Étapes

#### 1. Préparer les données

**Uploader des fichiers Parquet dans S3 :**
- `s3://analytics-bucket/sales/year=2024/month=01/`
- `s3://analytics-bucket/sales/year=2024/month=02/`

#### 2. Créer des tables partitionnées

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

-- Ajouter les partitions
ALTER TABLE sales ADD PARTITION (year=2024, month=1)
LOCATION 's3://analytics-bucket/sales/year=2024/month=01/';

ALTER TABLE sales ADD PARTITION (year=2024, month=2)
LOCATION 's3://analytics-bucket/sales/year=2024/month=02/';
```

#### 3. Requêtes analytiques

**Ventes par mois :**
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

**Top produits :**
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

**Tendances :**
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

#### 4. Sauvegarder les résultats

**Créer une table pour résultats :**
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

## Projet 4 : Pipeline automatisé complet

### Objectif

Créer un pipeline ETL complètement automatisé avec plusieurs services AWS.

### Architecture complète

```
Fichier CSV uploadé → S3 (raw/)
    ↓ (Event)
Lambda (Validation)
    ↓
S3 (validated/)
    ↓ (Event)
Glue Job (Transform CSV → Parquet)
    ↓
S3 (processed/parquet/)
    ↓
Glue Crawler (Update Catalog)
    ↓
Athena (Analytics)
    ↓
S3 (results/)
```

### Implémentation

#### 1. Lambda de validation

```python
import boto3
import csv

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Télécharger et valider
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    reader = csv.DictReader(content.splitlines())
    
    valid_rows = []
    for row in reader:
        if row.get('email') and '@' in row['email']:
            valid_rows.append(row)
    
    # Uploader les données validées
    if valid_rows:
        validated_key = key.replace('raw/', 'validated/')
        # Convertir en CSV et uploader
        # ...
    
    return {'statusCode': 200}
```

#### 2. Glue Job de transformation

```python
# Transform validated CSV to Parquet
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "pipeline_db",
    table_name = "validated_data"
)

# Transformer
transformed = Map.apply(
    frame = datasource,
    f = lambda x: {
        'id': x['id'],
        'name': x['name'].upper(),
        'email': x['email'].lower(),
        'created_at': x['created_at']
    }
)

# Écrire en Parquet
glueContext.write_dynamic_frame.from_options(
    frame = transformed,
    connection_type = "s3",
    connection_options = {"path": "s3://bucket/processed/"},
    format = "parquet"
)
```

#### 3. Workflow Glue

**Créer un workflow :**
1. Trigger : Nouveau fichier dans `validated/`
2. Action : Exécuter le job Glue
3. Action suivante : Mettre à jour le crawler

### Résultat

- Pipeline complètement automatisé
- Validation automatique
- Transformation automatique
- Analytics disponibles immédiatement

---

## Bonnes pratiques pour portfolio

### Documentation

**Créer un README pour chaque projet :**

```markdown
# Projet : Pipeline ETL AWS

## Description
Pipeline ETL automatisé pour transformer des données CSV en Parquet.

## Architecture
- S3 : Stockage
- Glue : Transformation
- Athena : Analytics

## Résultats
- Réduction des coûts de 60%
- Temps de traitement réduit de 80%
```

### Visualisations

**Créer des diagrammes :**
- Architecture du système
- Flux de données
- Schéma de données

**Outils :**
- Draw.io
- Lucidchart
- Diagrammes ASCII dans README

### Métriques

**Inclure des métriques :**
- Temps d'exécution avant/après
- Coûts avant/après
- Volume de données traitées
- Performance des requêtes

### Code

**Bonnes pratiques :**
- Code commenté
- Variables d'environnement pour configuration
- Gestion d'erreurs
- Logging

### GitHub

**Créer un repository :**
- README avec documentation
- Scripts Lambda
- Scripts Glue
- Configuration
- Diagrammes

---

## 📊 Points clés à retenir

1. **Projets pratiques** : Essentiels pour portfolio
2. **Documentation** : Expliquer l'architecture et les résultats
3. **Métriques** : Montrer l'impact (performance, coûts)
4. **Code propre** : Commenté et organisé
5. **GitHub** : Partager vos projets

## 🔗 Ressources

- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Solutions](https://aws.amazon.com/solutions/)
- [GitHub AWS Examples](https://github.com/aws-samples)

---

**Félicitations !** Vous avez terminé la formation AWS pour Data Analyst. Vous pouvez maintenant créer des projets complets sur AWS en utilisant uniquement des ressources gratuites.

