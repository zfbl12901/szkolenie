# 4. Azure SQL Database - Base de données

## 🎯 Objectifs

- Comprendre Azure SQL Database
- Créer une base SQL Database (gratuit jusqu'à 32 Go)
- Migrer des données
- Optimiser les requêtes
- Intégrer avec PowerBI

## 📋 Table des matières

1. [Introduction à SQL Database](#introduction-à-sql-database)
2. [Créer une base SQL Database](#créer-une-base-sql-database)
3. [Se connecter à la base](#se-connecter-à-la-base)
4. [Charger des données](#charger-des-données)
5. [Requêtes SQL](#requêtes-sql)
6. [Intégration avec PowerBI](#intégration-avec-powerbi)

---

## Introduction à SQL Database

### Qu'est-ce qu'Azure SQL Database ?

**Azure SQL Database** = Base de données SQL cloud gérée

- **SQL Server compatible** : Syntaxe SQL standard
- **Géré** : Microsoft gère l'infrastructure
- **Scalable** : De quelques Go à plusieurs To
- **Haute disponibilité** : 99.99% de disponibilité

### Cas d'usage pour Data Analyst

- **Data Warehouse** : Centraliser les données
- **Analytics** : Requêtes complexes
- **Business Intelligence** : Source pour PowerBI
- **Data Integration** : Point central pour ETL

### Free Tier SQL Database

**Gratuit 12 mois :**
- **Basic tier** : Jusqu'à 32 Go
- **DTU** : 5 DTU (Database Transaction Units)
- **Backup** : Automatique (7 jours)

**⚠️ Important :** Après 12 mois, facturation normale. Surveiller les coûts.

---

## Créer une base SQL Database

### Étape 1 : Accéder à SQL Database

1. Portail Azure → Rechercher "SQL databases"
2. Cliquer sur "SQL databases"
3. Cliquer sur "Create"

### Étape 2 : Configuration de base

**Informations de base :**
- **Subscription** : Choisir votre abonnement
- **Resource group** : Créer ou utiliser existant
- **Database name** : `analytics-db`
- **Server** : Créer un nouveau serveur ou utiliser existant

**Créer un serveur SQL :**
- **Server name** : `my-sql-server-xxxxx` (unique globalement)
- **Location** : Choisir la région
- **Authentication method** : SQL authentication (ou Azure AD)
- **Server admin login** : `sqladmin` (ou autre)
- **Password** : Mot de passe fort
- **Allow Azure services** : ✅ Oui (pour Data Factory)

### Étape 3 : Configuration de la base

**Compute + storage :**
- **Service tier** : Basic (pour Free Tier)
- **Compute tier** : Serverless (ou Provisioned)
- **Storage** : 2 Go (gratuit, extensible jusqu'à 32 Go)

**⚠️ Important :** Basic tier = 5 DTU, suffisant pour débuter.

### Étape 4 : Configuration réseau

**Networking :**
- **Public endpoint** : ✅ Enable
- **Firewall rules** :
  - ✅ Allow Azure services and resources
  - Ajouter votre IP pour accès local

### Étape 5 : Créer la base

1. Cliquer sur "Review + create"
2. Vérifier la configuration
3. Cliquer sur "Create"
4. Attendre la création (2-3 minutes)

**⚠️ Important :** Noter le nom du serveur et les credentials.

---

## Se connecter à la base

### Via Azure Portal (Query Editor)

1. SQL Database → "Query editor"
2. Entrer les credentials
3. Exécuter des requêtes SQL

### Via SQL Server Management Studio (SSMS)

**Télécharger SSMS :**
- https://aka.ms/ssmsfullsetup

**Connexion :**
- **Server name** : `my-sql-server-xxxxx.database.windows.net`
- **Authentication** : SQL Server Authentication
- **Login** : `sqladmin`
- **Password** : Votre mot de passe

### Via Azure Data Studio

**Télécharger Azure Data Studio :**
- https://aka.ms/azuredatastudio

**Avantages :**
- Gratuit et open-source
- Interface moderne
- Support notebooks
- Intégration Git

### Via Python (pyodbc)

```python
import pyodbc

# Connexion
server = 'my-sql-server-xxxxx.database.windows.net'
database = 'analytics-db'
username = 'sqladmin'
password = 'your-password'
driver = '{ODBC Driver 17 for SQL Server}'

conn = pyodbc.connect(
    f'DRIVER={driver};SERVER={server};DATABASE={database};UID={username};PWD={password}'
)

# Exécuter une requête
cursor = conn.cursor()
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()
for row in rows:
    print(row)
```

---

## Charger des données

### Méthode 1 : INSERT (petites quantités)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Méthode 2 : BULK INSERT depuis Blob Storage

**Prérequis :**
- Créer une clé SAS pour Blob Storage
- Créer un credential dans SQL Database

**Exemple :**

```sql
-- Créer un credential
CREATE DATABASE SCOPED CREDENTIAL BlobCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
SECRET = 'your-sas-token';

-- Créer une source de données externe
CREATE EXTERNAL DATA SOURCE BlobStorage
WITH (
    TYPE = BLOB_STORAGE,
    LOCATION = 'https://mystorageaccount.blob.core.windows.net',
    CREDENTIAL = BlobCredential
);

-- Importer depuis Blob Storage
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

### Méthode 3 : Via Data Factory

**Pipeline :**
1. Source : Azure Blob Storage (CSV)
2. Activity : Copy Data
3. Sink : Azure SQL Database

**Configuration :**
- Source : `raw-data/users.csv`
- Sink : Table `users` dans SQL Database
- Mapping : Colonnes automatique ou manuel

### Méthode 4 : Via Python (pandas)

```python
import pandas as pd
import pyodbc

# Lire un fichier CSV
df = pd.read_csv('users.csv')

# Connexion
conn = pyodbc.connect(connection_string)

# Écrire dans SQL Database
df.to_sql('users', conn, if_exists='append', index=False)
```

---

## Requêtes SQL

### Requêtes de base

**SELECT simple :**

```sql
SELECT * FROM users LIMIT 10;
```

**Filtrer :**

```sql
SELECT id, name, email
FROM users
WHERE created_at > '2024-01-01'
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

## Intégration avec PowerBI

### Connexion directe

**Étape 1 : Dans PowerBI Desktop**

1. "Get Data" → "Azure" → "Azure SQL Database"
2. Entrer les informations :
   - **Server** : `my-sql-server-xxxxx.database.windows.net`
   - **Database** : `analytics-db`
   - **Data connectivity mode** : Import (ou DirectQuery)

**Étape 2 : Authentification**

- **Authentication method** : Database
- **Username** : `sqladmin`
- **Password** : Votre mot de passe

**Étape 3 : Sélectionner les tables**

- Choisir les tables à importer
- Cliquer sur "Load"

### DirectQuery vs Import

**Import :**
- ✅ Rapide pour visualisations
- ✅ Fonctionne hors ligne
- ❌ Données statiques (nécessite refresh)

**DirectQuery :**
- ✅ Données en temps réel
- ✅ Pas de limite de taille
- ❌ Plus lent (requêtes à chaque interaction)

### Créer des visualisations

**Exemple :**
1. Importer la table `users`
2. Créer un graphique : Nombre d'utilisateurs par mois
3. Ajouter des filtres
4. Publier sur PowerBI Service

---

## Bonnes pratiques

### Performance

1. **Créer des index** sur colonnes fréquemment utilisées
2. **Optimiser les requêtes** avec EXPLAIN
3. **Utiliser des vues** pour simplifier
4. **Partitionner** les grandes tables

### Coûts

1. **Surveiller l'utilisation** dans Azure Cost Management
2. **Utiliser Basic tier** pour débuter
3. **Arrêter la base** si non utilisée (Serverless)
4. **Nettoyer les données** inutiles

### Sécurité

1. **Utiliser Azure AD** pour authentification
2. **Limiter les accès** avec firewall rules
3. **Chiffrer les données** (activé par défaut)
4. **Auditer les accès** avec SQL Auditing

### Organisation

1. **Nommer clairement** les tables et colonnes
2. **Documenter** les schémas
3. **Utiliser des schémas** pour organiser
4. **Versionner** les scripts SQL (Git)

---

## Exemples pratiques

### Exemple 1 : Pipeline complet Blob → SQL Database

**Via Data Factory :**
1. Source : Azure Blob Storage (CSV)
2. Activity : Copy Data
3. Sink : Azure SQL Database
4. Trigger : Schedule (quotidien)

### Exemple 2 : Requêtes analytiques

```sql
-- Top 10 utilisateurs par dépenses
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

### Exemple 3 : Export vers PowerBI

1. Créer une vue pour PowerBI
2. Connecter PowerBI à la vue
3. Créer des visualisations
4. Publier le rapport

---

## 📊 Points clés à retenir

1. **SQL Database = Base SQL cloud** gérée par Microsoft
2. **Free Tier : 32 Go** pendant 12 mois (Basic tier)
3. **Compatible SQL Server** : Syntaxe standard
4. **Intégration PowerBI** : Connexion directe
5. **Scalable** : De Basic à Premium

## 🔗 Prochain module

Passer au module [5. Azure Synapse Analytics - Data Warehouse](../05-synapse/README.md) pour apprendre à utiliser Synapse pour l'analyse de données.

