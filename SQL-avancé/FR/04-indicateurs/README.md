# 4. Indicateurs de performance

## 🎯 Objectifs

- Identifier les métriques clés à surveiller
- Interpréter les indicateurs Dalibo
- Définir des seuils d'alerte
- Comprendre les corrélations entre indicateurs

## 📋 Table des matières

1. [Métriques système](#métriques-système)
2. [Métriques de requêtes](#métriques-de-requêtes)
3. [Métriques d'index](#métriques-dindex)
4. [Métriques d'I/O](#métriques-dio)
5. [Seuils d'alerte](#seuils-dalerte)
6. [Tableau de bord des indicateurs](#tableau-de-bord-des-indicateurs)

---

## Métriques système

### Utilisation CPU

```sql
-- Voir l'utilisation CPU par processus
SELECT 
    pid,
    usename,
    application_name,
    state,
    query_start,
    state_change,
    wait_event_type,
    wait_event,
    query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY query_start;
```

**Indicateurs clés :**
- **CPU élevé** : Requêtes en cours d'exécution
- **wait_event_type = CPU** : Processus en attente CPU

### Utilisation mémoire

```sql
-- Mémoire partagée utilisée
SELECT 
    setting AS shared_buffers,
    pg_size_pretty(setting::bigint * 8192) AS shared_buffers_size
FROM pg_settings
WHERE name = 'shared_buffers';

-- Mémoire de travail utilisée
SELECT 
    setting AS work_mem,
    pg_size_pretty(setting::bigint * 1024) AS work_mem_size
FROM pg_settings
WHERE name = 'work_mem';

-- Statistiques de mémoire
SELECT 
    name,
    setting,
    unit,
    short_desc
FROM pg_settings
WHERE name IN ('shared_buffers', 'work_mem', 'maintenance_work_mem', 'effective_cache_size')
ORDER BY name;
```

**Indicateurs clés :**
- **shared_buffers** : Cache partagé (recommandé: 25% RAM)
- **work_mem** : Mémoire par opération de tri/hash
- **effective_cache_size** : Estimation du cache OS (recommandé: 50-75% RAM)

### Connexions actives

```sql
-- Nombre de connexions par état
SELECT 
    state,
    COUNT(*) AS count,
    COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () AS percent
FROM pg_stat_activity
GROUP BY state
ORDER BY count DESC;

-- Connexions par base de données
SELECT 
    datname,
    COUNT(*) AS connections,
    COUNT(*) FILTER (WHERE state = 'active') AS active,
    COUNT(*) FILTER (WHERE state = 'idle') AS idle,
    COUNT(*) FILTER (WHERE state = 'idle in transaction') AS idle_in_transaction
FROM pg_stat_activity
WHERE datname IS NOT NULL
GROUP BY datname
ORDER BY connections DESC;

-- Limite de connexions
SELECT 
    setting AS max_connections,
    (SELECT COUNT(*) FROM pg_stat_activity) AS current_connections,
    ROUND(100.0 * (SELECT COUNT(*) FROM pg_stat_activity) / setting::numeric, 2) AS percent_used
FROM pg_settings
WHERE name = 'max_connections';
```

**Indicateurs clés :**
- **max_connections** : Limite configurée
- **idle in transaction** : Connexions bloquantes (alerte si > 5%)
- **active** : Requêtes en cours

### Verrous (Locks)

```sql
-- Verrous en attente
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

**Indicateurs clés :**
- **Verrous en attente** : Blocages actifs (alerte si > 0)
- **Durée des verrous** : Temps d'attente (alerte si > 1s)

---

## Métriques de requêtes

### Temps d'exécution

```sql
-- Temps d'exécution moyen par requête (top 20)
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    ROUND(total_exec_time::numeric, 2) AS total_time_ms,
    ROUND(mean_exec_time::numeric, 2) AS mean_time_ms,
    ROUND(max_exec_time::numeric, 2) AS max_time_ms,
    ROUND(stddev_exec_time::numeric, 2) AS stddev_time_ms,
    ROUND((total_exec_time / SUM(total_exec_time) OVER ()) * 100, 2) AS percent_total_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

**Indicateurs clés :**
- **mean_exec_time** : Temps moyen (alerte si > 1000ms)
- **max_exec_time** : Temps maximum (alerte si > 5000ms)
- **stddev_exec_time** : Variabilité (alerte si stddev > mean)

### Fréquence d'exécution

```sql
-- Requêtes les plus fréquentes
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    ROUND(mean_exec_time::numeric, 2) AS mean_time_ms,
    ROUND((calls * mean_exec_time)::numeric, 2) AS total_time_ms,
    ROUND((calls::numeric / (SELECT SUM(calls) FROM pg_stat_statements)) * 100, 2) AS percent_calls
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 20;
```

**Indicateurs clés :**
- **calls** : Nombre d'appels (identifier les requêtes N+1)
- **percent_calls** : Pourcentage du total (alerte si > 50% pour une seule requête)

### Efficacité du cache

```sql
-- Taux de hit du cache par requête
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    shared_blks_hit,
    shared_blks_read,
    ROUND(100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0), 2) AS cache_hit_ratio,
    ROUND((shared_blks_read * 8)::numeric / 1024, 2) AS disk_read_mb
FROM pg_stat_statements
WHERE shared_blks_hit + shared_blks_read > 0
ORDER BY shared_blks_read DESC
LIMIT 20;
```

**Indicateurs clés :**
- **cache_hit_ratio** : Taux de hit (objectif: > 95%)
- **disk_read_mb** : Lectures disque (alerte si > 100MB)

### Requêtes avec I/O temporaire

```sql
-- Requêtes utilisant des fichiers temporaires
SELECT 
    LEFT(query, 100) AS query_preview,
    calls,
    temp_blks_read,
    temp_blks_written,
    ROUND((temp_blks_read + temp_blks_written) * 8.0 / 1024, 2) AS temp_mb,
    ROUND(mean_exec_time::numeric, 2) AS mean_time_ms
FROM pg_stat_statements
WHERE temp_blks_read > 0 OR temp_blks_written > 0
ORDER BY (temp_blks_read + temp_blks_written) DESC
LIMIT 20;
```

**Indicateurs clés :**
- **temp_blks_read/written** : I/O temporaire (alerte si > 0)
- **Action** : Augmenter `work_mem` si présent

---

## Métriques d'index

### Utilisation des index

```sql
-- Statistiques d'utilisation des index
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS index_scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS tuples_fetched,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC, pg_relation_size(indexrelid) DESC
LIMIT 20;
```

**Indicateurs clés :**
- **idx_scan = 0** : Index non utilisé (candidat à suppression)
- **index_size** : Taille de l'index (coût de maintenance)

### Index manquants (via pg_qualstats)

```sql
-- Prédicats fréquents sans index
SELECT 
    qs.left_schema,
    qs.left_table,
    qs.left_column,
    qs.operator,
    COUNT(*) AS execution_count,
    pg_size_pretty(pg_relation_size((qs.left_schema||'.'||qs.left_table)::regclass)) AS table_size
FROM pg_qualstats qs
WHERE NOT EXISTS (
    SELECT 1
    FROM pg_index i
    JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
    WHERE i.indrelid = (qs.left_schema||'.'||qs.left_table)::regclass
    AND a.attname = qs.left_column
)
GROUP BY qs.left_schema, qs.left_table, qs.left_column, qs.operator
ORDER BY execution_count DESC
LIMIT 20;
```

**Indicateurs clés :**
- **execution_count** : Fréquence d'utilisation (priorité)
- **table_size** : Taille de la table (impact de l'index)

### Bloat des index

```sql
-- Détection du bloat des index
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans,
    CASE 
        WHEN idx_scan = 0 THEN 'UNUSED'
        WHEN idx_scan < 100 THEN 'LOW'
        ELSE 'OK'
    END AS usage_status
FROM pg_stat_user_indexes
WHERE pg_relation_size(indexrelid) > 1048576  -- > 1MB
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Indicateurs clés :**
- **UNUSED** : Index jamais utilisé (candidat à suppression)
- **LOW** : Index peu utilisé (à évaluer)

---

## Métriques d'I/O

### Statistiques d'I/O par table

```sql
-- I/O par table
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins AS inserts,
    n_tup_upd AS updates,
    n_tup_del AS deletes,
    n_live_tup AS live_rows,
    n_dead_tup AS dead_rows,
    ROUND(n_dead_tup::numeric / NULLIF(n_live_tup + n_dead_tup, 0) * 100, 2) AS dead_tuple_percent,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY seq_scan DESC
LIMIT 20;
```

**Indicateurs clés :**
- **seq_scan élevé** : Beaucoup de scans séquentiels (créer des index)
- **dead_tuple_percent** : Pourcentage de tuples morts (alerte si > 10%)
- **last_vacuum** : Dernier vacuum (alerte si > 7 jours)

### Statistiques d'I/O globales

```sql
-- Statistiques d'I/O globales
SELECT 
    datname,
    blks_read,
    blks_hit,
    ROUND(100.0 * blks_hit / NULLIF(blks_hit + blks_read, 0), 2) AS cache_hit_ratio,
    tup_returned,
    tup_fetched,
    tup_inserted,
    tup_updated,
    tup_deleted
FROM pg_stat_database
WHERE datname NOT IN ('template0', 'template1', 'postgres')
ORDER BY blks_read DESC;
```

**Indicateurs clés :**
- **cache_hit_ratio** : Taux de hit global (objectif: > 95%)
- **blks_read** : Lectures disque (alerte si élevé)

---

## Seuils d'alerte

### Tableau des seuils recommandés

| Métrique | Seuil d'alerte | Seuil critique | Action |
|----------|----------------|----------------|--------|
| **Temps d'exécution moyen** | > 1000ms | > 5000ms | Analyser le plan, créer index |
| **Taux de hit cache** | < 95% | < 90% | Augmenter shared_buffers |
| **Tuples morts** | > 10% | > 20% | Exécuter VACUUM |
| **I/O temporaire** | > 0 | > 100MB | Augmenter work_mem |
| **Connexions idle in transaction** | > 5% | > 10% | Identifier et corriger |
| **Verrous en attente** | > 0 | > 10 | Analyser les blocages |
| **Index non utilisés** | Taille > 100MB | Taille > 1GB | Évaluer suppression |
| **Dernier VACUUM** | > 7 jours | > 30 jours | Configurer autovacuum |

### Requête de monitoring global

```sql
-- Dashboard de santé PostgreSQL
WITH metrics AS (
    SELECT 
        (SELECT COUNT(*) FROM pg_stat_activity WHERE state = 'active') AS active_queries,
        (SELECT COUNT(*) FROM pg_stat_activity WHERE state = 'idle in transaction') AS idle_in_transaction,
        (SELECT COUNT(*) FROM pg_locks WHERE NOT granted) AS waiting_locks,
        (SELECT AVG(mean_exec_time) FROM pg_stat_statements) AS avg_query_time,
        (SELECT SUM(shared_blks_read) FROM pg_stat_statements) AS total_disk_reads,
        (SELECT SUM(shared_blks_hit) FROM pg_stat_statements) AS total_cache_hits,
        (SELECT COUNT(*) FROM pg_stat_user_indexes WHERE idx_scan = 0) AS unused_indexes,
        (SELECT SUM(n_dead_tup)::numeric / NULLIF(SUM(n_live_tup + n_dead_tup), 0) * 100 
         FROM pg_stat_user_tables) AS dead_tuple_percent
)
SELECT 
    active_queries,
    idle_in_transaction,
    waiting_locks,
    ROUND(avg_query_time::numeric, 2) AS avg_query_time_ms,
    ROUND(100.0 * total_cache_hits / NULLIF(total_cache_hits + total_disk_reads, 0), 2) AS cache_hit_ratio,
    unused_indexes,
    ROUND(dead_tuple_percent::numeric, 2) AS dead_tuple_percent,
    CASE 
        WHEN idle_in_transaction > (SELECT setting::int FROM pg_settings WHERE name = 'max_connections') * 0.1 
        THEN 'ALERT'
        WHEN waiting_locks > 0 THEN 'WARNING'
        WHEN avg_query_time > 1000 THEN 'WARNING'
        WHEN dead_tuple_percent > 10 THEN 'WARNING'
        ELSE 'OK'
    END AS health_status
FROM metrics;
```

---

## Tableau de bord des indicateurs

### Vue consolidée pour Dalibo

```sql
-- Vue consolidée des indicateurs clés
CREATE OR REPLACE VIEW v_performance_dashboard AS
SELECT 
    'QUERIES' AS category,
    COUNT(*) AS metric_count,
    ROUND(AVG(mean_exec_time)::numeric, 2) AS avg_value,
    ROUND(MAX(mean_exec_time)::numeric, 2) AS max_value,
    'ms' AS unit
FROM pg_stat_statements
WHERE mean_exec_time > 100

UNION ALL

SELECT 
    'CACHE_HIT',
    NULL,
    ROUND(100.0 * SUM(shared_blks_hit) / NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2),
    NULL,
    '%'
FROM pg_stat_statements

UNION ALL

SELECT 
    'IDLE_IN_TRANSACTION',
    COUNT(*),
    NULL,
    NULL,
    'count'
FROM pg_stat_activity
WHERE state = 'idle in transaction'

UNION ALL

SELECT 
    'WAITING_LOCKS',
    COUNT(*),
    NULL,
    NULL,
    'count'
FROM pg_locks
WHERE NOT granted

UNION ALL

SELECT 
    'UNUSED_INDEXES',
    COUNT(*),
    NULL,
    NULL,
    'count'
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND pg_relation_size(indexrelid) > 1048576;  -- > 1MB

-- Utilisation
SELECT * FROM v_performance_dashboard;
```

### Export pour monitoring externe

```sql
-- Export JSON pour outils de monitoring
SELECT json_build_object(
    'timestamp', NOW(),
    'metrics', json_build_object(
        'active_queries', (SELECT COUNT(*) FROM pg_stat_activity WHERE state = 'active'),
        'cache_hit_ratio', (
            SELECT ROUND(100.0 * SUM(shared_blks_hit) / NULLIF(SUM(shared_blks_hit + shared_blks_read), 0), 2)
            FROM pg_stat_statements
        ),
        'avg_query_time_ms', (
            SELECT ROUND(AVG(mean_exec_time)::numeric, 2)
            FROM pg_stat_statements
        ),
        'unused_indexes', (
            SELECT COUNT(*) FROM pg_stat_user_indexes WHERE idx_scan = 0
        )
    )
);
```

---

## 📊 Points clés à retenir

1. **Surveiller régulièrement** : Les métriques changent avec le temps
2. **Seuils contextuels** : Adapter selon l'environnement
3. **Corrélations** : Analyser plusieurs métriques ensemble
4. **Tendances** : Surveiller l'évolution dans le temps
5. **Actions prioritaires** : Agir sur les métriques critiques d'abord

## 🔗 Prochain module

Passer au module [5. Techniques d'optimisation](../05-techniques/README.md) pour apprendre les techniques pratiques d'optimisation.

