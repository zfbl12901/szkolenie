# 6. Cas pratiques d'optimisation

## 🎯 Objectifs

- Appliquer les techniques d'optimisation sur des cas réels
- Analyser les problèmes de performance
- Mesurer l'impact des optimisations
- Utiliser Dalibo pour identifier et résoudre les problèmes

## 📋 Table des matières

1. [Cas 1 : Requête lente avec scan séquentiel](#cas-1--requête-lente-avec-scan-séquentiel)
2. [Cas 2 : Jointure lente sur grande table](#cas-2--jointure-lente-sur-grande-table)
3. [Cas 3 : Agrégation lente](#cas-3--agrégation-lente)
4. [Cas 4 : Sous-requête corrélée](#cas-4--sous-requête-corrélée)
5. [Cas 5 : Problème de cache hit ratio](#cas-5--problème-de-cache-hit-ratio)
6. [Cas 6 : Index non utilisés](#cas-6--index-non-utilisés)

---

## Cas 1 : Requête lente avec scan séquentiel

### Problème initial

**Requête :**
```sql
SELECT * FROM users WHERE email = 'user@example.com';
```

**Plan d'exécution :**
```
Seq Scan on users  (cost=0.00..25000.00 rows=1 width=64)
  (actual time=0.123..1500.456 rows=1 loops=1)
  Filter: (email = 'user@example.com'::text)
  Rows Removed by Filter: 999999
Planning Time: 0.234 ms
Execution Time: 1500.678 ms
```

**Problèmes identifiés :**
- 🔴 Scan séquentiel sur 1 million de lignes
- 🔴 Temps d'exécution : 1.5 secondes
- 🔴 999999 lignes filtrées

### Analyse avec Dalibo

```sql
-- Vérifier dans pg_stat_statements
SELECT 
    query,
    calls,
    mean_exec_time,
    shared_blks_read,
    shared_blks_hit
FROM pg_stat_statements
WHERE query LIKE '%users WHERE email%'
ORDER BY mean_exec_time DESC;

-- Vérifier avec pg_qualstats
SELECT 
    left_table,
    left_column,
    operator,
    execution_count
FROM pg_qualstats
WHERE left_table = 'users' AND left_column = 'email';
```

### Solution

```sql
-- Créer un index sur email
CREATE INDEX idx_users_email ON users(email);

-- Vérifier le nouveau plan
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'user@example.com';
```

**Plan optimisé :**
```
Index Scan using idx_users_email on users
  (cost=0.42..8.44 rows=1 width=64)
  (actual time=0.123..0.125 rows=1 loops=1)
  Index Cond: (email = 'user@example.com'::text)
Planning Time: 0.234 ms
Execution Time: 0.125 ms
```

### Résultats

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Temps d'exécution | 1500ms | 0.125ms | **99.99%** |
| Type de scan | Seq Scan | Index Scan | ✅ |
| Lignes scannées | 1,000,000 | 1 | ✅ |

---

## Cas 2 : Jointure lente sur grande table

### Problème initial

**Requête :**
```sql
SELECT 
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email;
```

**Plan d'exécution :**
```
Hash Join
  (cost=125000.00..250000.00 rows=50000 width=64)
  (actual time=5000.123..15000.456 rows=45000 loops=1)
  Hash Cond: (o.user_id = u.id)
  -> Seq Scan on orders o
      (cost=0.00..100000.00 rows=1000000 width=16)
      (actual time=0.123..5000.456 rows=1000000 loops=1)
  -> Hash
      (cost=25000.00..25000.00 rows=100000 width=48)
      (actual time=2000.123..2000.123 rows=100000 loops=1)
      Buckets: 131072  Batches: 8  Memory Usage: 5120kB
      -> Seq Scan on users u
          (cost=0.00..25000.00 rows=100000 width=48)
          (actual time=0.089..1000.567 rows=100000 loops=1)
          Filter: (created_at > '2024-01-01'::date)
Planning Time: 50.234 ms
Execution Time: 15000.678 ms
```

**Problèmes identifiés :**
- 🔴 Hash Join avec 8 batches (tri sur disque)
- 🔴 Scan séquentiel sur orders (1 million de lignes)
- 🔴 Temps d'exécution : 15 secondes

### Analyse avec Dalibo

```sql
-- Identifier les index manquants
SELECT 
    qs.left_table,
    qs.left_column,
    qs.operator,
    COUNT(*) AS execution_count
FROM pg_qualstats qs
WHERE qs.left_table IN ('users', 'orders')
GROUP BY qs.left_table, qs.left_column, qs.operator
ORDER BY execution_count DESC;
```

### Solution

```sql
-- Créer des index sur les clés de jointure et filtres
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Index composite pour la requête complète
CREATE INDEX idx_orders_user_id_amount ON orders(user_id, amount);

-- Augmenter work_mem pour éviter les batches
SET work_mem = '256MB';

-- Vérifier le nouveau plan
EXPLAIN ANALYZE
SELECT 
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email;
```

**Plan optimisé :**
```
Hash Join
  (cost=5000.00..15000.00 rows=50000 width=64)
  (actual time=200.123..800.456 rows=45000 loops=1)
  Hash Cond: (o.user_id = u.id)
  -> Index Scan using idx_orders_user_id on orders o
      (cost=0.42..8000.00 rows=500000 width=16)
      (actual time=0.123..300.456 rows=500000 loops=1)
  -> Hash
      (cost=2500.00..2500.00 rows=100000 width=48)
      (actual time=100.123..100.123 rows=100000 loops=1)
      Buckets: 131072  Batches: 1  Memory Usage: 5120kB
      -> Index Scan using idx_users_created_at on users u
          (cost=0.42..2500.00 rows=100000 width=48)
          (actual time=0.089..50.567 rows=100000 loops=1)
          Index Cond: (created_at > '2024-01-01'::date)
Planning Time: 5.234 ms
Execution Time: 800.678 ms
```

### Résultats

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Temps d'exécution | 15000ms | 800ms | **94.7%** |
| Batches Hash Join | 8 | 1 | ✅ |
| Type de scan orders | Seq Scan | Index Scan | ✅ |
| Lignes scannées | 1,000,000 | 500,000 | ✅ |

---

## Cas 3 : Agrégation lente

### Problème initial

**Requête :**
```sql
SELECT 
    status,
    COUNT(*) AS count,
    AVG(amount) AS avg_amount,
    SUM(amount) AS total_amount
FROM orders
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY status
ORDER BY count DESC;
```

**Plan d'exécution :**
```
Sort
  (cost=50000.00..50000.00 rows=5 width=32)
  (actual time=10000.123..10000.234 rows=5 loops=1)
  Sort Key: (count(*)) DESC
  Sort Method: quicksort  Memory: 25kB
  -> HashAggregate
      (cost=45000.00..45000.00 rows=5 width=32)
      (actual time=8000.123..8000.456 rows=5 loops=1)
      Group Key: status
      Batches: 1  Memory Usage: 24kB
      -> Seq Scan on orders
          (cost=0.00..40000.00 rows=2000000 width=16)
          (actual time=0.123..5000.456 rows=2000000 loops=1)
          Filter: ((created_at >= '2024-01-01'::date) 
                   AND (created_at <= '2024-12-31'::date))
          Rows Removed by Filter: 0
Planning Time: 10.234 ms
Execution Time: 10000.678 ms
```

**Problèmes identifiés :**
- 🔴 Scan séquentiel sur 2 millions de lignes
- 🔴 Temps d'exécution : 10 secondes
- 🔴 Pas d'index sur created_at

### Solution

```sql
-- Créer un index sur created_at et status
CREATE INDEX idx_orders_created_at_status ON orders(created_at, status);

-- Alternative : Index partiel si certaines status sont rares
CREATE INDEX idx_orders_created_at_status_partial 
ON orders(created_at, status) 
WHERE status IN ('pending', 'processing');

-- Vérifier le nouveau plan
EXPLAIN ANALYZE
SELECT 
    status,
    COUNT(*) AS count,
    AVG(amount) AS avg_amount,
    SUM(amount) AS total_amount
FROM orders
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY status
ORDER BY count DESC;
```

**Plan optimisé :**
```
Sort
  (cost=5000.00..5000.00 rows=5 width=32)
  (actual time=500.123..500.234 rows=5 loops=1)
  Sort Key: (count(*)) DESC
  Sort Method: quicksort  Memory: 25kB
  -> HashAggregate
      (cost=4500.00..4500.00 rows=5 width=32)
      (actual time=400.123..400.456 rows=5 loops=1)
      Group Key: status
      Batches: 1  Memory Usage: 24kB
      -> Index Scan using idx_orders_created_at_status on orders
          (cost=0.42..4000.00 rows=2000000 width=16)
          (actual time=0.123..200.456 rows=2000000 loops=1)
          Index Cond: ((created_at >= '2024-01-01'::date) 
                       AND (created_at <= '2024-12-31'::date))
Planning Time: 5.234 ms
Execution Time: 500.678 ms
```

### Résultats

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Temps d'exécution | 10000ms | 500ms | **95%** |
| Type de scan | Seq Scan | Index Scan | ✅ |
| Lignes scannées | 2,000,000 | 2,000,000 | (même nombre, mais index) |

---

## Cas 4 : Sous-requête corrélée

### Problème initial

**Requête :**
```sql
SELECT 
    u.id,
    u.name,
    u.email,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count,
    (SELECT MAX(created_at) FROM orders o WHERE o.user_id = u.id) AS last_order_date,
    (SELECT SUM(amount) FROM orders o WHERE o.user_id = u.id) AS total_spent
FROM users u
WHERE u.active = true;
```

**Plan d'exécution :**
```
Seq Scan on users u
  (cost=0.00..250000.00 rows=100000 width=64)
  (actual time=0.123..50000.456 rows=100000 loops=1)
  Filter: (active = true)
  SubPlan 1
    -> Aggregate
        (cost=2.50..2.50 rows=1 width=8)
        (actual time=0.100..0.100 rows=1 loops=100000)
        -> Seq Scan on orders o
            (cost=0.00..2.25 rows=10 width=0)
            (actual time=0.050..0.050 rows=5 loops=100000)
            Filter: (user_id = u.id)
  SubPlan 2
    -> Result
        (cost=2.50..2.50 rows=1 width=8)
        (actual time=0.100..0.100 rows=1 loops=100000)
        -> Aggregate
            (cost=2.50..2.50 rows=1 width=8)
            (actual time=0.100..0.100 rows=1 loops=100000)
            -> Seq Scan on orders o
                (cost=0.00..2.25 rows=10 width=0)
                (actual time=0.050..0.050 rows=5 loops=100000)
                Filter: (user_id = u.id)
  SubPlan 3
    -> Aggregate
        (cost=2.50..2.50 rows=1 width=8)
        (actual time=0.100..0.100 rows=1 loops=100000)
        -> Seq Scan on orders o
            (cost=0.00..2.25 rows=10 width=0)
            (actual time=0.050..0.050 rows=5 loops=100000)
            Filter: (user_id = u.id)
Planning Time: 5.234 ms
Execution Time: 50000.678 ms
```

**Problèmes identifiés :**
- 🔴 3 sous-requêtes corrélées exécutées 100,000 fois chacune
- 🔴 300,000 scans séquentiels sur orders
- 🔴 Temps d'exécution : 50 secondes

### Solution

```sql
-- Remplacer par des JOIN avec agrégation
SELECT 
    u.id,
    u.name,
    u.email,
    COALESCE(o.order_count, 0) AS order_count,
    o.last_order_date,
    COALESCE(o.total_spent, 0) AS total_spent
FROM users u
LEFT JOIN (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        MAX(created_at) AS last_order_date,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id
WHERE u.active = true;

-- Créer un index pour accélérer la jointure
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

**Plan optimisé :**
```
Hash Right Join
  (cost=5000.00..15000.00 rows=100000 width=64)
  (actual time=200.123..800.456 rows=100000 loops=1)
  Hash Cond: (o.user_id = u.id)
  -> HashAggregate
      (cost=4000.00..4500.00 rows=50000 width=24)
      (actual time=150.123..200.456 rows=50000 loops=1)
      Group Key: orders.user_id
      Batches: 1  Memory Usage: 5120kB
      -> Index Scan using idx_orders_user_id on orders
          (cost=0.42..3000.00 rows=500000 width=16)
          (actual time=0.123..100.456 rows=500000 loops=1)
  -> Hash
      (cost=2000.00..2000.00 rows=100000 width=48)
      (actual time=50.123..50.123 rows=100000 loops=1)
      Buckets: 131072  Batches: 1  Memory Usage: 5120kB
      -> Seq Scan on users u
          (cost=0.00..2000.00 rows=100000 width=48)
          (actual time=0.089..25.567 rows=100000 loops=1)
          Filter: (active = true)
Planning Time: 5.234 ms
Execution Time: 800.678 ms
```

### Résultats

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Temps d'exécution | 50000ms | 800ms | **98.4%** |
| Scans sur orders | 300,000 | 1 | ✅ |
| Type d'opération | Sous-requêtes | Hash Join | ✅ |

---

## Cas 5 : Problème de cache hit ratio

### Problème initial

**Métriques :**
```sql
-- Cache hit ratio global
SELECT 
    ROUND(100.0 * SUM(shared_blks_hit) / 
          NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2) AS cache_hit_ratio
FROM pg_stat_statements;
-- Résultat: 75% (objectif: > 95%)
```

**Requêtes avec beaucoup de lectures disque :**
```sql
SELECT 
    LEFT(query, 100) AS query_preview,
    shared_blks_read,
    shared_blks_hit,
    ROUND(100.0 * shared_blks_hit / 
          NULLIF(shared_blks_hit + shared_blks_read, 0), 2) AS cache_hit_ratio,
    ROUND((shared_blks_read * 8)::numeric / 1024, 2) AS disk_read_mb
FROM pg_stat_statements
WHERE shared_blks_read > 1000
ORDER BY shared_blks_read DESC
LIMIT 10;
```

### Solution

```sql
-- 1. Augmenter shared_buffers (dans postgresql.conf)
-- shared_buffers = 4GB  (25% de RAM pour serveur dédié)

-- 2. Précharger les tables fréquemment utilisées
-- Créer une fonction de préchargement
CREATE OR REPLACE FUNCTION pg_prewarm_table(table_name TEXT)
RETURNS void AS $$
BEGIN
    EXECUTE format('SELECT * FROM %I LIMIT 1', table_name);
END;
$$ LANGUAGE plpgsql;

-- Précharger les tables importantes
SELECT pg_prewarm_table('users');
SELECT pg_prewarm_table('orders');
SELECT pg_prewarm_table('products');

-- 3. Utiliser pg_prewarm extension
CREATE EXTENSION IF NOT EXISTS pg_prewarm;

-- Précharger une table complète
SELECT pg_prewarm('users');
SELECT pg_prewarm('orders');
```

**Après optimisation :**
```sql
-- Vérifier l'amélioration
SELECT 
    ROUND(100.0 * SUM(shared_blks_hit) / 
          NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2) AS cache_hit_ratio
FROM pg_stat_statements;
-- Résultat: 98% ✅
```

### Résultats

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Cache hit ratio | 75% | 98% | **+23%** |
| Lectures disque | Élevées | Faibles | ✅ |
| Temps de réponse | Variable | Stable | ✅ |

---

## Cas 6 : Index non utilisés

### Problème initial

**Identification des index non utilisés :**
```sql
-- Index jamais utilisés
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans,
    pg_relation_size(indexrelid) AS size_bytes
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND pg_relation_size(indexrelid) > 1048576  -- > 1MB
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Résultat :**
```
 schemaname | tablename |      indexname       | index_size | index_scans | size_bytes
------------+-----------+----------------------+------------+-------------+------------
 public     | orders    | idx_orders_old_field | 250 MB     |           0 |  262144000
 public     | users     | idx_users_old_email  | 150 MB     |           0 |  157286400
```

**Impact :**
- 🔴 400 MB d'espace disque perdu
- 🔴 Ralentissement des INSERT/UPDATE
- 🔴 Maintenance inutile

### Solution

```sql
-- 1. Vérifier avec HypoPG si l'index est vraiment inutile
CREATE EXTENSION IF NOT EXISTS hypopg;

-- 2. Analyser les requêtes qui pourraient utiliser l'index
SELECT 
    query,
    calls,
    mean_exec_time
FROM pg_stat_statements
WHERE query LIKE '%old_field%' OR query LIKE '%old_email%';

-- 3. Si vraiment inutile, supprimer l'index
DROP INDEX idx_orders_old_field;
DROP INDEX idx_users_old_email;

-- 4. Vérifier l'espace libéré
SELECT 
    pg_size_pretty(pg_database_size(current_database())) AS database_size;
```

### Résultats

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Espace index | 400 MB | 0 MB | **-400 MB** |
| Temps INSERT | +10% | Normal | ✅ |
| Temps UPDATE | +15% | Normal | ✅ |

---

## 📊 Points clés à retenir

1. **Toujours analyser avec EXPLAIN ANALYZE** avant d'optimiser
2. **Utiliser Dalibo** pour identifier les problèmes automatiquement
3. **Mesurer l'impact** avant et après chaque optimisation
4. **Index appropriés** : Solution la plus courante
5. **Éviter les sous-requêtes corrélées** : Utiliser JOIN
6. **Surveiller régulièrement** : Les problèmes évoluent

## 🔗 Prochain module

Passer au module [7. Exercices](../07-exercices/README.md) pour pratiquer avec des exercices guidés.

