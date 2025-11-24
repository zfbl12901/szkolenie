# 6. Azure Databricks - Big Data Analytics

## 🎯 Objectifs

- Comprendre Azure Databricks
- Créer un workspace Databricks
- Utiliser des notebooks Python/SQL
- Traiter des données avec Spark
- Intégrer avec autres services Azure

## 📋 Table des matières

1. [Introduction à Databricks](#introduction-à-databricks)
2. [Créer un workspace Databricks](#créer-un-workspace-databricks)
3. [Créer un cluster](#créer-un-cluster)
4. [Notebooks Python/SQL](#notebooks-pythonsql)
5. [Traitement de données avec Spark](#traitement-de-données-avec-spark)
6. [Intégration avec autres services](#intégration-avec-autres-services)

---

## Introduction à Databricks

### Qu'est-ce qu'Azure Databricks ?

**Azure Databricks** = Plateforme Big Data basée sur Apache Spark

- **Apache Spark** : Moteur de traitement distribué
- **Notebooks** : Python, SQL, Scala, R
- **Géré** : Microsoft gère l'infrastructure
- **Scalable** : Clusters auto-scaling

### Cas d'usage pour Data Analyst

- **Big Data processing** : Traiter de grandes quantités
- **ETL** : Transformations complexes
- **Machine Learning** : MLlib intégré
- **Data Science** : Notebooks interactifs

### Free Tier Databricks

**Gratuit avec crédit Azure :**
- Utiliser les 200$ de crédit gratuit (30 jours)
- Après : facturation normale

**⚠️ Important :** Databricks peut être coûteux. Surveiller attentivement les coûts.

---

## Créer un workspace Databricks

### Étape 1 : Accéder à Databricks

1. Portail Azure → Rechercher "Azure Databricks"
2. Cliquer sur "Azure Databricks"
3. Cliquer sur "Create"

### Étape 2 : Configuration de base

**Informations de base :**
- **Subscription** : Choisir votre abonnement
- **Resource group** : Créer ou utiliser existant
- **Workspace name** : `my-databricks-workspace`
- **Region** : Choisir la région
- **Pricing tier** : Standard (ou Premium)

**Networking :**
- **Virtual network** : Créer nouveau ou utiliser existant
- **Public IP** : ✅ Enable (pour accès facile)

### Étape 3 : Créer le workspace

1. Cliquer sur "Review + create"
2. Vérifier la configuration
3. Cliquer sur "Create"
4. Attendre la création (5-10 minutes)

**⚠️ Important :** Noter l'URL du workspace.

### Étape 4 : Ouvrir Databricks

1. Une fois créé, cliquer sur "Launch Workspace"
2. Interface web Databricks
3. Se connecter avec Azure AD

---

## Créer un cluster

### Étape 1 : Accéder aux clusters

1. Databricks Workspace → "Compute"
2. Cliquer sur "Create Cluster"

### Étape 2 : Configuration du cluster

**Configuration de base :**
- **Cluster name** : `my-cluster`
- **Cluster mode** : Standard (ou Single Node pour tests)
- **Databricks runtime version** : Latest LTS (recommandé)
- **Python version** : 3.11

**Node type :**
- **Worker type** : Standard_DS3_v2 (pour débuter)
- **Driver type** : Standard_DS3_v2
- **Min workers** : 0 (pour économiser)
- **Max workers** : 2 (pour débuter)

**⚠️ Important :** Min workers = 0 permet l'auto-termination quand inactif.

### Étape 3 : Options avancées

**Auto-termination :**
- ✅ Enable (arrête le cluster après inactivité)
- **Terminate after** : 30 minutes

**Tags :**
- Ajouter des tags pour organisation

### Étape 4 : Créer le cluster

1. Cliquer sur "Create Cluster"
2. Attendre le démarrage (3-5 minutes)
3. Cluster prêt quand status = "Running"

**⚠️ Important :** Le cluster consomme des ressources même inactif. L'arrêter quand non utilisé.

---

## Notebooks Python/SQL

### Créer un notebook

**Étape 1 : Créer un notebook**

1. Databricks Workspace → "Workspace"
2. Clic droit → "Create" → "Notebook"
3. Nom : `data-processing`
4. Language : Python (ou SQL)
5. Cluster : Attacher au cluster créé

### Étape 2 : Utiliser le notebook

**Cellules Python :**

```python
# Cellule 1 : Importer des bibliothèques
import pandas as pd
from pyspark.sql import SparkSession

# Cellule 2 : Créer une session Spark
spark = SparkSession.builder.appName("DataProcessing").getOrCreate()

# Cellule 3 : Lire des données
df = spark.read.csv("dbfs:/FileStore/data/users.csv", header=True, inferSchema=True)

# Cellule 4 : Afficher les données
df.show()

# Cellule 5 : Transformer
df_filtered = df.filter(df["status"] == "active")
df_filtered.show()
```

**Cellules SQL :**

```sql
-- Cellule SQL : Créer une vue temporaire
CREATE OR REPLACE TEMPORARY VIEW users AS
SELECT * FROM csv.`dbfs:/FileStore/data/users.csv`

-- Requête SQL
SELECT 
    YEAR(created_at) AS year,
    COUNT(*) AS user_count
FROM users
GROUP BY YEAR(created_at)
ORDER BY year;
```

### Exécuter un notebook

- **Run cell** : Exécuter une cellule
- **Run all** : Exécuter toutes les cellules
- **Run all above** : Exécuter toutes les cellules au-dessus

---

## Traitement de données avec Spark

### Lire des données

**Depuis Data Lake Storage :**

```python
# Lire CSV
df = spark.read.csv(
    "abfss://container@account.dfs.core.windows.net/data/users.csv",
    header=True,
    inferSchema=True
)

# Lire Parquet
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/data/users.parquet"
)

# Lire JSON
df = spark.read.json(
    "abfss://container@account.dfs.core.windows.net/data/users.json"
)
```

**Depuis Azure Blob Storage :**

```python
# Configurer l'accès
spark.conf.set(
    "fs.azure.account.key.accountname.blob.core.windows.net",
    "your-account-key"
)

# Lire
df = spark.read.csv(
    "wasbs://container@accountname.blob.core.windows.net/data/users.csv",
    header=True
)
```

### Transformer des données

**Filtrer :**

```python
df_filtered = df.filter(df["age"] > 18)
```

**Sélectionner des colonnes :**

```python
df_selected = df.select("id", "name", "email")
```

**Agrégations :**

```python
df_aggregated = df.groupBy("category").agg({
    "amount": "sum",
    "id": "count"
})
```

**Joindre :**

```python
df_joined = df1.join(df2, df1.id == df2.user_id, "inner")
```

### Écrire des données

**Vers Data Lake Storage :**

```python
# Écrire en Parquet
df.write.mode("overwrite").parquet(
    "abfss://container@account.dfs.core.windows.net/processed/users.parquet"
)

# Écrire en CSV
df.write.mode("overwrite").csv(
    "abfss://container@account.dfs.core.windows.net/processed/users.csv"
)
```

---

## Intégration avec autres services

### Databricks + Data Lake Storage

**Accès direct :**

```python
# Configurer l'accès
spark.conf.set(
    "fs.azure.account.auth.type.account.dfs.core.windows.net",
    "OAuth"
)
spark.conf.set(
    "fs.azure.account.oauth.provider.type.account.dfs.core.windows.net",
    "org.apache.hadoop.fs.azurebfs.oauth2.MsiTokenProvider"
)

# Lire
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/data/users.parquet"
)
```

### Databricks + Azure SQL Database

**Lire depuis SQL Database :**

```python
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:sqlserver://server.database.windows.net:1433;database=db") \
    .option("dbtable", "users") \
    .option("user", "sqladmin") \
    .option("password", "password") \
    .load()
```

**Écrire vers SQL Database :**

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
1. Source : Azure Blob Storage
2. Activity : Databricks Notebook
3. Sink : Azure SQL Database

**Configuration :**
- Notebook path : `/Workspace/path/to/notebook`
- Parameters : Passer des paramètres

---

## Bonnes pratiques

### Performance

1. **Utiliser le cache** pour réutiliser des données
2. **Partitionner** les données pour améliorer les performances
3. **Optimiser les transformations** pour réduire le temps
4. **Utiliser le bon nombre de workers**

### Coûts

1. **Arrêter les clusters** quand non utilisés
2. **Utiliser auto-termination** pour économiser
3. **Surveiller les coûts** dans Azure Cost Management
4. **Utiliser des clusters plus petits** pour débuter

### Organisation

1. **Organiser les notebooks** dans des dossiers
2. **Nommer clairement** les notebooks et clusters
3. **Documenter** le code
4. **Versionner** avec Git

### Sécurité

1. **Utiliser Azure AD** pour authentification
2. **Limiter les accès** avec RBAC
3. **Chiffrer les données** en transit et au repos
4. **Auditer** les accès

---

## Exemples pratiques

### Exemple 1 : Pipeline ETL complet

**Notebook Databricks :**

```python
# 1. Lire depuis Data Lake
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/raw/users.parquet"
)

# 2. Transformer
df_processed = df \
    .filter(df["status"] == "active") \
    .select("id", "name", "email", "created_at") \
    .withColumn("year", year(col("created_at")))

# 3. Écrire vers Data Lake
df_processed.write.mode("overwrite").parquet(
    "abfss://container@account.dfs.core.windows.net/processed/users.parquet"
)
```

### Exemple 2 : Analyse avec Spark SQL

```python
# Créer une vue temporaire
df.createOrReplaceTempView("users")

# Requête SQL
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

### Exemple 3 : Intégration avec Data Factory

1. Créer un notebook Databricks
2. Dans Data Factory, ajouter activité "Databricks Notebook"
3. Configurer le notebook
4. Exécuter le pipeline

---

## 📊 Points clés à retenir

1. **Databricks = Big Data** avec Apache Spark
2. **Notebooks** Python/SQL pour développement
3. **Clusters auto-scaling** pour performance
4. **Intégration native** avec services Azure
5. **Payant** : Surveiller les coûts

## 🔗 Prochain module

Passer au module [7. Projets pratiques](../07-projets/README.md) pour créer des projets complets avec Azure.

