# 5. Azure Synapse Analytics - Data Warehouse

## 🎯 Objectifs

- Comprendre Azure Synapse Analytics
- Créer un workspace Synapse
- Charger des données
- Exécuter des requêtes SQL avancées
- Intégrer avec PowerBI

## 📋 Table des matières

1. [Introduction à Synapse](#introduction-à-synapse)
2. [Créer un workspace Synapse](#créer-un-workspace-synapse)
3. [Charger des données](#charger-des-données)
4. [Requêtes SQL avancées](#requêtes-sql-avancées)
5. [Intégration avec PowerBI](#intégration-avec-powerbi)
6. [Bonnes pratiques](#bonnes-pratiques)

---

## Introduction à Synapse

### Qu'est-ce qu'Azure Synapse Analytics ?

**Azure Synapse Analytics** = Plateforme d'analytics unifiée

- **Data Warehouse** : Stockage et analyse de données
- **Big Data** : Traitement de grandes quantités
- **SQL** : Requêtes SQL standard
- **Spark** : Traitement distribué
- **Intégration** : Avec tous les services Azure

### Composants Synapse

1. **SQL Pool** : Data warehouse SQL (anciennement SQL Data Warehouse)
2. **Spark Pool** : Clusters Spark pour Big Data
3. **Synapse Studio** : Interface web unifiée
4. **Pipelines** : ETL intégrés
5. **Notebooks** : Python, SQL, Scala

### Free Tier Synapse

**Gratuit avec crédit Azure :**
- Utiliser les 200$ de crédit gratuit (30 jours)
- Après : facturation normale

**⚠️ Important :** Synapse peut être coûteux. Surveiller attentivement les coûts.

---

## Créer un workspace Synapse

### Étape 1 : Accéder à Synapse

1. Portail Azure → Rechercher "Azure Synapse Analytics"
2. Cliquer sur "Azure Synapse Analytics"
3. Cliquer sur "Create"

### Étape 2 : Configuration de base

**Informations de base :**
- **Subscription** : Choisir votre abonnement
- **Resource group** : Créer ou utiliser existant
- **Workspace name** : `my-synapse-workspace`
- **Region** : Choisir la région
- **Data Lake Storage Gen2** : Créer nouveau ou utiliser existant

**SQL Administrator :**
- **SQL admin name** : `sqladmin`
- **Password** : Mot de passe fort

### Étape 3 : Configuration SQL Pool

**SQL Pool :**
- **Create a SQL pool** : ✅ Oui (pour débuter)
- **Performance level** : DW100c (le moins cher)
- **Or** : Créer plus tard (Serverless SQL)

**⚠️ Important :** Serverless SQL = pay-per-query, plus économique pour débuter.

### Étape 4 : Créer le workspace

1. Cliquer sur "Review + create"
2. Vérifier la configuration
3. Cliquer sur "Create"
4. Attendre la création (5-10 minutes)

**⚠️ Important :** Noter les credentials SQL.

---

## Charger des données

### Méthode 1 : COPY depuis Data Lake Storage

**Le plus rapide pour grandes quantités :**

```sql
-- Créer une table
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

-- Charger depuis Data Lake Storage
COPY INTO users
FROM 'https://mystorageaccount.dfs.core.windows.net/data-lake/raw/users.csv'
WITH (
    FILE_TYPE = 'CSV',
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
);
```

### Méthode 2 : Via Synapse Pipelines

**Pipeline intégré :**

1. Synapse Studio → "Integrate" → "Pipelines"
2. Créer un nouveau pipeline
3. Ajouter activité "Copy Data"
4. Source : Azure Blob Storage ou Data Lake
5. Sink : SQL Pool
6. Exécuter le pipeline

### Méthode 3 : INSERT (petites quantités)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Méthode 4 : Via PolyBase (External Tables)

**Créer une table externe :**

```sql
-- Créer un credential
CREATE DATABASE SCOPED CREDENTIAL BlobCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
SECRET = 'your-sas-token';

-- Créer une source de données externe
CREATE EXTERNAL DATA SOURCE BlobStorage
WITH (
    TYPE = HADOOP,
    LOCATION = 'wasbs://container@account.blob.core.windows.net',
    CREDENTIAL = BlobCredential
);

-- Créer un format externe
CREATE EXTERNAL FILE FORMAT CSVFormat
WITH (
    FORMAT_TYPE = DELIMITEDTEXT,
    FORMAT_OPTIONS (FIELD_TERMINATOR = ',')
);

-- Créer une table externe
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

-- Charger dans table interne
INSERT INTO users
SELECT * FROM users_external;
```

---

## Requêtes SQL avancées

### Requêtes de base

**SELECT simple :**

```sql
SELECT TOP 100 * FROM users;
```

**Agrégations :**

```sql
SELECT 
    YEAR(created_at) AS year,
    MONTH(created_at) AS month,
    COUNT(*) AS user_count
FROM users
GROUP BY YEAR(created_at), MONTH(created_at)
ORDER BY year, month;
```

### Window Functions

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

### Distribution et Performance

**Distribution keys :**

```sql
-- Distribution HASH (pour jointures)
CREATE TABLE users (
    id INT,
    name VARCHAR(100)
)
WITH (
    DISTRIBUTION = HASH(id),
    CLUSTERED COLUMNSTORE INDEX
);

-- Distribution ROUND_ROBIN (par défaut)
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
- Optimisé pour analytics
- Compression élevée
- Requêtes rapides sur grandes tables

---

## Intégration avec PowerBI

### Connexion directe

**Étape 1 : Dans PowerBI Desktop**

1. "Get Data" → "Azure" → "Azure Synapse Analytics SQL"
2. Entrer les informations :
   - **Server** : `my-synapse-workspace-ondemand.sql.azuresynapse.net` (Serverless)
   - **Database** : Nom de la base
   - **Data connectivity mode** : DirectQuery (recommandé)

**Étape 2 : Authentification**

- **Authentication method** : Database
- **Username** : `sqladmin`
- **Password** : Votre mot de passe

**Étape 3 : Sélectionner les tables**

- Choisir les tables ou vues
- Cliquer sur "Load"

### Créer des vues pour PowerBI

**Vue optimisée :**

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

**Utiliser la vue dans PowerBI :**
- Plus simple pour les utilisateurs
- Logique métier centralisée
- Performance optimisée

---

## Bonnes pratiques

### Performance

1. **Utiliser Columnstore Index** pour analytics
2. **Choisir les bonnes distribution keys**
3. **Partitionner** les grandes tables
4. **Optimiser les requêtes** avec EXPLAIN

### Coûts

1. **Utiliser Serverless SQL** pour débuter (pay-per-query)
2. **Pauser le SQL Pool** quand non utilisé
3. **Surveiller les coûts** dans Azure Cost Management
4. **Utiliser les bonnes tailles** de pool

### Organisation

1. **Créer des schémas** pour organiser
2. **Nommer clairement** les tables et vues
3. **Documenter** les schémas
4. **Utiliser des vues** pour simplifier

### Sécurité

1. **Utiliser Azure AD** pour authentification
2. **Limiter les accès** avec firewall rules
3. **Chiffrer les données** (activé par défaut)
4. **Auditer les accès**

---

## Exemples pratiques

### Exemple 1 : Pipeline complet Data Lake → Synapse

**Pipeline Synapse :**
1. Source : Data Lake Storage (Parquet)
2. Activity : Copy Data
3. Sink : SQL Pool
4. Trigger : Schedule (quotidien)

### Exemple 2 : Requêtes analytiques complexes

```sql
-- Analyse des ventes avec window functions
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

### Exemple 3 : Export vers PowerBI

1. Créer une vue analytique
2. Connecter PowerBI à la vue
3. Créer des visualisations
4. Publier le rapport

---

## 📊 Points clés à retenir

1. **Synapse = Plateforme analytics** unifiée
2. **SQL Pool** pour data warehouse
3. **Serverless SQL** pour pay-per-query
4. **Intégration PowerBI** native
5. **Scalable** de quelques Go à plusieurs Po

## 🔗 Prochain module

Passer au module [6. Azure Databricks - Big Data Analytics](../06-databricks/README.md) pour apprendre à utiliser Databricks pour le Big Data.

