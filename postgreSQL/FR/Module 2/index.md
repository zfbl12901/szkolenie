# Module 2 — Optimisation des requêtes

## 🎯 Objectifs
- Comprendre comment PostgreSQL exécute réellement une requête.
- Diagnostiquer et optimiser les performances SQL.

---

## 📚 Contenu

### 1. Lecture du Query Planner
- Comprendre le rôle du planner et du coût estimé.
- Commandes essentielles :
```sql
EXPLAIN SELECT ...;
EXPLAIN ANALYZE SELECT ...;
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) SELECT ...;
```
- Analyse des principaux indicateurs :
  - `cost` : coût estimé
  - `rows` : lignes estimées
  - `actual rows` : lignes réellement parcourues
  - `loops` : nombre de répétitions d'un nœud
  - `timings` : temps d'exécution
  - `buffers` : mémoire et I/O

---

### 2. Types de scans
- **Seq Scan** : lecture complète de la table, efficace pour les petites tables ou peu de filtrage.
- **Index Scan** : recherche via index pour filtrer rapidement.
- **Bitmap Index Scan / Bitmap Heap Scan** : combiner plusieurs conditions avec un scan optimisé.

---

### 3. Types de jointures
- **Nested Loop** : efficace pour petits datasets ou conditions sélectives.
- **Merge Join** : efficace si les tables sont triées ou indexées.
- **Hash Join** : efficace pour jointures de gros datasets sans ordre spécifique.

---

### 4. Overhead mémoire & disk I/O
- Comprendre l'impact du cache, des shared buffers et du WAL.
- Identifier les points où la lecture disque devient un goulot d'étranglement.

---

### 5. Indexation avancée

#### Types d'index spécialisés :
- **GIN** : JSONB, tableaux, recherche textuelle
- **GiST** : PostGIS, trigrammes, recherches approximatives
- **BRIN** : tables massives triées naturellement

#### Index partiels : indexer uniquement certaines lignes
```sql
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```

#### Index sur expressions : pour optimiser les fonctions ou transformations
```sql
CREATE INDEX idx_lower_email ON users(LOWER(email));
```

#### Index multicolonnes : importance de l'ordre et de la sélectivité des colonnes
```sql
CREATE INDEX idx_orders_customer_date ON orders(customer_id, created_at);
```

---

### 6. Optimisation pratique

#### Réécriture de requêtes :
- Préférer JOIN aux sous-requêtes corrélées
- Éviter les `SELECT *`
- CTE matérialisés vs non matérialisés (impact sur performances)
- Usage du partitionnement pour les grosses tables

#### Antipatterns classiques :
- Seq Scan non désiré
- OR multiples
- Fonctions dans WHERE sur colonnes non indexées
- Trop de CTE inutiles

---

### 7. Analyse visuelle des plans via explain.dalibo.com

#### Génération du plan JSON
```sql
EXPLAIN (ANALYZE, COSTS, VERBOSE, BUFFERS, FORMAT JSON) SELECT ...;
```

#### Lecture de l'arbre pour identifier :
- Nœuds coûteux
- Sélectivité mal estimée
- Types de scans utilisés
- Comparaison avant / après optimisation
- Usage hors-ligne pour sécuriser les données sensibles

---

## ✅ Compétences attendues à l'issue du module
- Lire et interpréter un plan d'exécution PostgreSQL.
- Identifier les goulots de performance et les points d'optimisation.
- Maîtriser l'usage d'index avancés pour accélérer les requêtes.
- Réécrire et structurer les requêtes pour de meilleures performances.
- Visualiser les plans via explain.dalibo.com pour l'analyse et le partage.
