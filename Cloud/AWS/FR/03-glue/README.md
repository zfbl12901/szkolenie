# 3. AWS Glue - ETL Serverless

## 🎯 Objectifs

- Comprendre AWS Glue et son rôle dans l'ETL
- Créer des crawlers pour découvrir les données
- Créer des jobs ETL avec Glue
- Transformer des données avec PySpark
- Intégrer Glue avec S3 et autres services

## 📋 Table des matières

1. [Introduction à AWS Glue](#introduction-à-aws-glue)
2. [Créer un Data Catalog](#créer-un-data-catalog)
3. [Crawlers - Découvrir les données](#crawlers---découvrir-les-données)
4. [Créer un job ETL](#créer-un-job-etl)
5. [Transformation de données](#transformation-de-données)
6. [Orchestration et scheduling](#orchestration-et-scheduling)

---

## Introduction à AWS Glue

### Qu'est-ce qu'AWS Glue ?

**AWS Glue** = Service ETL serverless géré

- **ETL** : Extract, Transform, Load
- **Serverless** : Pas de serveurs à gérer
- **Géré** : AWS gère l'infrastructure
- **Scalable** : S'adapte automatiquement

### Composants Glue

1. **Data Catalog** : Catalogue de métadonnées
2. **Crawlers** : Découvrent automatiquement les schémas
3. **ETL Jobs** : Scripts de transformation (Python/PySpark)
4. **Triggers** : Déclenchement automatique
5. **Workflows** : Orchestration de plusieurs jobs

### Free Tier Glue

**Gratuit à vie :**
- 10 000 objets/mois dans le Data Catalog
- 1 million requêtes/mois au Data Catalog
- 0.44$ par DPU-heure (premier million gratuit)

**⚠️ Important :** Les jobs Glue consomment des DPU (Data Processing Units). Surveiller les coûts.

---

## Créer un Data Catalog

### Qu'est-ce que le Data Catalog ?

**Data Catalog** = Catalogue centralisé de métadonnées

- Schémas de données
- Emplacements (S3, bases de données)
- Types de données
- Partitions

### Structure du Data Catalog

- **Databases** : Groupes de tables
- **Tables** : Métadonnées des données
- **Partitions** : Organisation des données

### Créer une base de données

1. Console AWS → Glue → "Databases"
2. "Add database"
3. Nom : `data_analyst_db`
4. Description (optionnel)
5. "Create"

**Utilisation :**
- Organiser les tables par projet
- Exemple : `raw_data_db`, `processed_data_db`

---

## Crawlers - Découvrir les données

### Qu'est-ce qu'un Crawler ?

**Crawler** = Service qui scanne les données et crée automatiquement les tables

- Analyse les fichiers dans S3
- Détecte le schéma automatiquement
- Crée les tables dans le Data Catalog
- Supporte : CSV, JSON, Parquet, etc.

### Créer un Crawler

**Étape 1 : Configuration de base**

1. Glue → "Crawlers" → "Add crawler"
2. Nom : `s3-csv-crawler`
3. Description (optionnel)

**Étape 2 : Source de données**

1. "Add a data source"
2. Type : "S3"
3. Chemin S3 : `s3://my-bucket/raw/`
4. Inclure les sous-dossiers (optionnel)

**Étape 3 : IAM Role**

1. Créer un nouveau rôle ou utiliser existant
2. Nom : `AWSGlueServiceRole-default`
3. Permissions : Accès S3 et Glue

**Étape 4 : Sortie**

1. Base de données : `data_analyst_db`
2. Préfixe des tables (optionnel)

**Étape 5 : Exécuter**

1. "Run crawler now" ou planifier
2. Attendre la fin (quelques minutes)
3. Vérifier les tables créées

### Résultat du Crawler

**Table créée automatiquement :**
- Colonnes détectées
- Types de données inférés
- Emplacement S3
- Format de fichier

**Exemple de table créée :**
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

## Créer un job ETL

### Types de jobs Glue

1. **Spark** : Jobs PySpark (recommandé)
2. **Python shell** : Scripts Python simples
3. **Ray** : Traitement distribué avancé

### Créer un job Spark

**Étape 1 : Configuration**

1. Glue → "ETL jobs" → "Add job"
2. Nom : `transform-csv-job`
3. IAM Role : `AWSGlueServiceRole-default`
4. Type : "Spark"
5. Glue version : "4.0" (recommandé)
6. DPU : 2 (minimum, ajustable)

**Étape 2 : Source de données**

1. "Data source" : Sélectionner une table du Data Catalog
2. Ou : Chemin S3 direct

**Étape 3 : Destination**

1. "Data target" : S3
2. Format : Parquet (recommandé pour analytics)
3. Chemin : `s3://my-bucket/processed/`

**Étape 4 : Script**

1. Générer un script automatique
2. Ou : Écrire un script personnalisé

### Script ETL de base

**Script généré automatiquement :**

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

# Lire depuis le Data Catalog
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_data"
)

# Transformer (exemple : filtrer)
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# Écrire vers S3
glueContext.write_dynamic_frame.from_options(
    frame = filtered,
    connection_type = "s3",
    connection_options = {"path": "s3://my-bucket/processed/"},
    format = "parquet"
)

job.commit()
```

---

## Transformation de données

### Transformations courantes

#### 1. Filtrer des lignes

```python
from awsglue.transforms import Filter

filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["age"] > 18
)
```

#### 2. Sélectionner des colonnes

```python
from awsglue.transforms import SelectFields

selected = SelectFields.apply(
    frame = datasource,
    paths = ["id", "name", "email"]
)
```

#### 3. Renommer des colonnes

```python
from awsglue.transforms import RenameField

renamed = RenameField.apply(
    frame = datasource,
    old_name = "old_column",
    new_name = "new_column"
)
```

#### 4. Joindre des données

```python
joined = Join.apply(
    frame1 = datasource1,
    frame2 = datasource2,
    keys1 = ["id"],
    keys2 = ["user_id"]
)
```

#### 5. Agrégations

```python
# Convertir en DataFrame Spark pour agrégations
df = datasource.toDF()

aggregated = df.groupBy("category").agg({
    "amount": "sum",
    "id": "count"
})

# Reconvertir en DynamicFrame
from awsglue.dynamicframe import DynamicFrame
result = DynamicFrame.fromDF(aggregated, glueContext, "result")
```

### Exemple complet : Transformation CSV → Parquet

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

# 1. Lire depuis S3 (via Data Catalog)
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_data"
)

# 2. Filtrer les données
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# 3. Sélectionner colonnes
selected = SelectFields.apply(
    frame = filtered,
    paths = ["id", "name", "email", "created_at"]
)

# 4. Convertir en DataFrame pour transformations avancées
df = selected.toDF()

# 5. Ajouter une colonne calculée
from pyspark.sql.functions import col, year
df = df.withColumn("year", year(col("created_at")))

# 6. Reconvertir en DynamicFrame
from awsglue.dynamicframe import DynamicFrame
result = DynamicFrame.fromDF(df, glueContext, "result")

# 7. Écrire vers S3 en Parquet (partitionné par année)
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

## Orchestration et scheduling

### Déclencher un job manuellement

1. Glue → "ETL jobs"
2. Sélectionner le job
3. "Run job"
4. Voir les logs en temps réel

### Planifier un job (Trigger)

**Créer un trigger :**

1. Glue → "Triggers" → "Add trigger"
2. Nom : `daily-etl-trigger`
3. Type : "Scheduled"
4. Fréquence : "Cron expression"
   - Exemple : `cron(0 2 * * ? *)` = Tous les jours à 2h
5. Actions : Sélectionner le job à exécuter
6. "Add"

**Types de triggers :**
- **On-demand** : Déclenchement manuel
- **Scheduled** : Planifié (cron)
- **Event-driven** : Déclenché par événement (ex: nouveau fichier S3)

### Workflows (orchestration complexe)

**Créer un workflow :**

1. Glue → "Workflows" → "Add workflow"
2. Nom : `etl-pipeline-workflow`
3. Ajouter des étapes :
   - Crawler → Job ETL → Autre Job
4. Définir les dépendances
5. Déclencher le workflow

**Exemple de workflow :**
```
1. Crawler S3 → Découvre nouveaux fichiers
2. Job ETL 1 → Transforme les données brutes
3. Job ETL 2 → Agrège les données
4. Job ETL 3 → Charge dans Redshift
```

---

## Bonnes pratiques

### Performance

1. **Utiliser Parquet** au lieu de CSV (plus rapide)
2. **Partitionner les données** (améliore les performances)
3. **Ajuster les DPU** selon la taille des données
4. **Utiliser le cache Spark** pour réutiliser des données

### Coûts

1. **Surveiller les DPU-heures** utilisées
2. **Optimiser les scripts** pour réduire le temps d'exécution
3. **Utiliser les bonnes classes S3** (Standard-IA pour archives)
4. **Arrêter les jobs** qui échouent rapidement

### Organisation

1. **Nommer les jobs** de manière cohérente
2. **Documenter les transformations**
3. **Versionner les scripts** (Git)
4. **Tester localement** avant de déployer

---

## Exemples pratiques

### Exemple 1 : Transformer CSV → Parquet

```python
# Lire CSV depuis S3
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_csv_data"
)

# Écrire en Parquet
glueContext.write_dynamic_frame.from_options(
    frame = datasource,
    connection_type = "s3",
    connection_options = {"path": "s3://my-bucket/parquet/"},
    format = "parquet"
)
```

### Exemple 2 : Nettoyer et valider

```python
# Filtrer les lignes invalides
cleaned = Filter.apply(
    frame = datasource,
    f = lambda x: x["email"] is not None and "@" in x["email"]
)

# Supprimer les doublons (via DataFrame)
df = cleaned.toDF()
df = df.dropDuplicates(["id"])

result = DynamicFrame.fromDF(df, glueContext, "result")
```

### Exemple 3 : Joindre plusieurs sources

```python
# Lire deux tables
users = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "users"
)

orders = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "orders"
)

# Joindre
joined = Join.apply(
    frame1 = users,
    frame2 = orders,
    keys1 = ["id"],
    keys2 = ["user_id"]
)
```

---

## 📊 Points clés à retenir

1. **Glue = ETL serverless** géré par AWS
2. **Crawlers** découvrent automatiquement les schémas
3. **Jobs ETL** utilisent PySpark pour transformations
4. **Data Catalog** centralise les métadonnées
5. **Triggers** permettent l'automatisation

## 🔗 Prochain module

Passer au module [4. Amazon Redshift - Data Warehouse](../04-redshift/README.md) pour apprendre à utiliser Redshift pour l'analyse de données.

