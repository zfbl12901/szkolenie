# 7. Exercices pratiques

## 🎯 Objectifs

- Appliquer les connaissances acquises
- Résoudre des problèmes de performance
- Analyser et optimiser des requêtes réelles
- Utiliser Dalibo pour l'analyse

## 📋 Structure des exercices

Les exercices sont organisés par niveau de difficulté :
- 🟢 **Débutant** : Concepts de base
- 🟡 **Intermédiaire** : Techniques avancées
- 🔴 **Avancé** : Optimisations complexes

---

## Exercice 1 : Analyse d'une requête simple (🟢 Débutant)

### Contexte

Vous avez une table `products` avec 1 million de lignes et une requête lente :

```sql
SELECT * FROM products WHERE category = 'electronics';
```

### Tâches

1. **Analyser la requête** avec `EXPLAIN ANALYZE`
2. **Identifier le problème** dans le plan d'exécution
3. **Proposer une solution** avec un index
4. **Vérifier l'amélioration** avec `EXPLAIN ANALYZE`

### Données de test

```sql
-- Créer la table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    category TEXT,
    price DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insérer des données de test (1 million de lignes)
INSERT INTO products (name, category, price)
SELECT 
    'Product ' || i,
    CASE (i % 10)
        WHEN 0 THEN 'electronics'
        WHEN 1 THEN 'clothing'
        WHEN 2 THEN 'books'
        WHEN 3 THEN 'food'
        WHEN 4 THEN 'toys'
        WHEN 5 THEN 'electronics'
        WHEN 6 THEN 'furniture'
        WHEN 7 THEN 'electronics'
        WHEN 8 THEN 'sports'
        ELSE 'other'
    END,
    (RANDOM() * 1000)::DECIMAL(10,2)
FROM generate_series(1, 1000000) i;

-- Analyser la table
ANALYZE products;
```

### Solution attendue

<details>
<summary>Cliquez pour voir la solution</summary>

```sql
-- 1. Analyser la requête
EXPLAIN ANALYZE
SELECT * FROM products WHERE category = 'electronics';

-- Problème identifié : Seq Scan sur 1 million de lignes

-- 2. Créer un index
CREATE INDEX idx_products_category ON products(category);

-- 3. Analyser à nouveau
EXPLAIN ANALYZE
SELECT * FROM products WHERE category = 'electronics';

-- Résultat attendu : Index Scan avec temps d'exécution < 100ms
```
</details>

---

## Exercice 2 : Optimisation d'une jointure (🟡 Intermédiaire)

### Contexte

Vous avez deux tables `users` et `orders` avec une jointure lente :

```sql
SELECT 
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email
HAVING COUNT(o.id) > 5;
```

### Tâches

1. **Analyser la requête** avec `EXPLAIN ANALYZE`
2. **Identifier les problèmes** (scans, jointures, agrégations)
3. **Créer les index nécessaires**
4. **Optimiser la requête** (HAVING → WHERE si possible)
5. **Mesurer l'amélioration**

### Données de test

```sql
-- Créer les tables
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    amount DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insérer des données
INSERT INTO users (name, email, created_at)
SELECT 
    'User ' || i,
    'user' || i || '@example.com',
    NOW() - (RANDOM() * 365 || ' days')::INTERVAL
FROM generate_series(1, 100000) i;

INSERT INTO orders (user_id, amount, created_at)
SELECT 
    (RANDOM() * 100000)::INTEGER + 1,
    (RANDOM() * 1000)::DECIMAL(10,2),
    NOW() - (RANDOM() * 365 || ' days')::INTERVAL
FROM generate_series(1, 1000000) i;

ANALYZE users, orders;
```

### Solution attendue

<details>
<summary>Cliquez pour voir la solution</summary>

```sql
-- 1. Analyser la requête
EXPLAIN ANALYZE
SELECT 
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email
HAVING COUNT(o.id) > 5;

-- Problèmes identifiés :
-- - Pas d'index sur users.created_at
-- - Pas d'index sur orders.user_id
-- - HAVING peut être optimisé

-- 2. Créer les index
CREATE INDEX idx_users_created_at ON users(created_at);
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 3. Optimiser la requête (filtrer dans la sous-requête)
SELECT 
    u.name,
    u.email,
    o.order_count,
    o.total_amount
FROM users u
JOIN (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        SUM(amount) AS total_amount
    FROM orders
    GROUP BY user_id
    HAVING COUNT(*) > 5
) o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';

-- 4. Analyser à nouveau
EXPLAIN ANALYZE [requête optimisée];

-- Résultat attendu : Temps d'exécution réduit de 80%+
```
</details>

---

## Exercice 3 : Sous-requête corrélée (🟡 Intermédiaire)

### Contexte

Une requête avec sous-requêtes corrélées très lente :

```sql
SELECT 
    u.id,
    u.name,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count,
    (SELECT MAX(created_at) FROM orders o WHERE o.user_id = u.id) AS last_order_date,
    (SELECT AVG(amount) FROM orders o WHERE o.user_id = u.id) AS avg_order_amount
FROM users u
WHERE u.active = true;
```

### Tâches

1. **Analyser la requête** et identifier les sous-requêtes corrélées
2. **Convertir en JOIN** avec agrégation
3. **Créer les index nécessaires**
4. **Comparer les performances**

### Solution attendue

<details>
<summary>Cliquez pour voir la solution</summary>

```sql
-- 1. Analyser
EXPLAIN ANALYZE [requête originale];
-- Problème : 3 sous-requêtes exécutées pour chaque ligne

-- 2. Convertir en JOIN
SELECT 
    u.id,
    u.name,
    COALESCE(o.order_count, 0) AS order_count,
    o.last_order_date,
    COALESCE(o.avg_order_amount, 0) AS avg_order_amount
FROM users u
LEFT JOIN (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        MAX(created_at) AS last_order_date,
        AVG(amount) AS avg_order_amount
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id
WHERE u.active = true;

-- 3. Créer index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 4. Comparer
-- Résultat attendu : Amélioration de 95%+
```
</details>

---

## Exercice 4 : Analyse avec Dalibo (🔴 Avancé)

### Contexte

Vous devez analyser les performances d'une base de données de production (simulée) et identifier les problèmes principaux.

### Tâches

1. **Activer pg_stat_statements** et pg_qualstats
2. **Exécuter des requêtes de test** pour générer des statistiques
3. **Identifier les requêtes lentes** avec pg_stat_statements
4. **Identifier les index manquants** avec pg_qualstats
5. **Générer un rapport** avec recommandations

### Données de test

```sql
-- Utiliser les tables de l'exercice 2
-- Exécuter diverses requêtes pour générer des statistiques

-- Requêtes de test
SELECT * FROM users WHERE email = 'user50000@example.com';
SELECT * FROM orders WHERE user_id = 12345;
SELECT * FROM users WHERE created_at > '2024-06-01';
SELECT COUNT(*) FROM orders WHERE amount > 500;
```

### Solution attendue

<details>
<summary>Cliquez pour voir la solution</summary>

```sql
-- 1. Activer les extensions
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pg_qualstats;

-- 2. Exécuter les requêtes de test
[requêtes de test]

-- 3. Identifier les requêtes lentes
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    mean_exec_time,
    total_exec_time,
    (total_exec_time / SUM(total_exec_time) OVER ()) * 100 AS percent_total_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- 4. Identifier les index manquants
SELECT 
    qs.left_schema,
    qs.left_table,
    qs.left_column,
    qs.operator,
    COUNT(*) AS execution_count
FROM pg_qualstats qs
WHERE NOT EXISTS (
    SELECT 1
    FROM pg_index i
    JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
    WHERE i.indrelid = (qs.left_schema||'.'||qs.left_table)::regclass
    AND a.attname = qs.left_column
)
GROUP BY qs.left_schema, qs.left_table, qs.left_column, qs.operator
ORDER BY execution_count DESC;

-- 5. Générer des recommandations
SELECT 
    'CREATE INDEX idx_' || left_table || '_' || left_column || 
    ' ON ' || left_schema || '.' || left_table || 
    ' (' || left_column || ');' AS recommendation,
    execution_count AS priority
FROM [requête pg_qualstats ci-dessus]
ORDER BY execution_count DESC
LIMIT 5;
```
</details>

---

## Exercice 5 : Optimisation complète (🔴 Avancé)

### Contexte

Vous avez une application e-commerce avec des performances dégradées. Analysez et optimisez le système complet.

### Schéma de base de données

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    email TEXT,
    name TEXT,
    created_at TIMESTAMP,
    active BOOLEAN
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    category TEXT,
    price DECIMAL(10,2),
    stock INTEGER
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    created_at TIMESTAMP,
    status TEXT,
    total_amount DECIMAL(10,2)
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER,
    price DECIMAL(10,2)
);
```

### Requêtes à optimiser

1. **Requête de dashboard** :
```sql
SELECT 
    c.name,
    c.email,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent,
    MAX(o.created_at) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE c.active = true
GROUP BY c.id, c.name, c.email
HAVING COUNT(o.id) > 0
ORDER BY total_spent DESC
LIMIT 100;
```

2. **Requête de produits populaires** :
```sql
SELECT 
    p.name,
    p.category,
    SUM(oi.quantity) AS total_sold,
    SUM(oi.quantity * oi.price) AS total_revenue
FROM products p
JOIN order_items oi ON p.id = oi.product_id
JOIN orders o ON oi.order_id = o.id
WHERE o.created_at > '2024-01-01'
  AND o.status = 'completed'
GROUP BY p.id, p.name, p.category
ORDER BY total_sold DESC
LIMIT 50;
```

3. **Requête de recherche** :
```sql
SELECT 
    p.*,
    (SELECT COUNT(*) FROM order_items oi WHERE oi.product_id = p.id) AS times_ordered
FROM products p
WHERE p.category = 'electronics'
  AND p.price BETWEEN 100 AND 500
ORDER BY times_ordered DESC;
```

### Tâches

1. **Analyser chaque requête** avec EXPLAIN ANALYZE
2. **Identifier tous les problèmes**
3. **Créer un plan d'optimisation**
4. **Implémenter les optimisations**
5. **Mesurer l'impact global**

### Solution attendue

<details>
<summary>Cliquez pour voir la solution</summary>

```sql
-- 1. Analyser toutes les requêtes
EXPLAIN ANALYZE [chaque requête];

-- 2. Créer les index nécessaires
CREATE INDEX idx_customers_active ON customers(active);
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_created_at_status ON orders(created_at, status);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_products_category_price ON products(category, price);

-- 3. Optimiser la requête 1 (HAVING → WHERE)
SELECT 
    c.name,
    c.email,
    o.order_count,
    o.total_spent,
    o.last_order_date
FROM customers c
JOIN (
    SELECT 
        customer_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent,
        MAX(created_at) AS last_order_date
    FROM orders
    GROUP BY customer_id
) o ON c.id = o.customer_id
WHERE c.active = true
ORDER BY o.total_spent DESC
LIMIT 100;

-- 4. Optimiser la requête 3 (sous-requête → JOIN)
SELECT 
    p.*,
    COALESCE(oi.times_ordered, 0) AS times_ordered
FROM products p
LEFT JOIN (
    SELECT product_id, COUNT(*) AS times_ordered
    FROM order_items
    GROUP BY product_id
) oi ON p.id = oi.product_id
WHERE p.category = 'electronics'
  AND p.price BETWEEN 100 AND 500
ORDER BY oi.times_ordered DESC NULLS LAST;

-- 5. Mesurer l'impact
-- Comparer les temps d'exécution avant/après
```
</details>

---

## Exercice 6 : Monitoring et alertes (🔴 Avancé)

### Contexte

Créez un système de monitoring pour détecter automatiquement les problèmes de performance.

### Tâches

1. **Créer une vue** consolidant les métriques clés
2. **Créer une fonction** de détection d'alertes
3. **Créer un rapport** automatique
4. **Tester le système** avec des données réelles

### Solution attendue

<details>
<summary>Cliquez pour voir la solution</summary>

```sql
-- 1. Vue consolidée
CREATE OR REPLACE VIEW v_performance_metrics AS
SELECT 
    'slow_queries' AS metric_type,
    COUNT(*) AS count,
    AVG(mean_exec_time) AS avg_value,
    MAX(mean_exec_time) AS max_value
FROM pg_stat_statements
WHERE mean_exec_time > 1000

UNION ALL

SELECT 
    'cache_hit_ratio',
    NULL,
    ROUND(100.0 * SUM(shared_blks_hit) / 
          NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2),
    NULL
FROM pg_stat_statements

UNION ALL

SELECT 
    'idle_in_transaction',
    COUNT(*),
    NULL,
    NULL
FROM pg_stat_activity
WHERE state = 'idle in transaction';

-- 2. Fonction d'alertes
CREATE OR REPLACE FUNCTION check_performance_alerts()
RETURNS TABLE(alert_type TEXT, message TEXT, severity TEXT) AS $$
BEGIN
    -- Alertes sur requêtes lentes
    RETURN QUERY
    SELECT 
        'SLOW_QUERY'::TEXT,
        format('Query with mean_exec_time: %s ms', ROUND(mean_exec_time::numeric, 2)),
        CASE WHEN mean_exec_time > 5000 THEN 'CRITICAL' ELSE 'WARNING' END
    FROM pg_stat_statements
    WHERE mean_exec_time > 1000
    ORDER BY mean_exec_time DESC
    LIMIT 5;

    -- Alerte sur cache hit ratio
    RETURN QUERY
    SELECT 
        'LOW_CACHE_HIT'::TEXT,
        format('Cache hit ratio: %s%%', 
               ROUND(100.0 * SUM(shared_blks_hit) / 
                     NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2)),
        CASE 
            WHEN 100.0 * SUM(shared_blks_hit) / 
                 NULLIF(SUM(shared_blks_hit + shared_blks_read), 0) < 90 
            THEN 'CRITICAL'
            WHEN 100.0 * SUM(shared_blks_hit) / 
                 NULLIF(SUM(shared_blks_hit + shared_blks_read), 0) < 95 
            THEN 'WARNING'
            ELSE 'INFO'
        END
    FROM pg_stat_statements;

    -- Alerte sur connexions idle in transaction
    RETURN QUERY
    SELECT 
        'IDLE_IN_TRANSACTION'::TEXT,
        format('%s connections idle in transaction', COUNT(*)),
        CASE WHEN COUNT(*) > 10 THEN 'CRITICAL' ELSE 'WARNING' END
    FROM pg_stat_activity
    WHERE state = 'idle in transaction';
END;
$$ LANGUAGE plpgsql;

-- 3. Utilisation
SELECT * FROM check_performance_alerts();
```
</details>

---

## 📝 Conseils pour les exercices

1. **Toujours commencer par EXPLAIN ANALYZE** : Comprendre avant d'optimiser
2. **Mesurer avant et après** : Quantifier l'amélioration
3. **Tester avec des données réalistes** : Utiliser des volumes similaires à la production
4. **Documenter vos décisions** : Noter pourquoi vous avez choisi telle optimisation
5. **Vérifier les effets secondaires** : Un index améliore les SELECT mais ralentit les INSERT/UPDATE

## 🔗 Retour aux modules

- [Module 1 : Fondamentaux](../01-fondamentaux/README.md)
- [Module 2 : Plans d'exécution](../02-plans-execution/README.md)
- [Module 3 : Dalibo](../03-dalibo/README.md)
- [Module 4 : Indicateurs](../04-indicateurs/README.md)
- [Module 5 : Techniques](../05-techniques/README.md)
- [Module 6 : Cas pratiques](../06-cas-pratiques/README.md)

