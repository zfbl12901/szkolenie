# 4. Amazon Redshift - Data Warehouse

## 🎯 Objectifs

- Comprendre Amazon Redshift et son rôle
- Créer un cluster Redshift (gratuit 2 mois)
- Charger des données dans Redshift
- Optimiser les requêtes Redshift
- Intégrer avec S3 et autres services

## 📋 Table des matières

1. [Introduction à Redshift](#introduction-à-redshift)
2. [Créer un cluster Redshift](#créer-un-cluster-redshift)
3. [Charger des données](#charger-des-données)
4. [Requêtes SQL avancées](#requêtes-sql-avancées)
5. [Optimisation](#optimisation)
6. [Intégration avec autres services](#intégration-avec-autres-services)

---

## Introduction à Redshift

### Qu'est-ce qu'Amazon Redshift ?

**Amazon Redshift** = Data warehouse cloud géré

- **OLAP** : Optimisé pour l'analyse (pas transactions)
- **Colonnes** : Stockage orienté colonnes
- **Massivement parallèle** : Traitement distribué
- **Scalable** : De quelques Go à plusieurs Po

### Cas d'usage pour Data Analyst

- **Data Warehouse** : Centraliser les données
- **Analytics** : Requêtes complexes sur grandes volumes
- **Business Intelligence** : Dashboards et rapports
- **Data Mining** : Analyses approfondies

### Free Tier Redshift

**Gratuit 2 mois :**
- 750 heures/mois de cluster `dc2.large`
- 32 Go de stockage par nœud
- Après 2 mois : facturation normale

**⚠️ Important :** Arrêter le cluster quand non utilisé pour éviter les coûts.

---

## Créer un cluster Redshift

### Étape 1 : Accéder à Redshift

1. Console AWS → Rechercher "Redshift"
2. Cliquer sur "Amazon Redshift"
3. "Create cluster"

### Étape 2 : Configuration du cluster

**Configuration de base :**

1. **Cluster identifier** : `data-analyst-cluster`
2. **Node type** : `dc2.large` (gratuit 2 mois)
3. **Number of nodes** : 1 (suffisant pour débuter)
4. **Database name** : `analytics` (par défaut : `dev`)
5. **Database port** : 5439 (par défaut)
6. **Master username** : `admin` (ou autre)
7. **Master password** : Mot de passe fort

**Configuration réseau :**

1. **VPC** : Choisir un VPC existant
2. **Subnet group** : Créer ou utiliser existant
3. **Publicly accessible** : ✅ Oui (pour accès facile)
4. **Availability zone** : Choisir une zone

**Sécurité :**

1. **VPC security groups** : Créer un groupe de sécurité
   - Autoriser le port 5439 depuis votre IP
2. **Encryption** : Activer (recommandé)

### Étape 3 : Créer le cluster

1. Cliquer sur "Create cluster"
2. Attendre 5-10 minutes (création)
3. Cluster prêt quand status = "Available"

**⚠️ Important :** Noter l'endpoint du cluster (ex: `data-analyst-cluster.xxxxx.eu-west-3.redshift.amazonaws.com:5439`)

---

## Charger des données

### Méthode 1 : COPY depuis S3 (recommandé)

**Le plus rapide pour grandes quantités :**

```sql
-- Créer une table
CREATE TABLE users (
    id INTEGER,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP
);

-- Charger depuis S3
COPY users
FROM 's3://my-bucket/data/users.csv'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
IGNOREHEADER 1;
```

**Configuration IAM Role :**

1. IAM → "Roles" → "Create role"
2. Type : "Redshift"
3. Attacher politique : `AmazonS3ReadOnlyAccess`
4. Nom : `RedshiftS3Role`
5. Copier l'ARN pour COPY

### Méthode 2 : INSERT (petites quantités)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Méthode 3 : INSERT depuis requête

```sql
INSERT INTO users_aggregated
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS user_count
FROM users
GROUP BY DATE_TRUNC('month', created_at);
```

### Formats supportés

- **CSV** : Fichiers CSV
- **JSON** : Fichiers JSON
- **Parquet** : Format optimisé (recommandé)
- **Avro** : Format Avro

---

## Requêtes SQL avancées

### Fonctions analytiques

**Window functions :**

```sql
-- ROW_NUMBER
SELECT 
    id,
    name,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY created_at) AS rank
FROM products;

-- LAG/LEAD
SELECT 
    date,
    sales,
    LAG(sales, 1) OVER (ORDER BY date) AS previous_sales,
    LEAD(sales, 1) OVER (ORDER BY date) AS next_sales
FROM daily_sales;

-- RANK
SELECT 
    user_id,
    total_spent,
    RANK() OVER (ORDER BY total_spent DESC) AS spending_rank
FROM user_totals;
```

### Agrégations complexes

```sql
-- GROUP BY avec ROLLUP
SELECT 
    category,
    region,
    SUM(amount) AS total
FROM sales
GROUP BY ROLLUP(category, region);

-- GROUP BY avec CUBE
SELECT 
    category,
    region,
    SUM(amount) AS total
FROM sales
GROUP BY CUBE(category, region);
```

### Jointures optimisées

```sql
-- Jointure avec distribution key
SELECT 
    u.name,
    o.amount,
    o.created_at
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';
```

---

## Optimisation

### Distribution keys

**Choisir la bonne distribution key :**

```sql
-- Distribution par clé (pour jointures)
CREATE TABLE users (
    id INTEGER DISTKEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Distribution ALL (pour petites tables)
CREATE TABLE categories (
    id INTEGER,
    name VARCHAR(100)
) DISTSTYLE ALL;

-- Distribution EVEN (par défaut)
CREATE TABLE logs (
    id INTEGER,
    message TEXT
) DISTSTYLE EVEN;
```

### Sort keys

**Améliorer les performances de requêtes :**

```sql
-- Sort key simple
CREATE TABLE orders (
    id INTEGER,
    user_id INTEGER,
    created_at TIMESTAMP,
    amount DECIMAL(10,2)
) SORTKEY (created_at);

-- Sort key composite
CREATE TABLE sales (
    date DATE,
    region VARCHAR(50),
    amount DECIMAL(10,2)
) SORTKEY (date, region);
```

### Compression

**Réduire l'espace de stockage :**

```sql
-- Compression automatique
CREATE TABLE users (
    id INTEGER,
    name VARCHAR(100) ENCODE lzo,
    email VARCHAR(100) ENCODE lzo,
    created_at TIMESTAMP ENCODE delta
);
```

### ANALYZE

**Mettre à jour les statistiques :**

```sql
-- Analyser une table
ANALYZE users;

-- Analyser toutes les tables
ANALYZE;
```

---

## Intégration avec autres services

### Redshift + S3

**Unload vers S3 :**

```sql
UNLOAD ('SELECT * FROM users WHERE created_at > ''2024-01-01''')
TO 's3://my-bucket/exports/users/'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
PARALLEL OFF;
```

### Redshift + Glue

**Glue peut charger dans Redshift :**

```python
# Dans un job Glue
glueContext.write_dynamic_frame.from_jdbc_conf(
    frame = transformed_data,
    catalog_connection = "redshift-connection",
    connection_options = {
        "dbtable": "users",
        "database": "analytics"
    }
)
```

### Redshift + QuickSight

**Connecter QuickSight à Redshift :**

1. QuickSight → "Data sources"
2. "Redshift"
3. Entrer les informations de connexion
4. Sélectionner les tables
5. Créer des visualisations

---

## Bonnes pratiques

### Performance

1. **Utiliser COPY** au lieu de INSERT pour grandes quantités
2. **Choisir les bonnes distribution keys**
3. **Utiliser des sort keys** pour requêtes fréquentes
4. **Compresser les colonnes** pour économiser l'espace
5. **VACUUM régulièrement** pour optimiser

### Coûts

1. **Arrêter le cluster** quand non utilisé
2. **Utiliser le bon type de nœud** selon les besoins
3. **Surveiller l'utilisation** du stockage
4. **Nettoyer les données** inutiles

### Sécurité

1. **Chiffrer les données** en transit et au repos
2. **Utiliser VPC** pour isoler le cluster
3. **Limiter l'accès** avec security groups
4. **Auditer les accès** avec CloudTrail

---

## Exemples pratiques

### Exemple 1 : Pipeline complet S3 → Redshift

```sql
-- 1. Créer la table
CREATE TABLE sales (
    id INTEGER,
    product_id INTEGER,
    amount DECIMAL(10,2),
    sale_date DATE
) DISTKEY(product_id) SORTKEY(sale_date);

-- 2. Charger depuis S3
COPY sales
FROM 's3://my-bucket/data/sales/'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
IGNOREHEADER 1;

-- 3. Analyser
ANALYZE sales;

-- 4. Requêtes analytiques
SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total_sales
FROM sales
GROUP BY DATE_TRUNC('month', sale_date)
ORDER BY month;
```

### Exemple 2 : Agrégations avec window functions

```sql
-- Top 10 produits par mois
SELECT 
    product_id,
    month,
    total_sales,
    RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) AS rank
FROM (
    SELECT 
        product_id,
        DATE_TRUNC('month', sale_date) AS month,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY product_id, DATE_TRUNC('month', sale_date)
) monthly_sales
WHERE RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) <= 10;
```

---

## 📊 Points clés à retenir

1. **Redshift = Data warehouse** pour analytics
2. **Free Tier : 2 mois** gratuit (750 heures)
3. **COPY depuis S3** = méthode la plus rapide
4. **Distribution et sort keys** = clés de performance
5. **Arrêter le cluster** quand non utilisé

## 🔗 Prochain module

Passer au module [5. Amazon Athena - Requêtes SQL sur S3](../05-athena/README.md) pour apprendre à interroger directement les fichiers S3.

