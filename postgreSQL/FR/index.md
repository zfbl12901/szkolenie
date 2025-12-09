# 📘 Plan de cours — PostgreSQL Avancé
**Data Analyst → Data Engineer**

---

# Formation PostgreSQL Avancé pour Data Engineer

Cette formation est conçue pour les Data Analysts souhaitant monter en compétences vers le Data Engineering, avec un focus sur l’optimisation SQL, la performance, la modélisation BI et la mise en place de pipelines de données.

---

## Module 1 — Optimisation des requêtes

### 🎯 Objectifs
- Comprendre comment PostgreSQL exécute réellement une requête.
- Diagnostiquer et optimiser les performances SQL sur de gros volumes.

### 📚 Contenu
- **EXPLAIN / EXPLAIN ANALYZE** : lecture et interprétation des plans d’exécution.
- Analyse des coûts, row estimates, loops, timings.
- Types de joins : nested loop, hash join, merge join.
- Scans : seq scan, index scan, bitmap scan.
- **Indexation avancée** : multicolonnes, partiels, expressions, GIN/GiST/BRIN.
- **Partitionnement** et bonnes pratiques.
- **CTE** : matérialisés vs non matérialisés.
- **Analyse visuelle** avec explain.dalibo.com.

---

## Module 2 — Performance & Scalabilité

### 🎯 Objectifs
- Savoir configurer PostgreSQL pour des workloads lourds.
- Comprendre l’impact des paramètres système sur la performance.

### 📚 Contenu
- Paramètres clés : `work_mem`, `shared_buffers`, `effective_cache_size`.
- WAL : `max_wal_size`, `checkpoint_timeout`, compression.
- Vacuum & autovacuum approfondi.
- Parallel queries : fonctionnement et tuning.
- Gestion du cache et des buffers.
- Optimisation pratique pour gros datasets.

---

## Module 3 — Modélisation & Data Engineering

### 🎯 Objectifs
- Structurer les données pour l’analytics, la scalabilité et les pipelines de données.

### 📚 Contenu
- Modélisation BI : **Star Schema / Snowflake**.
- Normalisation / dénormalisation selon les besoins analytiques.
- Techniques pour **big tables** (100M – 1B rows).
- Retention & archival strategies.
- Pipelines de données :
  - Ingestion
  - ELT / ETL en SQL
  - Change Data Capture (CDC)
  - Logical replication pour scalabilité et HA.

---

## Module 4 — Monitoring, Outils & Maintenance

### 🎯 Objectifs
- Superviser, maintenir et diagnostiquer un cluster PostgreSQL en production.

### 📚 Contenu
- Outils de monitoring :
  - `pg_stat_statements`
  - `pg_stat_activity`
  - pgBadger
  - Grafana + Prometheus
  - Datadog / New Relic
- Sauvegarde & restauration :
  - `pg_dump` / `pg_restore`
  - `pg_basebackup`
  - Snapshots filesystem
- Maintenance proactive :
  - Surveillance des index, bloat et fragmentation
  - Ajustements autovacuum et paramètres

---

## Module 5 — Atelier pratique & Certification interne

### 🎯 Objectifs
- Mettre en pratique l’ensemble des connaissances acquises dans la formation.

### 🛠 Atelier : construire un Mini Data Warehouse
- **Modélisation** d’un Star Schema.
- **Import** d’un dataset massif.
- **Partitionnement** et indexation intelligente.
- **Optimisation** de 10 requêtes avec benchmarks.
- **Analyse complète** via explain.dalibo.com.
- **Mise en place d’un pipeline ELT** simple.
- **Monitoring** et statistiques évolutives.
- **Documentation** et restitution orale.

### 🏁 Résultat final
À la fin de cette formation, l’apprenant maîtrise :
- Les plans d’exécution et l’optimisation SQL (niveau expert)
- La configuration et le tuning PostgreSQL
- La conception BI / DWH
- La création de pipelines de données
- Les outils de monitoring et de maintenance pour la production
