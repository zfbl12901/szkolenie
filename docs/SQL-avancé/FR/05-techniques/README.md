# 5. Techniques d'optimisation

## 🎯 Objectifs

- Maîtriser les techniques d'optimisation des jointures
- Optimiser les agrégations et sous-requêtes
- Utiliser le partitionnement efficacement
- Exploiter le parallélisme PostgreSQL

## 📋 Table des matières

1. [Optimisation des jointures](#optimisation-des-jointures)
2. [Optimisation des agrégations](#optimisation-des-agrégations)
3. [Optimisation des sous-requêtes](#optimisation-des-sous-requêtes)
4. [Partitionnement](#partitionnement)
5. [Parallélisme](#parallélisme)
6. [Optimisation des types de données](#optimisation-des-types-de-données)

---

## Optimisation des jointures

### Choix du type de jointure

PostgreSQL choisit automatiquement, mais vous pouvez influencer :

**Nested Loop :**
- ✅ Petite table externe (< 1000 lignes)
- ✅ Index sur la clé de jointure
- ❌ Grande table externe

**Hash Join :**
- ✅ Tables de taille similaire
- ✅ Jointure d'égalité
- ✅ Suffisamment de `work_mem`
- ❌ Pas d'index nécessaire

**Merge Join :**
- ✅ Données déjà triées
- ✅ Jointures sur clés triées
- ❌ Nécessite un tri si non trié

### Optimiser avec des index

```sql
-- Avant : Jointure lente
EXPLAIN ANALYZE
SELECT u.*, o.*
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';

-- Créer des index sur les clés de jointure
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Après : Jointure optimisée
EXPLAIN ANALYZE
SELECT u.*, o.*
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';
```

### Réduire la taille des tables de jointure

```sql
-- Avant : Jointure sur toutes les lignes
SELECT u.*, o.*
FROM users u
JOIN orders o ON u.id = o.user_id;

-- Après : Filtrer avant la jointure
SELECT u.*, o.*
FROM (
    SELECT * FROM users WHERE active = true
) u
JOIN (
    SELECT * FROM orders WHERE status = 'completed'
) o ON u.id = o.user_id;
```

### Ordre des jointures

Le planificateur choisit l'ordre, mais vous pouvez influencer :

```sql
-- Utiliser des CTE pour forcer l'ordre
WITH filtered_users AS (
    SELECT * FROM users WHERE active = true
),
filtered_orders AS (
    SELECT * FROM orders WHERE status = 'completed'
)
SELECT u.*, o.*
FROM filtered_users u
JOIN filtered_orders o ON u.id = o.user_id;
```

### Jointures multiples

```sql
-- Optimiser l'ordre des jointures
-- PostgreSQL choisit généralement le bon ordre, mais vérifiez avec EXPLAIN

-- Mauvais : Jointure sur grande table en premier
SELECT *
FROM large_table l
JOIN small_table1 s1 ON l.id = s1.large_id
JOIN small_table2 s2 ON l.id = s2.large_id;

-- Meilleur : Filtrer d'abord
SELECT *
FROM large_table l
JOIN (
    SELECT large_id FROM small_table1 WHERE condition = true
) s1 ON l.id = s1.large_id
JOIN (
    SELECT large_id FROM small_table2 WHERE condition = true
) s2 ON l.id = s2.large_id;
```

---

## Optimisation des agrégations

### GROUP BY optimisé

```sql
-- Avant : Agrégation sur toutes les lignes
SELECT status, COUNT(*), AVG(amount)
FROM orders
GROUP BY status;

-- Après : Filtrer avant l'agrégation
SELECT status, COUNT(*), AVG(amount)
FROM orders
WHERE created_at > '2024-01-01'
GROUP BY status;

-- Index pour accélérer
CREATE INDEX idx_orders_status_created ON orders(status, created_at);
```

### Agrégations avec HAVING

```sql
-- Filtrer avec WHERE avant GROUP BY (plus efficace)
-- Mauvais
SELECT status, COUNT(*)
FROM orders
GROUP BY status
HAVING COUNT(*) > 100;

-- Meilleur : Utiliser une sous-requête
SELECT status, cnt
FROM (
    SELECT status, COUNT(*) AS cnt
    FROM orders
    GROUP BY status
) sub
WHERE cnt > 100;
```

### DISTINCT optimisé

```sql
-- DISTINCT peut être coûteux
SELECT DISTINCT user_id FROM orders;

-- Parfois GROUP BY est plus rapide
SELECT user_id FROM orders GROUP BY user_id;

-- Avec index, les deux peuvent être rapides
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

### Agrégations avec fenêtres

```sql
-- Utiliser des fonctions de fenêtre pour éviter les sous-requêtes
-- Avant : Sous-requête corrélée
SELECT 
    o.*,
    (SELECT AVG(amount) FROM orders o2 WHERE o2.user_id = o.user_id) AS avg_user_amount
FROM orders o;

-- Après : Fonction de fenêtre
SELECT 
    o.*,
    AVG(amount) OVER (PARTITION BY user_id) AS avg_user_amount
FROM orders o;
```

---

## Optimisation des sous-requêtes

### Sous-requêtes corrélées → JOIN

```sql
-- Avant : Sous-requête corrélée (lente)
SELECT 
    u.*,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;

-- Après : JOIN avec agrégation (plus rapide)
SELECT 
    u.*,
    COALESCE(o.order_count, 0) AS order_count
FROM users u
LEFT JOIN (
    SELECT user_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id;
```

### EXISTS vs IN vs JOIN

```sql
-- EXISTS : Généralement le plus rapide
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);

-- IN : Peut être lent si la liste est grande
SELECT *
FROM users u
WHERE u.id IN (
    SELECT user_id FROM orders
);

-- JOIN : Bon compromis
SELECT DISTINCT u.*
FROM users u
JOIN orders o ON u.id = o.user_id;
```

### Sous-requêtes dans SELECT

```sql
-- Éviter les sous-requêtes dans SELECT si possible
-- Avant : Sous-requête exécutée pour chaque ligne
SELECT 
    u.*,
    (SELECT MAX(created_at) FROM orders WHERE user_id = u.id) AS last_order_date
FROM users u;

-- Après : JOIN avec agrégation
SELECT 
    u.*,
    o.last_order_date
FROM users u
LEFT JOIN (
    SELECT user_id, MAX(created_at) AS last_order_date
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id;
```

### CTE (Common Table Expressions)

```sql
-- CTE pour améliorer la lisibilité et parfois la performance
WITH active_users AS (
    SELECT * FROM users WHERE active = true
),
recent_orders AS (
    SELECT * FROM orders WHERE created_at > '2024-01-01'
)
SELECT 
    u.*,
    COUNT(o.id) AS order_count
FROM active_users u
LEFT JOIN recent_orders o ON u.id = o.user_id
GROUP BY u.id;
```

---

## Partitionnement

### Partitionnement par plage (Range)

```sql
-- Créer une table partitionnée
CREATE TABLE orders (
    id SERIAL,
    user_id INTEGER,
    amount DECIMAL,
    created_at DATE
) PARTITION BY RANGE (created_at);

-- Créer les partitions
CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

CREATE TABLE orders_2024_q3 PARTITION OF orders
    FOR VALUES FROM ('2024-07-01') TO ('2024-10-01');

CREATE TABLE orders_2024_q4 PARTITION OF orders
    FOR VALUES FROM ('2024-10-01') TO ('2025-01-01');
```

**Avantages :**
- ✅ Partition pruning (seules les partitions pertinentes sont scannées)
- ✅ Maintenance par partition (VACUUM, ANALYZE)
- ✅ Suppression rapide de partitions entières

### Partitionnement par liste (List)

```sql
-- Partitionnement par région
CREATE TABLE users (
    id SERIAL,
    name TEXT,
    region TEXT
) PARTITION BY LIST (region);

CREATE TABLE users_europe PARTITION OF users
    FOR VALUES IN ('FR', 'DE', 'UK', 'IT');

CREATE TABLE users_america PARTITION OF users
    FOR VALUES IN ('US', 'CA', 'MX');

CREATE TABLE users_asia PARTITION OF users
    FOR VALUES IN ('JP', 'CN', 'IN');
```

### Partitionnement par hash

```sql
-- Partitionnement par hash (pour distribution uniforme)
CREATE TABLE events (
    id SERIAL,
    user_id INTEGER,
    event_type TEXT,
    created_at TIMESTAMP
) PARTITION BY HASH (user_id);

CREATE TABLE events_0 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE events_1 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE events_2 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE events_3 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

### Index sur tables partitionnées

```sql
-- Créer un index sur la table partitionnée (créé sur toutes les partitions)
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Ou créer des index spécifiques par partition
CREATE INDEX idx_orders_2024_q1_user_id ON orders_2024_q1(user_id);
```

### Maintenance des partitions

```sql
-- Vérifier les partitions
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
  AND tablename LIKE 'orders_%'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Supprimer une partition (très rapide)
DROP TABLE orders_2024_q1;  -- Supprime la partition et ses données

-- Détacher une partition (garder les données)
ALTER TABLE orders DETACH PARTITION orders_2024_q1;
```

---

## Parallélisme

### Configuration du parallélisme

```sql
-- Vérifier la configuration
SHOW max_parallel_workers_per_gather;
SHOW max_parallel_workers;
SHOW max_worker_processes;

-- Modifier (dans postgresql.conf)
-- max_parallel_workers_per_gather = 4
-- max_parallel_workers = 8
-- max_worker_processes = 8
```

### Quand le parallélisme est utilisé

PostgreSQL utilise le parallélisme pour :
- ✅ Scans séquentiels de grandes tables
- ✅ Jointures sur grandes tables
- ✅ Agrégations sur grandes tables
- ❌ Petites tables (< 8MB par défaut)
- ❌ Requêtes avec verrous

### Forcer le parallélisme

```sql
-- Augmenter min_parallel_table_scan_size pour forcer le parallélisme
SET min_parallel_table_scan_size = 0;  -- Toujours considérer le parallélisme

-- Voir le plan avec parallélisme
EXPLAIN ANALYZE
SELECT COUNT(*) FROM large_table WHERE condition = 'value';
```

**Résultat typique :**
```
Finalize Aggregate
  -> Gather
      Workers Planned: 4
      -> Partial Aggregate
          -> Parallel Seq Scan on large_table
```

### Optimiser pour le parallélisme

```sql
-- Tables avec beaucoup de colonnes : réduire work_mem par worker
SET work_mem = '64MB';

-- Tables partitionnées : parallélisme par partition
-- Chaque partition peut être scannée en parallèle
```

---

## Optimisation des types de données

### Choisir le bon type

```sql
-- Éviter TEXT pour des valeurs limitées
-- Avant
CREATE TABLE users (
    id SERIAL,
    status TEXT  -- 'active', 'inactive', 'pending'
);

-- Après : Utiliser ENUM ou VARCHAR
CREATE TYPE user_status AS ENUM ('active', 'inactive', 'pending');
CREATE TABLE users (
    id SERIAL,
    status user_status
);

-- Ou VARCHAR avec contrainte
CREATE TABLE users (
    id SERIAL,
    status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'pending'))
);
```

### Types numériques

```sql
-- Utiliser le type le plus petit possible
-- Avant
CREATE TABLE products (
    id BIGINT,  -- Si jamais > 2 milliards
    price DECIMAL(10,2)
);

-- Après : Adapter selon les besoins
CREATE TABLE products (
    id INTEGER,  -- Suffisant pour < 2 milliards
    price NUMERIC(10,2)  -- NUMERIC = DECIMAL
);
```

### Types de date/heure

```sql
-- Utiliser TIMESTAMP WITH TIME ZONE pour les dates/heures
CREATE TABLE events (
    id SERIAL,
    created_at TIMESTAMPTZ,  -- Stocke avec timezone
    event_date DATE  -- Pour les dates uniquement
);

-- Index sur les dates
CREATE INDEX idx_events_created_at ON events(created_at);
```

### JSON vs colonnes normales

```sql
-- JSON : Flexible mais plus lent
CREATE TABLE products (
    id SERIAL,
    metadata JSONB
);

-- Colonnes normales : Plus rapide si structure fixe
CREATE TABLE products (
    id SERIAL,
    brand TEXT,
    category TEXT,
    tags TEXT[]
);

-- Index GIN pour JSONB
CREATE INDEX idx_products_metadata_gin ON products USING gin(metadata);
```

---

## 📊 Points clés à retenir

1. **Index sur les clés de jointure** : Essentiel pour les jointures rapides
2. **Filtrer avant d'agréger** : Réduire la taille des données
3. **Éviter les sous-requêtes corrélées** : Utiliser JOIN à la place
4. **Partitionner les grandes tables** : Améliore les performances et la maintenance
5. **Parallélisme** : Automatique, mais configurable
6. **Types de données** : Choisir le plus approprié

## 🔗 Prochain module

Passer au module [6. Cas pratiques](../06-cas-pratiques/README.md) pour voir des exemples concrets d'optimisation.

