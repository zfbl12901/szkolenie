# 3. Dalibo - Outil d'analyse de performance

## 🎯 Objectifs

- Comprendre l'écosystème Dalibo
- Installer et configurer les outils Dalibo
- Utiliser pg_stat_statements pour l'analyse
- Générer et interpréter les rapports de performance
- Utiliser les recommandations automatiques

## 📋 Table des matières

1. [Présentation de Dalibo](#présentation-de-dalibo)
2. [Installation et configuration](#installation-et-configuration)
3. [pg_stat_statements](#pg_stat_statements)
4. [pg_qualstats](#pg_qualstats)
5. [pg_stat_monitor](#pg_stat_monitor)
6. [Rapports et visualisations](#rapports-et-visualisations)
7. [Recommandations automatiques](#recommandations-automatiques)

---

## Présentation de Dalibo

### Écosystème Dalibo

Dalibo propose une suite d'outils open-source pour l'analyse de performance PostgreSQL :

- **pg_stat_statements** : Statistiques sur les requêtes SQL
- **pg_qualstats** : Statistiques sur les prédicats (WHERE, JOIN)
- **pg_stat_monitor** : Monitoring avancé avec agrégation temporelle
- **pg_wait_sampling** : Analyse des attentes
- **HypoPG** : Test d'index hypothétiques
- **pgBadger** : Analyse des logs PostgreSQL
- **pg_activity** : Monitoring en temps réel

### Avantages

✅ **Open-source** : Gratuit et modifiable
✅ **Complet** : Couvre tous les aspects de performance
✅ **Intégré** : Fonctionne avec PostgreSQL natif
✅ **Communauté** : Support actif et documentation

---

## Installation et configuration

### Prérequis

- PostgreSQL 12+ (certains outils nécessitent des versions spécifiques)
- Accès superutilisateur pour installer les extensions
- Compilateur C pour certaines extensions

### Installation de pg_stat_statements

**PostgreSQL 9.2+ :** Inclus par défaut

```sql
-- Activer l'extension
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Vérifier l'installation
SELECT * FROM pg_extension WHERE extname = 'pg_stat_statements';
```

**Configuration dans postgresql.conf :**

```ini
# Charger l'extension au démarrage
shared_preload_libraries = 'pg_stat_statements'

# Nombre de requêtes uniques à suivre (défaut: 10000)
pg_stat_statements.max = 10000

# Taille maximale de la requête stockée (défaut: 1024)
pg_stat_statements.track = all
pg_stat_statements.track_utility = on
pg_stat_statements.save = on
```

**Redémarrer PostgreSQL après modification**

### Installation de pg_qualstats

**Téléchargement et compilation :**

```bash
# Cloner le repository
git clone https://github.com/dalibo/pg_qualstats.git
cd pg_qualstats

# Compiler et installer
make
sudo make install
```

**Activation :**

```sql
-- Ajouter à shared_preload_libraries
-- Dans postgresql.conf:
-- shared_preload_libraries = 'pg_stat_statements,pg_qualstats'

-- Créer l'extension
CREATE EXTENSION IF NOT EXISTS pg_qualstats;

-- Vérifier
SELECT * FROM pg_extension WHERE extname = 'pg_qualstats';
```

### Installation de pg_stat_monitor

**Pour PostgreSQL 12+ :**

```bash
# Installation via package manager (exemple Ubuntu)
sudo apt-get install postgresql-14-pg-stat-monitor

# Ou compilation depuis source
git clone https://github.com/percona/pg_stat_monitor.git
cd pg_stat_monitor
make
sudo make install
```

**Activation :**

```sql
-- Dans postgresql.conf:
-- shared_preload_libraries = 'pg_stat_monitor'

CREATE EXTENSION IF NOT EXISTS pg_stat_monitor;
```

---

## pg_stat_statements

### Vue d'ensemble

`pg_stat_statements` collecte des statistiques sur toutes les requêtes SQL exécutées.

### Requêtes les plus coûteuses

```sql
-- Top 10 requêtes par temps total
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time,
    stddev_exec_time,
    rows,
    100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

### Requêtes les plus fréquentes

```sql
-- Top 10 requêtes par nombre d'appels
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    (total_exec_time / sum(total_exec_time) OVER ()) * 100 AS percent_total_time
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;
```

### Requêtes avec I/O élevé

```sql
-- Requêtes avec beaucoup de lectures disque
SELECT 
    query,
    calls,
    shared_blks_read,
    shared_blks_hit,
    shared_blks_dirtied,
    shared_blks_written,
    temp_blks_read,
    temp_blks_written
FROM pg_stat_statements
WHERE shared_blks_read > 1000
ORDER BY shared_blks_read DESC
LIMIT 10;
```

### Analyse détaillée d'une requête

```sql
-- Statistiques complètes pour une requête spécifique
SELECT 
    query,
    calls,
    total_exec_time,
    min_exec_time,
    max_exec_time,
    mean_exec_time,
    stddev_exec_time,
    rows,
    shared_blks_hit,
    shared_blks_read,
    shared_blks_dirtied,
    shared_blks_written,
    temp_blks_read,
    temp_blks_written,
    blk_read_time,
    blk_write_time
FROM pg_stat_statements
WHERE query LIKE '%SELECT * FROM users%'
ORDER BY total_exec_time DESC;
```

### Réinitialiser les statistiques

```sql
-- Réinitialiser toutes les statistiques
SELECT pg_stat_statements_reset();

-- Réinitialiser pour une base spécifique
SELECT pg_stat_statements_reset(userid, dbid, queryid);
```

### Normalisation des requêtes

`pg_stat_statements` normalise les requêtes en remplaçant les valeurs par `$1`, `$2`, etc.

**Exemple :**
```sql
-- Requête originale
SELECT * FROM users WHERE id = 123;

-- Normalisée dans pg_stat_statements
SELECT * FROM users WHERE id = $1;
```

**Avantage :** Regroupe les requêtes similaires avec des paramètres différents.

---

## pg_qualstats

### Vue d'ensemble

`pg_qualstats` collecte des statistiques sur les **prédicats** (conditions WHERE, JOIN) pour identifier les index manquants.

### Statistiques sur les prédicats

```sql
-- Top prédicats les plus utilisés
SELECT 
    left_schema,
    left_table,
    left_column,
    operator,
    count(*) AS execution_count,
    n_distinct,
    most_common_vals
FROM pg_qualstats
GROUP BY left_schema, left_table, left_column, operator
ORDER BY execution_count DESC
LIMIT 20;
```

### Identification d'index manquants

```sql
-- Prédicats sans index correspondant
SELECT 
    qs.left_schema,
    qs.left_table,
    qs.left_column,
    qs.operator,
    qs.execution_count,
    pg_size_pretty(pg_relation_size(qs.left_schema||'.'||qs.left_table)) AS table_size
FROM pg_qualstats qs
WHERE NOT EXISTS (
    SELECT 1
    FROM pg_index i
    JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
    WHERE i.indrelid = (qs.left_schema||'.'||qs.left_table)::regclass
    AND a.attname = qs.left_column
)
ORDER BY qs.execution_count DESC
LIMIT 20;
```

### Recommandations d'index

```sql
-- Générer des commandes CREATE INDEX
SELECT 
    'CREATE INDEX idx_' || 
    left_table || '_' || 
    left_column || 
    ' ON ' || left_schema || '.' || left_table || 
    ' (' || left_column || ');' AS create_index_command,
    execution_count,
    n_distinct
FROM (
    SELECT 
        qs.left_schema,
        qs.left_table,
        qs.left_column,
        COUNT(*) AS execution_count,
        COUNT(DISTINCT qs.most_common_vals) AS n_distinct
    FROM pg_qualstats qs
    WHERE NOT EXISTS (
        SELECT 1
        FROM pg_index i
        JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
        WHERE i.indrelid = (qs.left_schema||'.'||qs.left_table)::regclass
        AND a.attname = qs.left_column
    )
    GROUP BY qs.left_schema, qs.left_table, qs.left_column
) AS missing_indexes
ORDER BY execution_count DESC
LIMIT 10;
```

### Réinitialiser les statistiques

```sql
-- Réinitialiser pg_qualstats
SELECT pg_qualstats_reset();
```

---

## pg_stat_monitor

### Vue d'ensemble

`pg_stat_monitor` offre un monitoring avancé avec agrégation temporelle et analyse des buckets.

### Configuration

```sql
-- Voir la configuration
SELECT * FROM pg_stat_monitor_settings;

-- Modifier la configuration
ALTER SYSTEM SET pg_stat_monitor.pgsm_max_buckets = 10;
SELECT pg_reload_conf();
```

### Requêtes par bucket (période)

```sql
-- Requêtes groupées par période
SELECT 
    bucket,
    bucket_start_time,
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time
FROM pg_stat_monitor
ORDER BY bucket DESC, total_exec_time DESC
LIMIT 20;
```

### Analyse des erreurs

```sql
-- Requêtes avec erreurs
SELECT 
    query,
    calls,
    errors,
    error_count,
    error_code
FROM pg_stat_monitor
WHERE errors > 0
ORDER BY errors DESC;
```

### Analyse des plans

```sql
-- Plans d'exécution les plus utilisés
SELECT 
    query,
    planid,
    calls,
    mean_exec_time,
    plans
FROM pg_stat_monitor
WHERE plans IS NOT NULL
ORDER BY calls DESC
LIMIT 10;
```

---

## Rapports et visualisations

### pgBadger - Analyse des logs

**Installation :**

```bash
# Ubuntu/Debian
sudo apt-get install pgbadger

# Ou via Perl CPAN
cpanm pgbadger
```

**Génération de rapport :**

```bash
# Générer un rapport HTML
pgbadger /var/log/postgresql/postgresql-*.log -o report.html

# Avec options avancées
pgbadger \
  --prefix '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h' \
  --outdir /var/www/pgbadger \
  /var/log/postgresql/postgresql-*.log
```

**Configuration PostgreSQL pour pgBadger :**

```ini
# Dans postgresql.conf
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 0
log_autovacuum_min_duration = 0
log_error_verbosity = default
log_min_duration_statement = 1000  # Log les requêtes > 1s
```

### pg_activity - Monitoring temps réel

**Installation :**

```bash
pip install pg_activity
```

**Utilisation :**

```bash
# Connexion simple
pg_activity -U postgres -d mydb

# Avec options
pg_activity -U postgres -d mydb --refresh 2 --no-database-size
```

### Visualisation avec Metabase/Grafana

**Intégration avec Grafana :**

1. Installer le plugin PostgreSQL
2. Créer des dashboards avec les vues système
3. Surveiller les métriques en temps réel

**Requêtes utiles pour Grafana :**

```sql
-- Temps d'exécution moyen par minute
SELECT 
    date_trunc('minute', now()) AS time,
    AVG(mean_exec_time) AS avg_exec_time
FROM pg_stat_statements
GROUP BY time;
```

---

## Recommandations automatiques

### Script de recommandations basique

```sql
-- Requêtes lentes sans index approprié
WITH slow_queries AS (
    SELECT 
        query,
        calls,
        mean_exec_time,
        total_exec_time
    FROM pg_stat_statements
    WHERE mean_exec_time > 100  -- > 100ms
    ORDER BY total_exec_time DESC
    LIMIT 10
),
missing_indexes AS (
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
)
SELECT 
    'MISSING INDEX' AS recommendation_type,
    'CREATE INDEX idx_' || left_table || '_' || left_column || 
    ' ON ' || left_schema || '.' || left_table || 
    ' (' || left_column || ');' AS recommendation,
    execution_count AS priority_score
FROM missing_indexes
ORDER BY execution_count DESC
LIMIT 10;
```

### Utiliser HypoPG pour tester les index

**Installation :**

```sql
CREATE EXTENSION IF NOT EXISTS hypopg;
```

**Tester un index hypothétique :**

```sql
-- Créer un index hypothétique
SELECT * FROM hypopg_create_index('CREATE INDEX ON users(email)');

-- Voir les index hypothétiques
SELECT * FROM hypopg_list_indexes();

-- Tester le plan avec l'index hypothétique
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Supprimer l'index hypothétique
SELECT hypopg_drop_index(oid) FROM hypopg_list_indexes();
```

---

## 📊 Points clés à retenir

1. **pg_stat_statements** : Essentiel pour identifier les requêtes lentes
2. **pg_qualstats** : Identifie les index manquants automatiquement
3. **pg_stat_monitor** : Monitoring avancé avec agrégation temporelle
4. **pgBadger** : Analyse complète des logs PostgreSQL
5. **HypoPG** : Teste les index avant de les créer

## 🔗 Prochain module

Passer au module [4. Indicateurs de performance](../04-indicateurs/README.md) pour apprendre à interpréter les indicateurs clés de performance.

