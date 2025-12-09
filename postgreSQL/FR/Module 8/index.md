# Module 8 — Monitoring, Outils & Maintenance

## 🎯 Objectifs
- Superviser, maintenir et diagnostiquer un cluster PostgreSQL en production pour assurer performance et disponibilité.

---

## 📚 Contenu

### 1. Outils de monitoring

#### 1.1 `pg_stat_statements`
- Extension pour suivre les requêtes SQL les plus coûteuses.
```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
SELECT query, calls, total_time, rows
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

#### 1.2 `pg_stat_activity`
- Visualiser les connexions actives et les transactions en cours.
```sql
SELECT pid, usename, state, query, wait_event_type, wait_event
FROM pg_stat_activity;
```

#### 1.3 pgBadger
- Analyse des logs PostgreSQL pour générer des rapports HTML détaillés.

#### Commande exemple :
```bash
pgbadger /var/log/postgresql/postgresql.log -o report.html
```

#### 1.4 Intégration avec systèmes de monitoring externes
- **Grafana + Prometheus** : dashboards temps réel pour métriques de cluster.
- **Datadog / New Relic** : monitoring avancé et alerting cloud.

---

### 2. Sauvegarde & restauration

#### 2.1 `pg_dump` & `pg_restore`
- Dump logique pour table/base complète.
```bash
pg_dump -Fc -f db.dump mydatabase
pg_restore -d mydatabase db.dump
```

#### 2.2 `pg_basebackup`
- Backup physique pour réplication ou restauration rapide.
```bash
pg_basebackup -D /backup/dir -F tar -z -P
```

#### 2.3 Snapshots filesystem
- Sauvegarde instantanée sur systèmes supportant snapshot (LVM, ZFS, cloud).
- Avantages : rapide, compatible avec réplication, cohérence des fichiers.

---

### 3. Maintenance proactive

#### 3.1 Surveillance des index
- Identifier les index inutilisés ou doublons.
```sql
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
```

#### 3.2 Bloat et fragmentation
- Mesurer la taille réelle vs estimée des tables/index.
- Extensions/utilitaires recommandés : `pgstattuple`, `pg_repack`.
```sql
SELECT * FROM pgstattuple('mytable');
```

#### 3.3 Autovacuum tuning
- Ajuster autovacuum pour éviter bloat et perte de performance.
- Paramètres : `autovacuum_vacuum_cost_delay`, `autovacuum_max_workers`, etc.

---

## ✅ Compétences attendues à l'issue du module
- Surveiller les requêtes et l'activité des transactions avec `pg_stat_statements` et `pg_stat_activity`.
- Analyser et générer des rapports de logs PostgreSQL avec pgBadger.
- Intégrer PostgreSQL à des outils de monitoring avancés (Grafana, Prometheus, Datadog).
- Effectuer sauvegardes et restaurations logiques et physiques.
- Maintenir le cluster en état optimal : index, bloat, fragmentation, autovacuum.
