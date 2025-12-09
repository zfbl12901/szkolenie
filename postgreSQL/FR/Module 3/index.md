# Module 3 — Performance & Scalabilité

## 🎯 Objectifs
- Savoir configurer PostgreSQL pour des workloads lourds.
- Comprendre l'impact des paramètres système sur les performances.

---

## 📚 Contenu

### 1. Tuning des paramètres clés

#### 1.1 `work_mem`
- Mémoire allouée pour les opérations de tri et hash dans une requête.
- Impact direct sur les performances des JOIN, ORDER BY, GROUP BY.
- Paramètre par session ou par requête :
```sql
SET work_mem = '64MB';
```

#### 1.2 `shared_buffers`
- Mémoire cache utilisée pour stocker des blocs de tables et d'index.
- Recommandation : ~25–40% de la RAM disponible.
- Contrôle le nombre de lectures disque évitées.

#### 1.3 `effective_cache_size`
- Estimation par PostgreSQL de la taille du cache disque disponible.
- Influence les choix du planner (seq scan vs index scan).
- Exemple :
```sql
SET effective_cache_size = '4GB';
```

---

### 2. WAL & checkpoints

#### 2.1 `max_wal_size` et `checkpoint_timeout`
- Définissent la taille maximale du WAL avant un checkpoint et la fréquence des checkpoints.
- Objectif : limiter le temps d'attente et le I/O de checkpoint.
- Exemple :
```conf
max_wal_size = 1GB
checkpoint_timeout = 10min
```

#### 2.2 Compression WAL
- Activer la compression WAL pour réduire l'occupation disque et le coût de réplication.
- Paramètre : `wal_compression = on`

---

### 3. VACUUM & Autovacuum

#### 3.1 VACUUM manuel
- Nettoyage des tuples morts et récupération d'espace.
- Analyse des statistiques simultanée :
```sql
VACUUM ANALYZE table_name;
```

#### 3.2 Autovacuum
- Automatique, mais configurable pour ajuster les tables volumineuses.
- Paramètres importants :
  - `autovacuum_max_workers`
  - `autovacuum_naptime`
  - `autovacuum_vacuum_threshold`
  - `autovacuum_analyze_threshold`
- Objectif : éviter le bloat et maintenir la précision des stats.

---

### 4. Parallel queries
- PostgreSQL peut exécuter certaines requêtes en parallèle pour exploiter plusieurs CPU.
- Paramètres :
  - `max_parallel_workers_per_gather` : nombre de workers parallèles par requête
  - `parallel_setup_cost` / `parallel_tuple_cost` : coût estimé pour activer le parallélisme
- Cas d'usage : grosses tables OLAP, agrégations lourdes.

---

### 5. Gestion du cache & buffers
- Comprendre le fonctionnement du cache mémoire (shared buffers, OS cache).
- Mesurer l'efficacité via les vues système :
  - `pg_stat_database`
  - `pg_statio_user_tables`
- Techniques pour optimiser les lectures et réduire les I/O :
  - Ajuster `work_mem` et `shared_buffers`
  - Minimiser les séquences Seq Scan inutiles
  - Utiliser les index adaptés

---

## ✅ Compétences attendues à l'issue du module
- Ajuster les paramètres clés pour optimiser les performances PostgreSQL.
- Comprendre l'impact de WAL, checkpoints et autovacuum sur le I/O.
- Configurer les queries parallèles pour exploiter le CPU efficacement.
- Suivre et analyser les performances du cache et des buffers.
- Préparer PostgreSQL à des workloads lourds et scalables.
