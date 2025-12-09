# Module 5 — Architecture interne & stockage

## 🎯 Objectifs
- Comprendre la mécanique interne de PostgreSQL pour un tuning professionnel et la gestion des données à grande échelle.

---

## 📚 Contenu

### 1. Organisation en pages / blocs / TOAST
- PostgreSQL stocke les données en **blocs/pages** (par défaut 8KB).
- Chaque table et index est découpée en pages.
- **TOAST** (The Oversized-Attribute Storage Technique) :
  - Gère les colonnes très volumineuses (`text`, `bytea`, `jsonb`).
  - Stockage automatique hors-page pour éviter les problèmes de taille de bloc.
- Concepts clés :
  - Heap : stockage principal des tuples.
  - Visibility Map : indique si un bloc contient des tuples visibles pour toutes les transactions.

---

### 2. Rôle du WAL (Write-Ahead Log)
- Assure la **durabilité (D dans ACID)**.
- Tout changement est d'abord écrit dans le WAL avant d'être appliqué sur la table.
- Permet :
  - La récupération après crash
  - La réplication logique et physique
  - Les sauvegardes PITR (Point In Time Recovery)

---

### 3. Checkpoints
- Moment où les pages modifiées sont écrites sur le disque.
- Paramètres clés :
  - `checkpoint_timeout`
  - `max_wal_size`
- Optimisation :
  - Checkpoints trop fréquents → I/O élevé
  - Checkpoints trop espacés → récupération plus longue après crash

---

### 4. MVCC (Multi-Version Concurrency Control) en profondeur
- Chaque transaction voit une **photo cohérente** de la base.
- Les anciennes versions de tuples restent visibles pour d'autres transactions.
- Garbage collection gérée par **VACUUM**.
- Avantages :
  - Lecture non bloquante
  - Concurrence élevée
- Points d'attention :
  - Tuples morts → bloat → impact performance
  - Importance du tuning de `autovacuum`

---

### 5. Deadlocks & contention
- Deadlocks : blocage circulaire entre transactions.
- Détection et résolution automatique par PostgreSQL.
- Analyse via `pg_locks` et `pg_stat_activity` :
```sql
SELECT pid, locktype, relation::regclass AS table, mode, granted
FROM pg_locks
WHERE NOT granted;
```
- Stratégies de mitigation :
  - Transactions courtes
  - Mise à jour des tables dans un ordre constant
  - `SKIP LOCKED` pour files de traitement concurrentes

---

### 6. Logical decoding & bases du CDC (Change Data Capture)
- Permet de capturer les modifications des données (INSERT, UPDATE, DELETE) pour :
  - Réplication logique
  - Streaming vers des systèmes externes (Kafka, ETL)
- Concepts clés :
  - Slot de réplication logique
  - Publication / Subscription
- Commandes de base :
```sql
CREATE PUBLICATION my_pub FOR TABLE users;
CREATE SUBSCRIPTION my_sub CONNECTION 'host=... dbname=... user=... password=...' PUBLICATION my_pub;
```

---

## ✅ Compétences attendues à l'issue du module
- Comprendre la structure physique des tables et la gestion des grandes colonnes via TOAST.
- Connaître le rôle du WAL et des checkpoints pour la durabilité et la réplication.
- Maîtriser MVCC et son impact sur la concurrence.
- Identifier et gérer les deadlocks et la contention.
- Comprendre les bases du CDC avec la réplication logique.
