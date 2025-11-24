# 5. Amazon Athena - Requêtes SQL sur S3

## 🎯 Objectifs

- Comprendre Amazon Athena et son utilisation
- Créer des tables externes pointant vers S3
- Exécuter des requêtes SQL sur fichiers S3
- Optimiser les coûts et performances
- Intégrer avec Glue Data Catalog

## 📋 Table des matières

1. [Introduction à Athena](#introduction-à-athena)
2. [Créer des tables externes](#créer-des-tables-externes)
3. [Exécuter des requêtes](#exécuter-des-requêtes)
4. [Optimisation des coûts](#optimisation-des-coûts)
5. [Intégration avec Glue](#intégration-avec-glue)
6. [Bonnes pratiques](#bonnes-pratiques)

---

## Introduction à Athena

### Qu'est-ce qu'Amazon Athena ?

**Amazon Athena** = Service de requêtes SQL serverless sur S3

- **Serverless** : Pas d'infrastructure à gérer
- **Pay-per-query** : Payez seulement ce que vous utilisez
- **Standard SQL** : Syntaxe SQL standard
- **S3 directement** : Pas besoin de charger dans base de données

### Cas d'usage pour Data Analyst

- **Exploration de données** : Analyser rapidement des fichiers S3
- **Data Lake queries** : Requêtes sur data lake
- **Ad-hoc analysis** : Analyses ponctuelles
- **Log analysis** : Analyser des logs stockés dans S3

### Free Tier Athena

**Gratuit à vie :**
- 10 Go de données scannées/mois
- Au-delà : 5$ par Téraoctet scanné

**⚠️ Important :** Les coûts dépendent de la quantité de données scannées. Optimiser les requêtes pour réduire les coûts.

---

## Créer des tables externes

### Méthode 1 : Via l'éditeur Athena

**Étape 1 : Accéder à Athena**

1. Console AWS → Rechercher "Athena"
2. Cliquer sur "Amazon Athena"
3. Première utilisation : Configurer le résultat S3

**Étape 2 : Configurer le résultat**

1. "Settings" → "Manage"
2. "Query result location" : `s3://my-bucket/athena-results/`
3. "Save"

**Étape 3 : Créer une table**

```sql
-- Table pour fichiers CSV
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

### Méthode 2 : Via Glue Data Catalog (recommandé)

**Utiliser les tables créées par Glue :**

1. Glue → Créer un crawler pour S3
2. Crawler crée automatiquement la table
3. Athena utilise directement cette table

**Avantages :**
- Schéma détecté automatiquement
- Pas besoin de définir manuellement
- Réutilisable par d'autres services

### Formats supportés

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

**Parquet (recommandé) :**
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

## Exécuter des requêtes

### Requêtes de base

**SELECT simple :**

```sql
SELECT * FROM users LIMIT 10;
```

**Filtrer :**

```sql
SELECT 
    id,
    name,
    email
FROM users
WHERE created_at > DATE '2024-01-01'
ORDER BY created_at DESC;
```

**Agrégations :**

```sql
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS user_count,
    COUNT(DISTINCT email) AS unique_emails
FROM users
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;
```

### Requêtes avancées

**Window functions :**

```sql
SELECT 
    id,
    name,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY DATE_TRUNC('month', created_at) ORDER BY created_at) AS rank
FROM users;
```

**Jointures :**

```sql
SELECT 
    u.name,
    o.amount,
    o.created_at
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.created_at > DATE '2024-01-01';
```

### Requêtes sur partitions

**Si données partitionnées :**

```sql
-- Table partitionnée par date
CREATE EXTERNAL TABLE sales (
    id INT,
    product_id INT,
    amount DECIMAL(10,2)
)
PARTITIONED BY (sale_date DATE)
STORED AS PARQUET
LOCATION 's3://bucket/sales/';

-- Ajouter des partitions
ALTER TABLE sales ADD PARTITION (sale_date='2024-01-01')
LOCATION 's3://bucket/sales/year=2024/month=01/day=01/';

-- Requête avec partition (plus rapide et moins cher)
SELECT * FROM sales
WHERE sale_date = DATE '2024-01-01';
```

---

## Optimisation des coûts

### Réduire les données scannées

**1. Utiliser WHERE pour filtrer tôt :**

```sql
-- ❌ Mauvais : Scanne tout puis filtre
SELECT * FROM large_table
WHERE date = '2024-01-01';

-- ✅ Bon : Filtre dès le début (si partitionné)
SELECT * FROM large_table
WHERE date = '2024-01-01';
```

**2. Sélectionner uniquement les colonnes nécessaires :**

```sql
-- ❌ Mauvais : Scanne toutes les colonnes
SELECT * FROM large_table;

-- ✅ Bon : Scanne seulement les colonnes nécessaires
SELECT id, name FROM large_table;
```

**3. Utiliser LIMIT :**

```sql
-- Limiter le nombre de résultats
SELECT * FROM large_table LIMIT 100;
```

### Utiliser Parquet

**Parquet est plus efficace que CSV :**

- **Compression** : Moins de données scannées
- **Colonnes** : Scanne seulement les colonnes nécessaires
- **Coût réduit** : Jusqu'à 90% de réduction

**Convertir CSV → Parquet avec Glue :**

```python
# Job Glue pour convertir
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

### Partitionner les données

**Partitionner par date (recommandé) :**

```
s3://bucket/data/
├── year=2024/
│   ├── month=01/
│   │   ├── day=01/
│   │   └── day=02/
│   └── month=02/
```

**Créer table partitionnée :**

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

## Intégration avec Glue

### Utiliser les tables Glue

**Tables créées par Glue sont automatiquement disponibles dans Athena :**

1. Glue → Crawler crée une table
2. Athena → "Tables" → Voir toutes les tables Glue
3. Utiliser directement dans les requêtes

**Avantages :**
- Schéma automatique
- Pas de définition manuelle
- Synchronisation automatique

### Mettre à jour les partitions

**Si nouvelles données ajoutées :**

```sql
-- Mettre à jour les partitions
MSCK REPAIR TABLE sales;

-- Ou ajouter manuellement
ALTER TABLE sales ADD PARTITION (sale_date='2024-01-02')
LOCATION 's3://bucket/sales/year=2024/month=01/day=02/';
```

---

## Bonnes pratiques

### Performance

1. **Utiliser Parquet** au lieu de CSV
2. **Partitionner les données** par date/catégorie
3. **Sélectionner uniquement les colonnes** nécessaires
4. **Filtrer tôt** avec WHERE
5. **Utiliser LIMIT** pour exploration

### Coûts

1. **Surveiller les données scannées** dans les résultats
2. **Optimiser les requêtes** pour réduire le scan
3. **Utiliser Parquet** pour compression
4. **Partitionner** pour réduire le scan
5. **Mettre en cache** les résultats fréquents

### Organisation

1. **Organiser S3** avec préfixes cohérents
2. **Nommer les tables** de manière claire
3. **Documenter les schémas**
4. **Utiliser des bases de données** pour organiser

---

## Exemples pratiques

### Exemple 1 : Analyser des logs

```sql
-- Table pour logs
CREATE EXTERNAL TABLE logs (
    timestamp TIMESTAMP,
    level STRING,
    message STRING,
    user_id INT
)
PARTITIONED BY (date DATE)
STORED AS TEXTFILE
LOCATION 's3://bucket/logs/';

-- Requête : Erreurs par jour
SELECT 
    date,
    COUNT(*) AS error_count
FROM logs
WHERE level = 'ERROR'
GROUP BY date
ORDER BY date DESC;
```

### Exemple 2 : Analyser des données CSV

```sql
-- Table CSV
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

-- Analyse : Ventes par mois
SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total_sales,
    COUNT(*) AS transaction_count
FROM sales_csv
GROUP BY DATE_TRUNC('month', sale_date)
ORDER BY month;
```

### Exemple 3 : Jointure de plusieurs tables

```sql
-- Analyser avec jointures
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

## 📊 Points clés à retenir

1. **Athena = SQL serverless** sur fichiers S3
2. **Free Tier : 10 Go/mois** de données scannées
3. **Parquet** = format le plus efficace
4. **Partitionner** = réduire les coûts
5. **Intégration Glue** = schémas automatiques

## 🔗 Prochain module

Passer au module [6. AWS Lambda - Serverless Computing](../06-lambda/README.md) pour apprendre à automatiser le traitement de données.

