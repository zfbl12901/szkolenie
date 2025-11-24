# 1. Fondamentaux de l'optimisation PostgreSQL

## 🎯 Objectifs

- Comprendre l'architecture PostgreSQL et le planificateur de requêtes
- Maîtriser les différents types d'index et leur utilisation optimale
- Comprendre le rôle des statistiques dans l'optimisation

## 📋 Table des matières

1. [Architecture PostgreSQL](#architecture-postgresql)
2. [Le planificateur de requêtes](#le-planificateur-de-requêtes)
3. [Types d'index](#types-dindex)
4. [Statistiques et ANALYZE](#statistiques-et-analyze)

---

## Architecture PostgreSQL

### Composants clés

PostgreSQL utilise une architecture multi-processus avec plusieurs composants importants :

- **Postmaster** : Processus principal qui gère les connexions
- **Backend processes** : Un processus par connexion client
- **Planificateur (Planner)** : Optimise les requêtes SQL
- **Exécuteur (Executor)** : Exécute les plans de requêtes

### Flux d'exécution d'une requête

```
Requête SQL
    ↓
Parser (analyse syntaxique)
    ↓
Rewriter (réécriture des vues/règles)
    ↓
Planner (génération du plan d'exécution)
    ↓
Executor (exécution du plan)
    ↓
Résultat
```

### Le planificateur de requêtes

Le planificateur est responsable de :
- Choisir le meilleur plan d'exécution
- Estimer les coûts de chaque opération
- Utiliser les statistiques de la base de données
- Optimiser les jointures, tri, agrégations

**Facteurs influençant le planificateur :**
- Statistiques des tables (`pg_stat_user_tables`)
- Statistiques des colonnes (`pg_stats`)
- Configuration des coûts (`random_page_cost`, `seq_page_cost`, etc.)
- Paramètres de mémoire (`work_mem`, `shared_buffers`)

---

## Le planificateur de requêtes

### Paramètres de coût

PostgreSQL utilise un système de coûts pour comparer les plans :

```sql
-- Voir les paramètres de coût actuels
SHOW random_page_cost;
SHOW seq_page_cost;
SHOW cpu_tuple_cost;
SHOW cpu_index_tuple_cost;
```

**Paramètres importants :**
- `seq_page_cost` : Coût de lecture séquentielle (défaut: 1.0)
- `random_page_cost` : Coût de lecture aléatoire (défaut: 4.0)
- `cpu_tuple_cost` : Coût de traitement d'une ligne (défaut: 0.01)
- `cpu_index_tuple_cost` : Coût d'utilisation d'un index (défaut: 0.005)

### Estimation des coûts

Le planificateur estime :
- **Nombre de lignes** : Basé sur les statistiques
- **Coût d'E/S** : Lecture/écriture disque
- **Coût CPU** : Traitement des données
- **Coût total** : Somme des coûts

**Limitation importante :** Les estimations peuvent être imprécises si les statistiques sont obsolètes.

---

## Types d'index

### Index B-tree (par défaut)

**Utilisation :**
- Égalité (`=`)
- Comparaisons (`<`, `>`, `<=`, `>=`)
- BETWEEN, IN
- LIKE avec préfixe fixe

**Exemple :**
```sql
CREATE INDEX idx_user_email ON users(email);
-- Utilisé pour: WHERE email = 'user@example.com'
```

### Index Hash

**Utilisation :**
- Uniquement pour l'égalité (`=`)
- Plus rapide que B-tree pour l'égalité simple
- Ne supporte pas les comparaisons

**Exemple :**
```sql
CREATE INDEX idx_user_id_hash ON users USING hash(id);
-- Utilisé pour: WHERE id = 123
```

### Index GIN (Generalized Inverted Index)

**Utilisation :**
- Types de données complexes (tableaux, JSONB, full-text)
- Opérateurs de recherche avancés

**Exemple :**
```sql
CREATE INDEX idx_product_tags_gin ON products USING gin(tags);
-- Utilisé pour: WHERE tags @> ARRAY['electronics']
```

### Index GiST (Generalized Search Tree)

**Utilisation :**
- Types géométriques
- Full-text search
- Types personnalisés

**Exemple :**
```sql
CREATE INDEX idx_location_gist ON places USING gist(location);
-- Utilisé pour: WHERE location <-> point(0,0) < 1000
```

### Index BRIN (Block Range Index)

**Utilisation :**
- Grandes tables avec données triées
- Très compact (peu d'espace)
- Efficace pour les plages de valeurs

**Exemple :**
```sql
CREATE INDEX idx_orders_date_brin ON orders USING brin(order_date);
-- Utilisé pour: WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'
```

### Index partiel

**Utilisation :**
- Réduire la taille de l'index
- Améliorer les performances pour des conditions spécifiques

**Exemple :**
```sql
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
-- Index uniquement sur les utilisateurs actifs
```

### Index composite

**Utilisation :**
- Plusieurs colonnes
- Ordre des colonnes important

**Exemple :**
```sql
CREATE INDEX idx_user_name_email ON users(last_name, first_name, email);
-- Utilisé pour: WHERE last_name = 'Doe' AND first_name = 'John'
```

**Règle importante :** L'index peut être utilisé si la requête utilise les colonnes dans l'ordre de l'index, en commençant par la première.

---

## Statistiques et ANALYZE

### Pourquoi les statistiques sont essentielles

Le planificateur utilise les statistiques pour :
- Estimer le nombre de lignes retournées
- Choisir entre différents plans d'exécution
- Déterminer l'ordre des jointures

### Collecte des statistiques

```sql
-- Analyser une table spécifique
ANALYZE table_name;

-- Analyser toutes les tables
ANALYZE;

-- Analyser avec un niveau de détail
ANALYZE VERBOSE table_name;
```

**Quand exécuter ANALYZE :**
- Après des modifications importantes (INSERT, UPDATE, DELETE)
- Après la création d'index
- Automatiquement par autovacuum (configurable)

### Configuration autovacuum

```sql
-- Voir la configuration actuelle
SHOW autovacuum;
SHOW autovacuum_analyze_scale_factor;
SHOW autovacuum_analyze_threshold;

-- Modifier pour une table spécifique
ALTER TABLE large_table SET (
    autovacuum_analyze_scale_factor = 0.05,
    autovacuum_analyze_threshold = 10000
);
```

### Consulter les statistiques

```sql
-- Statistiques des tables
SELECT 
    schemaname,
    tablename,
    n_tup_ins AS inserts,
    n_tup_upd AS updates,
    n_tup_del AS deletes,
    n_live_tup AS live_rows,
    n_dead_tup AS dead_rows,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;

-- Statistiques des colonnes
SELECT 
    schemaname,
    tablename,
    attname AS column_name,
    n_distinct,
    correlation,
    most_common_vals
FROM pg_stats
WHERE tablename = 'your_table';
```

### Statistiques étendues

PostgreSQL 10+ supporte les statistiques étendues :

```sql
-- Créer des statistiques sur plusieurs colonnes
CREATE STATISTICS stats_user_name_email 
ON users(last_name, first_name);

-- Analyser pour collecter les statistiques
ANALYZE users;

-- Consulter les statistiques étendues
SELECT * FROM pg_statistic_ext;
```

**Utilité :** Améliore les estimations pour les requêtes avec plusieurs colonnes corrélées.

---

## 📊 Points clés à retenir

1. **Le planificateur dépend des statistiques** : Des statistiques obsolètes = mauvais plans
2. **Choisir le bon type d'index** : Chaque type a ses avantages
3. **ANALYZE régulier** : Essentiel pour maintenir de bonnes performances
4. **Comprendre les coûts** : Aide à interpréter les plans d'exécution

## 🔗 Prochain module

Passer au module [2. Analyse des plans d'exécution](../02-plans-execution/README.md) pour apprendre à interpréter les plans d'exécution.

