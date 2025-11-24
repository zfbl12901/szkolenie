# 2. Analyse des plans d'exécution

## 🎯 Objectifs

- Maîtriser EXPLAIN et EXPLAIN ANALYZE
- Interpréter les différents types d'opérations
- Comprendre les coûts et temps d'exécution
- Identifier les problèmes de performance dans les plans

## 📋 Table des matières

1. [EXPLAIN et EXPLAIN ANALYZE](#explain-et-explain-analyze)
2. [Types d'opérations](#types-dopérations)
3. [Interprétation des coûts](#interprétation-des-coûts)
4. [Signaux d'alerte](#signaux-dalerte)
5. [Bonnes pratiques](#bonnes-pratiques)

---

## EXPLAIN et EXPLAIN ANALYZE

### EXPLAIN (sans exécution)

Affiche le plan d'exécution estimé **sans exécuter la requête** :

```sql
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';
```

**Résultat :**
```
Seq Scan on users  (cost=0.00..25.00 rows=1 width=64)
  Filter: (email = 'user@example.com'::text)
```

### EXPLAIN ANALYZE (avec exécution)

Exécute la requête et affiche les **temps réels** :

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'user@example.com';
```

**Résultat :**
```
Seq Scan on users  (cost=0.00..25.00 rows=1 width=64) 
  (actual time=0.123..15.456 rows=1 loops=1)
  Filter: (email = 'user@example.com'::text)
  Rows Removed by Filter: 9999
Planning Time: 0.234 ms
Execution Time: 15.678 ms
```

### Options utiles

```sql
-- Format JSON (pour outils externes)
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) 
SELECT * FROM users WHERE email = 'user@example.com';

-- Afficher les buffers
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM users WHERE email = 'user@example.com';

-- Afficher les paramètres de planification
EXPLAIN (ANALYZE, VERBOSE, SETTINGS) 
SELECT * FROM users WHERE email = 'user@example.com';

-- Format YAML
EXPLAIN (ANALYZE, FORMAT YAML) 
SELECT * FROM users WHERE email = 'user@example.com';
```

### Interprétation des métriques

**Coûts estimés :**
- `cost=0.00..25.00` : Coût de démarrage..Coût total
- `rows=1` : Nombre de lignes estimées
- `width=64` : Taille moyenne d'une ligne en octets

**Temps réels (ANALYZE) :**
- `actual time=0.123..15.456` : Temps de démarrage..Temps total (ms)
- `rows=1` : Nombre réel de lignes retournées
- `loops=1` : Nombre d'exécutions de cette opération
- `Planning Time` : Temps de planification
- `Execution Time` : Temps total d'exécution

**Buffers (avec BUFFERS) :**
- `shared hit=15` : Pages lues depuis le cache partagé
- `shared read=3` : Pages lues depuis le disque
- `shared written=0` : Pages écrites
- `temp read/written` : Pages temporaires

---

## Types d'opérations

### Seq Scan (Scan séquentiel)

**Quand utilisé :**
- Pas d'index approprié
- Table petite (< 10% de la table)
- Index non sélectif

**Exemple :**
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE status = 'inactive';
```

**Interprétation :**
- ✅ Acceptable pour petites tables
- ⚠️ Problématique pour grandes tables
- 🔍 **Action** : Créer un index si la table est grande

### Index Scan

**Quand utilisé :**
- Index disponible et sélectif
- Accès direct par index

**Exemple :**
```sql
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'user@example.com';
```

**Résultat typique :**
```
Index Scan using idx_users_email on users  
  (cost=0.42..8.44 rows=1 width=64)
  (actual time=0.123..0.125 rows=1 loops=1)
  Index Cond: (email = 'user@example.com'::text)
```

**Interprétation :**
- ✅ Bonne performance
- ✅ Accès direct aux lignes

### Index Only Scan

**Quand utilisé :**
- Toutes les colonnes nécessaires sont dans l'index
- Pas besoin d'accéder à la table

**Exemple :**
```sql
-- Index sur (id, email)
CREATE INDEX idx_users_id_email ON users(id, email);

EXPLAIN ANALYZE 
SELECT id, email FROM users WHERE id BETWEEN 1 AND 100;
```

**Résultat typique :**
```
Index Only Scan using idx_users_id_email on users
  (cost=0.42..5.44 rows=100 width=64)
  (actual time=0.123..0.456 rows=100 loops=1)
  Index Cond: ((id >= 1) AND (id <= 100))
  Heap Fetches: 0
```

**Interprétation :**
- ✅ Performance optimale
- ✅ `Heap Fetches: 0` = pas d'accès à la table

### Bitmap Index Scan + Bitmap Heap Scan

**Quand utilisé :**
- Conditions multiples avec plusieurs index
- Retourne plusieurs lignes

**Exemple :**
```sql
EXPLAIN ANALYZE 
SELECT * FROM users 
WHERE status = 'active' AND created_at > '2024-01-01';
```

**Résultat typique :**
```
Bitmap Heap Scan on users
  (cost=4.44..25.67 rows=50 width=64)
  (actual time=0.234..1.456 rows=45 loops=1)
  Recheck Cond: ((status = 'active'::text) AND (created_at > '2024-01-01'::date))
  Heap Blocks: exact=12
  -> Bitmap Index Scan on idx_users_status
      (cost=0.00..4.43 rows=50 width=0)
      (actual time=0.123..0.123 rows=45 loops=1)
      Index Cond: (status = 'active'::text)
```

**Interprétation :**
- ✅ Efficace pour plusieurs conditions
- ⚠️ `Recheck Cond` = vérification supplémentaire

### Nested Loop

**Quand utilisé :**
- Petites tables ou résultats limités
- Une table externe petite

**Exemple :**
```sql
EXPLAIN ANALYZE 
SELECT u.*, o.* 
FROM users u 
JOIN orders o ON u.id = o.user_id 
WHERE u.id = 123;
```

**Résultat typique :**
```
Nested Loop
  (cost=0.85..25.67 rows=10 width=128)
  (actual time=0.123..2.456 rows=8 loops=1)
  -> Index Scan using idx_users_id on users
      (cost=0.42..8.44 rows=1 width=64)
      (actual time=0.089..0.090 rows=1 loops=1)
      Index Cond: (id = 123)
  -> Index Scan using idx_orders_user_id on orders
      (cost=0.42..17.23 rows=10 width=64)
      (actual time=0.234..2.345 rows=8 loops=1)
      Index Cond: (user_id = 123)
```

**Interprétation :**
- ✅ Efficace pour petites boucles
- ⚠️ Peut être lent si la boucle externe est grande

### Hash Join

**Quand utilisé :**
- Tables de taille similaire
- Pas d'index sur la clé de jointure
- Égalité simple

**Exemple :**
```sql
EXPLAIN ANALYZE 
SELECT u.*, o.* 
FROM users u 
JOIN orders o ON u.id = o.user_id;
```

**Résultat typique :**
```
Hash Join
  (cost=125.67..456.78 rows=10000 width=128)
  (actual time=2.345..15.678 rows=9876 loops=1)
  Hash Cond: (o.user_id = u.id)
  -> Seq Scan on orders o
      (cost=0.00..234.56 rows=10000 width=64)
      (actual time=0.123..5.678 rows=10000 loops=1)
  -> Hash
      (cost=123.45..123.45 rows=1000 width=64)
      (actual time=1.234..1.234 rows=1000 loops=1)
      Buckets: 1024  Batches: 1  Memory Usage: 64kB
      -> Seq Scan on users u
          (cost=0.00..123.45 rows=1000 width=64)
          (actual time=0.089..0.567 rows=1000 loops=1)
```

**Interprétation :**
- ✅ Efficace pour jointures d'égalité
- ⚠️ Nécessite de la mémoire (`work_mem`)
- 🔍 **Action** : Augmenter `work_mem` si "Batches > 1"

### Merge Join

**Quand utilisé :**
- Données déjà triées
- Jointures sur clés triées
- Opérateurs de comparaison (<, >, <=, >=)

**Exemple :**
```sql
EXPLAIN ANALYZE 
SELECT u.*, o.* 
FROM users u 
JOIN orders o ON u.id = o.user_id 
ORDER BY u.id;
```

**Interprétation :**
- ✅ Efficace si les données sont triées
- ⚠️ Nécessite un tri si les données ne le sont pas

### Sort

**Quand utilisé :**
- ORDER BY
- GROUP BY (parfois)
- Opérations nécessitant un tri

**Exemple :**
```sql
EXPLAIN ANALYZE 
SELECT * FROM users ORDER BY created_at DESC LIMIT 100;
```

**Résultat typique :**
```
Limit
  (cost=234.56..256.78 rows=100 width=64)
  (actual time=12.345..15.678 rows=100 loops=1)
  -> Sort
      (cost=234.56..256.78 rows=10000 width=64)
      (actual time=12.345..15.234 rows=100 loops=1)
      Sort Key: created_at DESC
      Sort Method: top-N heapsort  Memory: 32kB
      -> Seq Scan on users
          (cost=0.00..123.45 rows=10000 width=64)
          (actual time=0.089..5.678 rows=10000 loops=1)
```

**Interprétation :**
- ⚠️ `Sort Method: external merge` = tri sur disque (lent)
- ✅ `Sort Method: quicksort` = tri en mémoire (rapide)
- 🔍 **Action** : Augmenter `work_mem` si tri sur disque

### Aggregate

**Quand utilisé :**
- Fonctions d'agrégation (COUNT, SUM, AVG, etc.)
- GROUP BY

**Exemple :**
```sql
EXPLAIN ANALYZE 
SELECT status, COUNT(*) 
FROM users 
GROUP BY status;
```

**Résultat typique :**
```
HashAggregate
  (cost=123.45..145.67 rows=5 width=12)
  (actual time=2.345..2.456 rows=5 loops=1)
  Group Key: status
  Batches: 1  Memory Usage: 24kB
  -> Seq Scan on users
      (cost=0.00..98.76 rows=10000 width=4)
      (actual time=0.089..1.234 rows=10000 loops=1)
```

**Interprétation :**
- ✅ `HashAggregate` = efficace
- ⚠️ `GroupAggregate` = peut être lent
- 🔍 **Action** : Augmenter `work_mem` si "Batches > 1"

---

## Interprétation des coûts

### Structure des coûts

```
cost=0.00..25.00
  ↑      ↑
  |      └─ Coût total
  └─ Coût de démarrage
```

**Coût de démarrage :** Coût avant de retourner la première ligne
**Coût total :** Coût pour retourner toutes les lignes

### Comparaison des coûts

**Règle générale :**
- Coût < 100 : Très rapide
- Coût 100-1000 : Rapide
- Coût 1000-10000 : Modéré
- Coût > 10000 : Potentiellement lent

**⚠️ Important :** Les coûts sont relatifs et dépendent de la configuration.

### Écart entre estimation et réalité

**Comparer :**
- `rows` (estimé) vs `rows` (réel dans ANALYZE)
- `cost` (estimé) vs `actual time` (réel)

**Exemple problématique :**
```
Seq Scan on users
  (cost=0.00..25.00 rows=1 width=64)
  (actual time=0.123..1500.456 rows=100000 loops=1)
```

**Problème :** Estimation très incorrecte (1 ligne estimée, 100000 réelles)
**Action :** Exécuter `ANALYZE users;`

---

## Signaux d'alerte

### 🔴 Alertes critiques

1. **Seq Scan sur grande table**
   ```
   Seq Scan on large_table (cost=0.00..50000.00 rows=1000000)
   ```
   **Action :** Créer un index approprié

2. **Tri sur disque**
   ```
   Sort Method: external merge  Disk: 50000kB
   ```
   **Action :** Augmenter `work_mem`

3. **Estimation très incorrecte**
   ```
   rows=1 (estimated) vs rows=100000 (actual)
   ```
   **Action :** Exécuter `ANALYZE`

4. **Hash Join avec plusieurs batches**
   ```
   Hash Join
     Batches: 16  Memory Usage: 512kB
   ```
   **Action :** Augmenter `work_mem`

5. **Nested Loop avec grande boucle externe**
   ```
   Nested Loop (loops=100000)
   ```
   **Action :** Vérifier les index ou changer le type de jointure

### 🟡 Alertes modérées

1. **Index Scan avec beaucoup de Heap Fetches**
   ```
   Index Only Scan
     Heap Fetches: 50000
   ```
   **Action :** Vérifier la visibilité des tuples

2. **Bitmap Heap Scan avec beaucoup de rechecks**
   ```
   Rows Removed by Filter: 50000
   ```
   **Action :** Améliorer la sélectivité de l'index

3. **Temps de planification élevé**
   ```
   Planning Time: 500.234 ms
   ```
   **Action :** Simplifier la requête ou augmenter `plan_cache_mode`

---

## Bonnes pratiques

### 1. Toujours utiliser EXPLAIN ANALYZE pour les requêtes lentes

```sql
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT ...;
```

### 2. Comparer les estimations et la réalité

Vérifier si `rows` (estimé) ≈ `rows` (réel)

### 3. Surveiller les buffers

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

- `shared hit` élevé = bon (cache)
- `shared read` élevé = peut être amélioré (I/O disque)

### 4. Identifier les opérations coûteuses

Chercher les opérations avec :
- `actual time` élevé
- `loops` élevé
- `rows` beaucoup plus élevé que l'estimation

### 5. Utiliser des outils de visualisation

- **pgAdmin** : Visualisation graphique des plans
- **explain.dalibo.com** : Analyse en ligne
- **pev** : PostgreSQL Explain Visualizer

---

## 📊 Points clés à retenir

1. **EXPLAIN** = estimation, **EXPLAIN ANALYZE** = réalité
2. **Seq Scan** sur grande table = problème potentiel
3. **Tri sur disque** = augmenter `work_mem`
4. **Estimation incorrecte** = exécuter `ANALYZE`
5. **Comparer toujours** estimation vs réalité

## 🔗 Prochain module

Passer au module [3. Dalibo - Outil d'analyse](../03-dalibo/README.md) pour apprendre à utiliser Dalibo pour l'analyse de performance.

