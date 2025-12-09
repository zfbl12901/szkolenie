# Module 1 — Optimisation des requêtes

## 🎯 Objectifs
- Comprendre comment PostgreSQL exécute réellement une requête.
- Diagnostiquer et optimiser les performances SQL sur de gros volumes.
- Apprendre à utiliser les plans d'exécution pour identifier les goulots d'étranglement.

---

## 📋 Prérequis

Avant de commencer ce module, vous devez maîtriser :

- **SQL de base** : SELECT, WHERE, JOIN, GROUP BY, ORDER BY
- **PostgreSQL fondamental** : connexion à une base, création de tables, insertion de données
- **Notions de base de données** : tables, colonnes, index, clés primaires/étrangères
- **Environnement** : accès à une instance PostgreSQL (locale ou distante) et à l'outil `psql`

**Niveau recommandé :** Intermédiaire (au moins 6 mois d'expérience SQL)

Si vous débutez avec PostgreSQL, consultez d'abord les bases : création de tables, requêtes simples, gestion des index basiques.

---

## 📚 Contenu

### 1. EXPLAIN / EXPLAIN ANALYZE

PostgreSQL fournit deux commandes essentielles pour comprendre et optimiser les performances des requêtes : **EXPLAIN** et **EXPLAIN ANALYZE**.

---

#### 1.1 EXPLAIN

- **Objectif** : voir le **plan d'exécution estimé** par le planner PostgreSQL **sans exécuter la requête**.  
- Utile pour analyser le coût estimé d'une requête et anticiper les goulots d'étranglement avant qu'elle ne touche de gros volumes de données.  

**Exemple :**

```sql
EXPLAIN SELECT * FROM orders WHERE order_date >= '2025-01-01';
```

**Résultat typique :**

```
Seq Scan on orders  (cost=0.00..431.00 rows=20 width=120)
```

- **Seq Scan** : le planner estime qu'un scan séquentiel est nécessaire.
- **Cost** : estimation du coût d'exécution, du début (0.00) à la fin (431.00).
- **Rows** : estimation du nombre de lignes retournées (ici 20).

À noter : ces chiffres sont des estimations basées sur les statistiques et peuvent diverger des résultats réels si les statistiques ne sont pas à jour.

---

#### 1.2 EXPLAIN ANALYZE

- **Objectif** : exécuter réellement la requête et fournir les temps réels, le nombre de lignes traitées, et les boucles pour chaque nœud du plan.
- Permet de comparer les estimations vs réalité, ce qui est crucial pour optimiser efficacement.

**Exemple :**

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE order_date >= '2025-01-01';
```

**Résultat typique :**

```
Seq Scan on orders  (cost=0.00..431.00 rows=20 width=120) 
(actual time=0.012..0.034 rows=18 loops=1)
```

- **Actual time** : temps réel d'exécution (début..fin) pour ce nœud.
- **Actual rows** : nombre réel de lignes traitées (ici 18 au lieu des 20 estimées).
- **Loops** : combien de fois l'opération a été répétée (1 ici, mais peut être plus pour des sous-queries ou joins).

---

#### 1.3 Types de scans les plus courants

| Scan | Description | Quand il apparaît |
|------|-------------|-------------------|
| **Seq Scan** | Lecture séquentielle de toute la table | Table petite ou pas d'index pertinent |
| **Index Scan** | Lecture via un index pour filtrer les lignes | Filtrage sur colonnes indexées |
| **Bitmap Index Scan** | Combinaison d'index et lecture de pages mémoire | Quand plusieurs index ou plages de valeurs sont utilisées |
| **Index Only Scan** | Lecture uniquement via l'index, sans accéder à la table | Si toutes les colonnes nécessaires sont dans l'index |

---

#### 1.4 Comprendre loops et rows

- **Loops** : nombre d'itérations de l'opération.
  - Par exemple, pour un nested loop join, le backend peut exécuter plusieurs fois la même opération pour chaque ligne de la table extérieure.
- **Rows** : nombre de tuples traités à chaque étape.
  - Identifier les nœuds où `rows` ou `loops` sont très élevés permet de localiser les goulots d'étranglement.

**Exemple de nested loop :**

```
Nested Loop  (cost=0.85..431.00 rows=20) (actual time=0.012..0.034 rows=18 loops=10)
```

Ici, la boucle a été exécutée 10 fois, ce qui peut indiquer un join inefficace ou un index manquant.

---

#### 1.5 Bonnes pratiques

- Toujours comparer `EXPLAIN` vs `EXPLAIN ANALYZE` pour vérifier si le planner se trompe dans ses estimations.
- Mettre à jour les statistiques régulièrement avec `ANALYZE`.
- Chercher à transformer un Seq Scan coûteux en Index Scan ou Bitmap Scan si possible (voir [section 3. Types de scans](#3-types-de-scans)).
- Identifier les joins coûteux et vérifier si un index ou un CTE matérialisé peut réduire le nombre de loops (voir [section 2. Types de joins](#2-types-de-joins) et [section 6. CTE](#6-cte-common-table-expressions)).
- Pour les gros volumes, générer le plan JSON et l'analyser sur explain.dalibo.com pour visualiser l'arbre et détecter les nœuds coûteux.

---

#### 1.6 Analyse visuelle avec explain.dalibo.com

Pour exploiter au mieux les plans d'exécution, PostgreSQL permet de générer le plan au format **JSON**, qui peut ensuite être visualisé graphiquement via [explain.dalibo.com](https://explain.dalibo.com/).

##### 1.6.1 Générer le plan JSON

```sql
EXPLAIN (ANALYZE, COSTS, BUFFERS, FORMAT JSON)
SELECT *
FROM orders
WHERE order_date >= '2025-01-01';
```

- **ANALYZE** : exécute la requête pour obtenir les temps réels.
- **COSTS** : inclut les coûts estimés dans le plan.
- **BUFFERS** : inclut les informations sur l'utilisation du cache.
- **FORMAT JSON** : permet d'exporter le plan sous forme de JSON compatible avec explain.dalibo.com.

##### 1.6.2 Exploiter le plan sur Dalibo

1. Copier le JSON généré par PostgreSQL.
2. Coller le JSON sur explain.dalibo.com.
3. Visualiser le plan sous forme d'arbre graphique.
4. Identifier les nœuds coûteux grâce aux informations suivantes :
   - **Total cost vs Actual time** : différence entre estimation et réalité.
   - **Rows vs Actual rows** : lignes estimées vs lignes traitées.
   - **Loops** : répétitions des opérations, indicateur de joins coûteux.
5. Comparer avant/après optimisation pour mesurer l'impact des changements (indexation, réécriture de requêtes, partitionnement, CTE).

##### 1.6.3 Bonnes pratiques avec Dalibo

- Toujours générer le plan JSON pour les requêtes critiques sur de gros volumes.
- Vérifier que les opérations coûteuses identifiées correspondent aux goulots d'étranglement observés.
- Utiliser l'arborescence graphique pour expliquer les optimisations à l'équipe ou documenter les améliorations.
- Coupler l'analyse Dalibo avec les statistiques PostgreSQL (`ANALYZE`, `pg_stat_statements`) pour une compréhension complète.

---

### ✅ À retenir

- `EXPLAIN` = plan estimé, `EXPLAIN ANALYZE` = plan réel.
- `Loops` et `rows` montrent la répétition et le volume réel de données.
- Les types de scans et joins déterminent l'efficacité d'une requête (voir [section 3. Types de scans](#3-types-de-scans) et [section 2. Types de joins](#2-types-de-joins)).
- Une lecture attentive du plan permet d'identifier rapidement les optimisations possibles.

---

### 2. Types de joins

Lorsqu'on joint plusieurs tables, PostgreSQL choisit un **type de join** selon la taille des tables, la présence d'index et les statistiques. Comprendre ces types de joins est essentiel pour optimiser les requêtes sur de gros volumes.

---

#### 2.1 Nested Loop Join

- **Principe** : PostgreSQL parcourt **chaque ligne** de la table extérieure (A) et recherche les correspondances dans la table intérieure (B).  
- **Fonctionnement** :
  1. Pour chaque ligne de A, effectuer une recherche sur B.  
  2. Combiner les lignes correspondantes.  
- **Quand c'est utilisé** :  
  - Tables petites ou moyennes.  
  - Quand la table intérieure dispose d'un **index efficace** sur la clé jointe.  
- **Avantages** :
  - Simple à comprendre et fiable.  
  - Efficace si la table extérieure est petite.  
- **Inconvénients** :
  - Devient très coûteux si les deux tables sont volumineuses et sans index.

**Exemple :**

```sql
SELECT *
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.region = 'Europe';
```

Si `customers` est petite et `orders.customer_id` est indexé → Nested Loop est choisi.

---

#### 2.2 Hash Join

- **Principe** : PostgreSQL crée une table de hachage en mémoire pour la table la plus petite, puis parcourt la table la plus grande pour trouver les correspondances.
- **Fonctionnement** :
  1. Construire une hash table pour la table la plus petite sur la colonne jointe.
  2. Scanner la table la plus grande et chercher chaque valeur dans la hash table.
- **Quand c'est utilisé** :
  - Pour les tables volumineuses.
  - Sur des conditions d'égalité (`=`) sur les colonnes join.
- **Avantages** :
  - Très rapide pour de gros volumes si la table hash fit en mémoire.
- **Inconvénients** :
  - Utilise beaucoup de mémoire.
  - Pas efficace pour les joins non-égalité (`<`, `>`) ou pour des tables déjà triées.

**Exemple :**

```sql
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

Si `orders` et `customers` sont grandes → PostgreSQL peut choisir un Hash Join.

---

#### 2.3 Merge Join

- **Principe** : Les deux tables sont triées sur la clé jointe, puis fusionnées en parcourant les deux tables en parallèle.
- **Fonctionnement** :
  1. Trier A et B sur la clé jointe (si non déjà triées ou indexées).
  2. Parcourir simultanément les deux tables pour trouver les correspondances.
- **Quand c'est utilisé** :
  - Tables déjà triées ou indexées sur la clé jointe.
  - Souvent utilisé pour de gros volumes avec des index B-Tree.
- **Avantages** :
  - Très efficace pour les grandes tables triées.
  - Moins gourmand en mémoire que le Hash Join.
- **Inconvénients** :
  - Nécessite un tri si les tables ne sont pas déjà triées.
  - Moins efficace si seules quelques lignes sont nécessaires (petit filtre).

**Exemple :**

```sql
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
ORDER BY c.customer_id;
```

PostgreSQL utilisera souvent Merge Join si les deux tables sont indexées sur `customer_id`.

---

### ✅ Points clés à retenir

- **Nested Loop** → simple et efficace pour petites tables ou index existants.
- **Hash Join** → idéal pour gros volumes et égalités sur colonnes join.
- **Merge Join** → efficace pour tables déjà triées ou indexées, bon compromis mémoire/performance.
- Comprendre le type de join permet de diagnostiquer les requêtes lentes et de décider des optimisations (index, partitionnement, réécriture de requête).

**💡 Cas d'usage Data Engineer :** Dans un pipeline ETL, les Hash Joins sont souvent préférés pour joindre de grandes tables de faits avec des dimensions. Si vous voyez beaucoup de Nested Loops sur de gros volumes, c'est un signal d'alerte : vérifiez les index sur les clés de jointure (voir [section 4. Indexation avancée](#4-indexation-avancée)).

---

### 3. Types de scans

Les scans sont la façon dont PostgreSQL lit les données dans les tables pour exécuter une requête. Le choix du scan a un impact direct sur les performances, surtout sur de grandes tables.

---

#### 3.1 Seq Scan (Sequential Scan)

- **Principe** : PostgreSQL lit **toutes les lignes** de la table, une par une.  
- **Quand utilisé** :  
  - Table petite ou très peu filtrée.  
  - Pas d'index disponible sur la colonne filtrée.  
- **Avantages** : simple, fiable.  
- **Inconvénients** : coûteux pour les grandes tables, lecture complète même si peu de lignes sont nécessaires.

**Exemple :**

```sql
SELECT * FROM orders WHERE amount > 1000;
```

Si `amount` n'est pas indexé, PostgreSQL effectue un Seq Scan sur toute la table.

---

#### 3.2 Index Scan

- **Principe** : PostgreSQL utilise un index pour trouver les lignes correspondantes.
- **Quand utilisé** :
  - Filtre sur une colonne indexée.
  - Table grande, mais faible proportion de lignes filtrées.
- **Avantages** : lecture ciblée → moins de données à lire.
- **Inconvénients** : moins efficace si la majorité de la table est nécessaire, car les lectures deviennent nombreuses et aléatoires.

**Exemple :**

```sql
CREATE INDEX idx_orders_amount ON orders(amount);
SELECT * FROM orders WHERE amount > 1000;
```

L'index permet de localiser rapidement les lignes concernées.

---

#### 3.3 Bitmap Index Scan

- **Principe** : PostgreSQL combine plusieurs index (ou plages de valeurs) pour créer un bitmap, puis lit uniquement les blocs nécessaires en mémoire.
- **Quand utilisé** :
  - Plusieurs filtres sur différentes colonnes indexées.
  - Grandes tables avec un grand nombre de lignes à récupérer.
- **Avantages** : réduit les lectures disque aléatoires, combine efficacement plusieurs index.
- **Inconvénients** : utilise plus de mémoire pour construire le bitmap.

**Exemple :**

```sql
SELECT * FROM orders
WHERE amount > 1000 AND status = 'active';
```

PostgreSQL peut utiliser un bitmap pour combiner les index sur `amount` et `status`.

---

#### 3.4 Index Only Scan

- **Principe** : PostgreSQL lit uniquement l'index sans accéder à la table si toutes les colonnes nécessaires sont dans l'index.
- **Quand utilisé** :
  - Colonnes nécessaires déjà stockées dans l'index.
  - Très efficace pour les requêtes qui ne nécessitent que peu de colonnes.
- **Avantages** : lecture très rapide → pas de lecture disque supplémentaire.
- **Inconvénients** : limité aux colonnes présentes dans l'index.

**Exemple :**

```sql
CREATE INDEX idx_orders_amount_status ON orders(amount, status);
SELECT amount, status FROM orders WHERE amount > 1000;
```

PostgreSQL peut satisfaire la requête entièrement via l'index.

---

#### 3.5 Bonnes pratiques

- Vérifier que toutes les colonnes filtrées ou jointes sont correctement indexées (voir [section 4. Indexation avancée](#4-indexation-avancée)).
- Privilégier Index Scan ou Index Only Scan sur les grandes tables pour limiter les lectures disque.
- Pour les filtres combinés, envisager Bitmap Index Scan si plusieurs colonnes sont concernées.
- Utiliser `EXPLAIN ANALYZE` pour vérifier que le scan choisi est adapté à la requête et au volume de données.

**💡 Cas d'usage Data Engineer :** Dans un Data Warehouse, les requêtes analytiques filtrent souvent sur des dates (ex: `WHERE order_date BETWEEN ...`). Un Index Scan ou Bitmap Index Scan sur une colonne de date partitionnée (voir [section 5. Partitionnement](#5-partitionnement)) peut réduire drastiquement le temps d'exécution des requêtes de reporting.

---

### ✅ À retenir

- **Seq Scan** : lecture complète → simple mais coûteux sur grandes tables.
- **Index Scan** : lecture ciblée → rapide si peu de lignes concernées.
- **Bitmap Index Scan** : combine plusieurs index → idéal pour gros volumes et filtres multiples.
- **Index Only Scan** : ultra rapide → tout est déjà dans l'index, pas besoin d'aller chercher les données dans la table.

---

### 4. Indexation avancée

Les index permettent à PostgreSQL de retrouver rapidement des lignes correspondant à un critère donné. Bien comprendre les différents types d'index et leur usage est crucial pour optimiser les requêtes, surtout sur de grandes tables.

---

#### 4.1 Index multicolonnes

- **Principe** : un index peut couvrir plusieurs colonnes.  
- **Ordre des colonnes critique** : l'index est optimisé pour les colonnes les plus sélectives ou les colonnes filtrées en premier.  

**Exemple :**

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

**Utilisation :**

```sql
SELECT * FROM orders
WHERE customer_id = 123
ORDER BY order_date;
```

Ici, l'index est utilisé efficacement car `customer_id` est en premier.

⚠️ **Attention** : si la requête filtre uniquement sur `order_date`, cet index ne sera pas optimal.

---

#### 4.2 Index partiels

- **Principe** : indexe seulement un sous-ensemble de lignes correspondant à une condition.
- **Avantages** : réduit la taille de l'index et accélère les requêtes ciblées.

**Exemple :**

```sql
CREATE INDEX idx_orders_active
ON orders(order_date)
WHERE status = 'active';
```

**Utilisation** : PostgreSQL utilisera cet index uniquement pour les lignes où `status = 'active'`.

Idéal pour les colonnes avec des valeurs majoritaires non pertinentes.

---

#### 4.3 Index sur expressions

- **Principe** : indexe le résultat d'une expression ou d'une fonction appliquée sur une colonne.
- **Avantages** : permet d'optimiser les requêtes utilisant des transformations fréquentes.

**Exemple :**

```sql
CREATE INDEX idx_orders_lower_customer
ON orders(LOWER(customer_name));
```

```sql
SELECT * FROM orders
WHERE LOWER(customer_name) = 'dupont';
```

Ici, la requête peut utiliser l'index directement, sans recalculer la fonction pour chaque ligne.

---

#### 4.4 Index spécialisés : GIN, GiST, BRIN

| Type | Usage principal | Exemples |
|------|----------------|----------|
| **GIN** (Generalized Inverted Index) | Colonnes textuelles, tableaux, JSONB | Recherche de mots-clés, full-text search |
| **GiST** (Generalized Search Tree) | Données géospatiales, hiérarchiques | PostGIS : index spatiaux, polygones |
| **BRIN** (Block Range Index) | Colonnes séquentielles ou très grandes tables | Dates, ID croissants : très léger et rapide pour plages |

**Exemples :**

```sql
-- GIN pour JSONB
CREATE INDEX idx_orders_jsonb
ON orders
USING GIN (data);

-- GiST pour géospatial
CREATE INDEX idx_locations_gist
ON locations
USING GiST (geom);

-- BRIN pour grandes tables
CREATE INDEX idx_orders_brin
ON orders
USING BRIN(order_date);
```

- GIN/GiST sont plus coûteux à créer mais accélèrent énormément les requêtes complexes.
- BRIN est très léger et adapté aux tables massives avec colonnes séquentielles.

---

#### 4.5 Bonnes pratiques

- Choisir le type d'index adapté aux données et aux requêtes les plus fréquentes.
- Pour les tables volumineuses, privilégier les index partiels et BRIN pour réduire la taille.
- Pour les colonnes utilisées dans des fonctions, créer des index sur expressions.
- Toujours tester avec `EXPLAIN ANALYZE` pour vérifier que l'index est effectivement utilisé.

**💡 Cas d'usage Data Engineer :** 
- **Index partiels** : Dans un pipeline ETL, vous pouvez avoir une table de logs avec 99% de lignes `status='processed'` et 1% `status='error'`. Un index partiel `WHERE status='error'` accélère les requêtes de diagnostic sans alourdir les écritures.
- **BRIN** : Pour les tables de time-series (événements, métriques), un index BRIN sur la colonne timestamp est très efficace et léger, même sur des milliards de lignes.
- **GIN pour JSONB** : Si vous stockez des données semi-structurées (logs, événements), un index GIN sur une colonne JSONB permet des recherches rapides dans la structure.

---

### ✅ À retenir

- L'indexation avancée est un levier majeur pour l'optimisation des requêtes sur gros volumes.
- **Multicolonnes** → attention à l'ordre.
- **Partiels** → réduisent la taille et accélèrent les filtres ciblés.
- **Expressions** → optimisent les transformations fréquentes.
- **GIN/GiST/BRIN** → adaptés pour text, spatial ou grandes tables séquentielles.

---

### 5. Partitionnement

Le partitionnement permet de **diviser une table volumineuse en plusieurs tables plus petites (partitions)**, tout en conservant une vue logique unique. Cela réduit les scans inutiles et améliore les performances et la maintenance.

---

#### 5.1 Pourquoi partitionner ?

- Les grandes tables (millions ou milliards de lignes) deviennent **coûteuses à scanner**.  
- Le partitionnement permet de **limiter la lecture** aux partitions pertinentes selon la requête.  
- Facilite la **maintenance** : suppression ou archivage de données anciennes sans toucher aux autres partitions.  
- Améliore le **parallélisme** lors des opérations de lecture ou de VACUUM.

---

#### 5.2 Types de partition

| Type | Description | Exemple d'usage |
|------|------------|----------------|
| **RANGE** | Partition selon des plages de valeurs | Dates : commandes par année |
| **LIST** | Partition selon des valeurs spécifiques | Pays, régions, catégories |
| **HASH** | Partition selon une fonction de hachage | Répartition équilibrée sur IDs |

---

#### 5.3 Création de partitions

**Exemple : partition RANGE par année**

```sql
-- Table principale partitionnée
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE NOT NULL,
    amount NUMERIC,
    status TEXT
) PARTITION BY RANGE (order_date);

-- Partition pour l'année 2025
CREATE TABLE orders_2025 PARTITION OF orders
FOR VALUES FROM ('2025-01-01') TO ('2025-12-31');

-- Partition pour l'année 2026
CREATE TABLE orders_2026 PARTITION OF orders
FOR VALUES FROM ('2026-01-01') TO ('2026-12-31');
```

PostgreSQL choisira automatiquement la partition appropriée selon la valeur de `order_date`.

Seules les partitions pertinentes sont scannées lors de la requête → gain de performance.

---

#### 5.4 Bonnes pratiques

- Choisir le type de partition selon la nature des données et les requêtes : RANGE pour les dates, LIST pour des catégories, HASH pour une répartition uniforme.
- Créer des indexes locaux sur chaque partition si nécessaire (chaque partition peut avoir ses propres indexes).
- Limiter le nombre de partitions : trop de partitions peuvent nuire aux performances du planner.
- Vacuum / analyse : les partitions sont des tables à part entière, pensez à maintenir les statistiques.
- Maintenance : drop ou detach des partitions anciennes plutôt que de supprimer des millions de lignes dans la table principale.

**💡 Cas d'usage Data Engineer :** Le partitionnement est essentiel pour les Data Warehouses. Par exemple, partitionner une table de faits par mois permet :
- **Performance** : Les requêtes filtrant sur une période ne scannent que les partitions concernées.
- **Maintenance** : Archivage facile des données anciennes (détacher/dropper les partitions de plus de 2 ans).
- **Parallélisme** : Chaque partition peut être traitée en parallèle lors des opérations de VACUUM ou d'analyse.
- **Scalabilité** : Gérer des tables de plusieurs milliards de lignes devient possible.

---

#### 5.5 Exemple de requête sur partitions

```sql
SELECT *
FROM orders
WHERE order_date BETWEEN '2025-01-01' AND '2025-06-30';
```

PostgreSQL scannera uniquement la partition `orders_2025`, pas les autres années.

Gain important sur les tables massives.

---

### ✅ À retenir

- Partitionnement = découper une table massive en morceaux logiques pour limiter les scans et faciliter la maintenance.
- Types principaux : RANGE, LIST, HASH.
- Toujours combiner avec des indexes adaptés et une gestion des statistiques pour optimiser les requêtes.

---

### 6. CTE (Common Table Expressions)

Les CTE (Common Table Expressions) permettent de **structurer et simplifier** les requêtes complexes en créant des tables temporaires à usage local dans une requête.

---

#### 6.1 CTE de base

- **Syntaxe générale** :

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

Permet de casser une requête complexe en parties plus lisibles et réutilisables.

**Exemple simple :**

```sql
WITH recent_orders AS (
    SELECT *
    FROM orders
    WHERE order_date >= '2025-01-01'
)
SELECT *
FROM recent_orders
WHERE amount > 1000;
```

Ici, `recent_orders` est utilisé une seule fois dans la requête principale.

---

#### 6.2 CTE matérialisés

- **Principe** : PostgreSQL évalue et stocke le résultat du CTE dans une structure temporaire avant de l'utiliser.
- **Avantages** : utile si vous réutilisez la même CTE plusieurs fois dans une requête complexe.
- **Inconvénients** : peut être coûteux en mémoire et temps si le CTE est volumineux.

**Exemple :**

```sql
WITH recent_orders AS MATERIALIZED (
    SELECT *
    FROM orders
    WHERE order_date >= '2025-01-01'
)
SELECT COUNT(*) FROM recent_orders
UNION ALL
SELECT SUM(amount) FROM recent_orders;
```

Ici, `recent_orders` est calculé une seule fois, puis réutilisé pour plusieurs agrégations.

---

#### 6.3 CTE non matérialisés (inline)

- **Principe** : PostgreSQL intègre le CTE directement dans le plan comme une sous-requête.
- **Avantages** : souvent plus performant pour des CTE simples ou volumineux.
- **Inconvénients** : si vous réutilisez le CTE plusieurs fois, il sera recalculé à chaque utilisation.

**Exemple :**

```sql
WITH recent_orders AS NOT MATERIALIZED (
    SELECT *
    FROM orders
    WHERE order_date >= '2025-01-01'
)
SELECT COUNT(*) FROM recent_orders
UNION ALL
SELECT SUM(amount) FROM recent_orders;
```

Ici, PostgreSQL peut intégrer le CTE dans le plan global pour optimiser l'exécution.

---

#### 6.4 Bonnes pratiques

- Pour des filtres ou transformations simples, privilégier les CTE non matérialisés pour éviter de stocker inutilement de grosses tables temporaires.
- Pour réutilisation multiple de la même CTE, envisager la matérialisation pour ne pas recalculer plusieurs fois.
- Toujours vérifier l'impact avec `EXPLAIN ANALYZE` : certaines requêtes volumineuses peuvent devenir plus lentes avec `MATERIALIZED`.
- Les CTE sont particulièrement utiles pour clarifier des requêtes complexes avant optimisation.

**💡 Cas d'usage Data Engineer :** Dans un pipeline ETL, les CTE permettent de structurer les transformations :
```sql
-- Exemple de pipeline ETL avec CTE
WITH raw_data AS (
    SELECT * FROM staging_table WHERE load_date = CURRENT_DATE
),
cleaned_data AS (
    SELECT 
        id,
        TRIM(LOWER(email)) AS email,
        CAST(amount AS NUMERIC(10,2)) AS amount
    FROM raw_data
    WHERE email IS NOT NULL
),
enriched_data AS (
    SELECT 
        c.*,
        d.region
    FROM cleaned_data c
    LEFT JOIN dimension_table d ON c.customer_id = d.id
)
INSERT INTO fact_table
SELECT * FROM enriched_data;
```
Cette approche facilite le débogage et l'optimisation étape par étape.

---

### ✅ À retenir

- CTE = structure temporaire pour simplifier la lecture et réutilisation des sous-requêtes.
- **Matérialisé** → calculé une fois, utile pour réutilisation, coûte plus de ressources.
- **Non matérialisé** → intégré dans le plan, souvent plus performant sur de gros volumes si non réutilisé.
- Toujours tester avec `EXPLAIN ANALYZE` pour choisir la meilleure option.

---

## ✅ Compétences acquises à l'issue du module
- Lire et interpréter un plan d'exécution PostgreSQL.
- Identifier les goulots d'étranglement et les opérations coûteuses.
- Choisir le bon type de join et de scan.
- Appliquer l'indexation avancée et le partitionnement pour optimiser les requêtes.
- Utiliser explain.dalibo.com pour visualiser et analyser les plans.
- Comprendre l'impact des CTE et des structures SQL sur la performance.

---

### 8. Pièges à éviter et antipatterns

Cette section présente les erreurs courantes à éviter lors de l'optimisation de requêtes PostgreSQL.

---

#### 8.1 Ne pas mettre à jour les statistiques

**Problème :** Les statistiques obsolètes conduisent le planner à choisir de mauvais plans d'exécution.

**Exemple problématique :**
```sql
-- Import massif de données
COPY orders FROM '/data/orders.csv' CSV HEADER;
-- Oubli d'ANALYZE → statistiques obsolètes
```

**Conséquence :** Le planner peut choisir un Seq Scan alors qu'un Index Scan serait 10x plus rapide.

**Solution :**
```sql
ANALYZE orders;  -- Toujours après import massif
```

**💡 Pour Data Engineers :** Dans un pipeline ETL, planifiez un `ANALYZE` automatique après chaque chargement de données volumineuses.

---

#### 8.2 Trop d'index sur une table très modifiée

**Problème :** Chaque index ralentit les INSERT/UPDATE/DELETE car il doit être maintenu.

**Exemple problématique :**
```sql
-- Table avec beaucoup d'écritures (logs, événements)
CREATE INDEX idx_col1 ON events(col1);
CREATE INDEX idx_col2 ON events(col2);
CREATE INDEX idx_col3 ON events(col3);
CREATE INDEX idx_col4 ON events(col4);
CREATE INDEX idx_col5 ON events(col5);
-- 5 index sur une table avec 1000 INSERT/seconde → ralentissement significatif
```

**Conséquence :** Les écritures deviennent très lentes, le système se sature.

**Solution :**
- Indexer uniquement les colonnes vraiment nécessaires
- Utiliser des index partiels pour réduire la taille
- Analyser l'utilisation avec `pg_stat_user_indexes` et supprimer les index inutilisés

---

#### 8.3 Index multicolonnes dans le mauvais ordre

**Problème :** L'ordre des colonnes dans un index multicolonnes est crucial.

**Exemple problématique :**
```sql
-- Mauvais ordre : order_date en premier alors qu'on filtre souvent sur customer_id
CREATE INDEX idx_orders_date_customer ON orders(order_date, customer_id);

-- Requête fréquente
SELECT * FROM orders WHERE customer_id = 123;  -- Index non utilisé efficacement
```

**Solution :**
```sql
-- Bon ordre : colonne la plus sélective en premier
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
```

**Règle :** La première colonne doit être celle sur laquelle on filtre le plus souvent, ou la plus sélective.

---

#### 8.4 Fonctions dans WHERE sans index sur expression

**Problème :** Utiliser une fonction dans WHERE empêche l'utilisation d'un index standard.

**Exemple problématique :**
```sql
-- Index standard
CREATE INDEX idx_orders_email ON orders(email);

-- Requête avec fonction
SELECT * FROM orders WHERE LOWER(email) = 'test@example.com';
-- Index non utilisé → Seq Scan
```

**Solution :**
```sql
-- Index sur expression
CREATE INDEX idx_orders_lower_email ON orders(LOWER(email));
-- Maintenant la requête utilise l'index
```

---

#### 8.5 SELECT * sur de grandes tables

**Problème :** Récupérer toutes les colonnes peut empêcher l'Index Only Scan.

**Exemple problématique :**
```sql
-- Index sur (amount, status)
CREATE INDEX idx_orders_amount_status ON orders(amount, status);

-- Requête
SELECT * FROM orders WHERE amount > 1000;
-- Index Only Scan impossible car toutes les colonnes sont demandées
```

**Solution :**
```sql
-- Ne sélectionner que les colonnes nécessaires
SELECT amount, status FROM orders WHERE amount > 1000;
-- Index Only Scan possible → beaucoup plus rapide
```

**💡 Pour Data Engineers :** Dans les pipelines ETL, évitez `SELECT *` et spécifiez explicitement les colonnes nécessaires. Cela améliore les performances et la maintenabilité.

---

#### 8.6 OR multiples sans index adapté

**Problème :** Plusieurs conditions OR peuvent empêcher l'utilisation efficace des index.

**Exemple problématique :**
```sql
SELECT * FROM orders 
WHERE status = 'active' OR status = 'pending' OR status = 'processing';
-- Peut forcer un Seq Scan même avec un index sur status
```

**Solution :**
```sql
-- Utiliser IN à la place
SELECT * FROM orders 
WHERE status IN ('active', 'pending', 'processing');
-- Plus efficace, peut utiliser un Index Scan ou Bitmap Scan
```

---

#### 8.7 Partitionnement sans filtre sur la colonne de partition

**Problème :** Oublier de filtrer sur la colonne de partition annule les bénéfices.

**Exemple problématique :**
```sql
-- Table partitionnée par order_date
SELECT * FROM orders WHERE customer_id = 123;
-- Scanne TOUTES les partitions → pas de gain
```

**Solution :**
```sql
-- Toujours inclure un filtre sur la colonne de partition
SELECT * FROM orders 
WHERE customer_id = 123 
  AND order_date >= '2025-01-01' AND order_date < '2025-02-01';
-- Scanne uniquement la partition concernée
```

**💡 Pour Data Engineers :** Dans un Data Warehouse, toujours inclure un filtre de date dans les requêtes pour bénéficier du partitionnement.

---

#### 8.8 CTE matérialisés inutiles

**Problème :** Forcer la matérialisation d'un CTE simple peut ralentir la requête.

**Exemple problématique :**
```sql
-- CTE simple, utilisé une seule fois
WITH filtered_orders AS MATERIALIZED (
    SELECT * FROM orders WHERE amount > 1000
)
SELECT COUNT(*) FROM filtered_orders;
-- Matérialisation inutile → surcoût mémoire et temps
```

**Solution :**
```sql
-- Laisser PostgreSQL décider (ou NOT MATERIALIZED si nécessaire)
WITH filtered_orders AS (
    SELECT * FROM orders WHERE amount > 1000
)
SELECT COUNT(*) FROM filtered_orders;
-- PostgreSQL optimise automatiquement
```

---

#### 8.9 Ignorer les signaux d'alerte dans EXPLAIN ANALYZE

**Problème :** Ne pas analyser les écarts entre estimations et réalité.

**Signaux d'alerte :**
- `rows` estimées très différentes de `actual rows` (> 2x)
- `cost` très différent de `actual time` (> 3x)
- `loops` très élevés (indique un join inefficace)

**Action :** Toujours investiguer ces écarts, ils indiquent des problèmes de statistiques ou de plan.

---

### ✅ Checklist d'évitement des pièges

Avant de déployer une optimisation en production :

- [ ] Statistiques à jour (`ANALYZE` récent)
- [ ] Index réellement utilisés (vérifier avec `EXPLAIN ANALYZE`)
- [ ] Pas trop d'index sur tables très modifiées
- [ ] Ordre des colonnes correct dans les index multicolonnes
- [ ] Index sur expressions pour les fonctions dans WHERE
- [ ] Filtre sur colonne de partition si table partitionnée
- [ ] Pas de `SELECT *` inutile
- [ ] Écarts estimation/réalité analysés et corrigés

---

### 9. Cas d'usage complets : du problème à la solution

Cette section présente des cas d'usage réels de Data Engineers, de l'identification du problème à la solution optimisée.

---

#### 9.1 Cas d'usage 1 : Requête ETL lente sur table de logs

**Contexte :** Pipeline ETL quotidien qui charge des logs dans une table PostgreSQL. La requête de transformation est très lente.

**Problème initial :**
```sql
-- Requête lente (30 secondes)
SELECT 
    DATE_TRUNC('hour', log_timestamp) AS hour,
    COUNT(*) AS event_count,
    AVG(duration) AS avg_duration
FROM event_logs
WHERE log_timestamp >= CURRENT_DATE - INTERVAL '7 days'
  AND log_type = 'api_call'
GROUP BY DATE_TRUNC('hour', log_timestamp)
ORDER BY hour;
```

**Diagnostic avec EXPLAIN ANALYZE :**
```
Seq Scan on event_logs  (cost=0.00..125000.00 rows=500000 width=16) 
  (actual time=0.123..28500.456 rows=450000 loops=1)
  Filter: ((log_timestamp >= ...) AND (log_type = 'api_call'::text))
  Rows Removed by Filter: 5500000
```

**Analyse :**
- Seq Scan sur 6 millions de lignes
- 5.5 millions de lignes filtrées (inefficace)
- Temps d'exécution : 28.5 secondes

**Solutions appliquées :**

1. **Créer un index composite :**
```sql
CREATE INDEX idx_logs_timestamp_type 
ON event_logs(log_timestamp, log_type);
```

2. **Partitionner par date (si applicable) :**
```sql
-- Si la table n'est pas déjà partitionnée
ALTER TABLE event_logs 
PARTITION BY RANGE (log_timestamp);
```

3. **Mettre à jour les statistiques :**
```sql
ANALYZE event_logs;
```

**Résultat après optimisation :**
```
Index Scan using idx_logs_timestamp_type on event_logs  
  (cost=0.43..12500.00 rows=500000 width=16) 
  (actual time=0.045..3200.123 rows=450000 loops=1)
  Index Cond: ((log_timestamp >= ...) AND (log_type = 'api_call'::text))
```

**Gain :** 28.5 secondes → 3.2 secondes (gain de 89%)

---

#### 9.2 Cas d'usage 2 : Join coûteux dans un Data Warehouse

**Contexte :** Requête de reporting qui joint une table de faits avec plusieurs dimensions. Performance dégradée.

**Problème initial :**
```sql
-- Requête lente (2 minutes)
SELECT 
    d.region,
    d.product_category,
    SUM(f.sales_amount) AS total_sales,
    COUNT(*) AS transaction_count
FROM fact_sales f
JOIN dim_customers c ON f.customer_id = c.customer_id
JOIN dim_products d ON f.product_id = d.product_id
WHERE f.sale_date >= '2025-01-01'
  AND f.sale_date < '2025-02-01'
GROUP BY d.region, d.product_category;
```

**Diagnostic avec EXPLAIN ANALYZE :**
```
Nested Loop  (cost=0.00..250000.00 rows=1000000) 
  (actual time=0.123..120000.456 rows=500000 loops=1000)
  -> Seq Scan on fact_sales f
  -> Index Scan using idx_customers_id on dim_customers c
```

**Analyse :**
- Nested Loop avec 1000 loops (très coûteux)
- Seq Scan sur la table de faits (pas d'index sur sale_date)
- Temps d'exécution : 120 secondes

**Solutions appliquées :**

1. **Indexer la colonne de jointure et de filtre :**
```sql
CREATE INDEX idx_fact_sales_date_customer 
ON fact_sales(sale_date, customer_id);
CREATE INDEX idx_fact_sales_product 
ON fact_sales(product_id);
```

2. **Partitionner la table de faits par date :**
```sql
-- Si pas déjà fait
ALTER TABLE fact_sales 
PARTITION BY RANGE (sale_date);
```

3. **Forcer un Hash Join si approprié :**
```sql
SET enable_nestloop = off;  -- Temporairement pour tester
-- Ou laisser PostgreSQL choisir avec de meilleures statistiques
```

**Résultat après optimisation :**
```
Hash Join  (cost=15000.00..25000.00 rows=1000000) 
  (actual time=500.123..8500.456 rows=500000 loops=1)
  Hash Cond: (f.customer_id = c.customer_id)
  -> Index Scan using idx_fact_sales_date_customer on fact_sales f
  -> Hash
     -> Seq Scan on dim_customers c
```

**Gain :** 120 secondes → 8.5 secondes (gain de 93%)

---

#### 9.3 Cas d'usage 3 : Requête analytique sur JSONB

**Contexte :** Table stockant des événements en JSONB. Requêtes de recherche lentes.

**Problème initial :**
```sql
-- Requête lente (15 secondes)
SELECT 
    event_id,
    event_data->>'user_id' AS user_id,
    event_data->>'action' AS action
FROM events
WHERE event_data->>'action' = 'purchase'
  AND event_data->>'timestamp' >= '2025-01-01';
```

**Diagnostic :**
```
Seq Scan on events  (cost=0.00..50000.00 rows=10000 width=64) 
  (actual time=0.123..15000.456 rows=8500 loops=1)
  Filter: (((event_data ->> 'action'::text) = 'purchase'::text) AND ...)
```

**Solutions appliquées :**

1. **Créer un index GIN sur JSONB :**
```sql
CREATE INDEX idx_events_data_gin ON events USING GIN (event_data);
```

2. **Utiliser les opérateurs JSONB optimisés :**
```sql
-- Requête optimisée
SELECT 
    event_id,
    event_data->>'user_id' AS user_id,
    event_data->>'action' AS action
FROM events
WHERE event_data @> '{"action": "purchase"}'::jsonb
  AND (event_data->>'timestamp')::timestamp >= '2025-01-01'::timestamp;
```

**Résultat après optimisation :**
```
Bitmap Index Scan on idx_events_data_gin  (cost=0.00..500.00 rows=10000) 
  (actual time=0.045..250.123 rows=8500 loops=1)
  Index Cond: (event_data @> '{"action": "purchase"}'::jsonb)
```

**Gain :** 15 secondes → 0.25 secondes (gain de 98%)

---

### 10. Ressources complémentaires

Pour approfondir vos connaissances sur l'optimisation PostgreSQL, voici des ressources recommandées.

---

#### 10.1 Documentation officielle

- **PostgreSQL Documentation - Performance Tips**  
  https://www.postgresql.org/docs/current/performance-tips.html

- **PostgreSQL Documentation - Using EXPLAIN**  
  https://www.postgresql.org/docs/current/using-explain.html

- **PostgreSQL Documentation - Indexes**  
  https://www.postgresql.org/docs/current/indexes.html

- **PostgreSQL Documentation - Partitioning**  
  https://www.postgresql.org/docs/current/ddl-partitioning.html

---

#### 10.2 Outils et extensions

- **explain.dalibo.com** : Visualisation graphique des plans d'exécution  
  https://explain.dalibo.com/

- **pg_stat_statements** : Extension pour analyser les requêtes les plus coûteuses  
  https://www.postgresql.org/docs/current/pgstatstatements.html

- **pgBadger** : Analyseur de logs PostgreSQL  
  https://pgbadger.darold.net/

- **pgAdmin** : Interface graphique avec visualisation de plans  
  https://www.pgadmin.org/

---

#### 10.3 Articles et blogs recommandés

- **2ndQuadrant Blog** : Articles sur l'optimisation PostgreSQL  
  https://www.2ndquadrant.com/en/blog/

- **Depesz Blog** : Analyses approfondies de plans d'exécution  
  https://www.depesz.com/

- **PostgreSQL Performance** : Guide complet sur les performances  
  https://wiki.postgresql.org/wiki/Performance_Optimization

---

#### 10.4 Livres recommandés

- **"PostgreSQL: Up and Running"** par Regina Obe & Leo Hsu
- **"PostgreSQL High Performance"** par Gregory Smith
- **"Mastering PostgreSQL in Application Development"** par Dimitri Fontaine

---

#### 10.5 Communautés et forums

- **PostgreSQL Mailing Lists** : Discussions techniques  
  https://www.postgresql.org/list/

- **Stack Overflow** : Questions/réponses PostgreSQL  
  https://stackoverflow.com/questions/tagged/postgresql

- **Reddit r/PostgreSQL** : Communauté active  
  https://www.reddit.com/r/PostgreSQL/

---

#### 10.6 Formations et certifications

- **PostgreSQL University** : Formations officielles  
  https://postgresql.org/about/training/

- **Cours en ligne** : Udemy, Coursera, Pluralsight proposent des formations PostgreSQL avancées

---

### 📚 Navigation dans le module

Pour faciliter votre navigation, voici les liens vers les sections principales :

- [1. EXPLAIN / EXPLAIN ANALYZE](#1-explain--explain-analyze)
- [2. Types de joins](#2-types-de-joins)
- [3. Types de scans](#3-types-de-scans)
- [4. Indexation avancée](#4-indexation-avancée)
- [5. Partitionnement](#5-partitionnement)
- [6. CTE (Common Table Expressions)](#6-cte-common-table-expressions)
- [7. Cas pratiques & exercices](#7-cas-pratiques--exercices)
- [8. Pièges à éviter et antipatterns](#8-pièges-à-éviter-et-antipatterns)
- [9. Cas d'usage complets](#9-cas-dusage-complets--du-problème-à-la-solution)
- [10. Ressources complémentaires](#10-ressources-complémentaires)

---




### 7. Cas pratiques & exercices

#### 7.1 Exercices sur EXPLAIN / EXPLAIN ANALYZE

**Objectifs :**  
- Savoir générer et lire un plan d'exécution PostgreSQL.  
- Identifier les types de scans et les nœuds coûteux.  
- Comparer les estimations et les valeurs réelles pour comprendre les goulots d'étranglement.
- Comprendre l'impact des index sur les performances.
- Utiliser explain.dalibo.com pour visualiser et analyser les plans.

---

**Exercice 1 : Plan d'exécution simple**

1. Créez la table `orders` et insérez quelques données factices :

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount NUMERIC,
    status TEXT
);

INSERT INTO orders (customer_id, order_date, amount, status)
VALUES
(1, '2025-01-05', 500, 'active'),
(2, '2025-02-10', 1200, 'inactive'),
(3, '2025-03-15', 800, 'active');
```

2. Exécutez la requête suivante avec `EXPLAIN` :

```sql
EXPLAIN
SELECT *
FROM orders
WHERE amount > 700;
```

**Questions :**
- Quel type de scan est utilisé ? Pourquoi PostgreSQL a-t-il choisi ce type de scan ?
- Quelle est l'estimation du nombre de lignes retournées (`rows`) ?
- Quel est le coût estimé (`cost`) ? Interprétez la plage de coût (ex: `cost=0.00..431.00`).

3. Comparez avec `EXPLAIN ANALYZE` :

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE amount > 700;
```

**Questions :**
- Combien de lignes ont été réellement retournées (`actual rows`) ?
- Combien de boucles (`loops`) ont été effectuées ?
- Y a-t-il un écart significatif entre l'estimation et le réel ? Si oui, que cela indique-t-il ?
- Notez le `actual time` et comparez-le avec le `cost` estimé.

---

**Exercice 2 : Observation de l'impact des index**

1. Créez un index sur la colonne `amount` :

```sql
CREATE INDEX idx_orders_amount ON orders(amount);
```

2. Réexécutez `EXPLAIN ANALYZE` pour la même requête :

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE amount > 700;
```

**Questions :**
- Le type de scan a-t-il changé ? Si oui, pourquoi ?
- Quelle est la différence en `actual time` par rapport au plan précédent ? Calculez le gain de performance.
- Comparez les `cost` estimés avant et après l'index. Que constatez-vous ?
- Dans quel cas un Seq Scan serait-il préférable à un Index Scan sur cette table ?

---

**Exercice 3 : JSON pour explain.dalibo.com**

1. Générez le plan JSON pour la même requête :

```sql
EXPLAIN (ANALYZE, COSTS, BUFFERS, FORMAT JSON)
SELECT *
FROM orders
WHERE amount > 700;
```

2. Copiez le JSON et collez-le dans [https://explain.dalibo.com/](https://explain.dalibo.com/) pour visualiser le plan graphique.

**Questions :**
- Identifiez le nœud le plus coûteux dans l'arbre visuel.
- Comparez les `Total cost` et `Actual time`. Y a-t-il une corrélation ?
- Analysez les `Buffers` : combien de lectures disque ont été effectuées ?
- Que pourriez-vous optimiser pour améliorer la performance ? Proposez au moins 2 optimisations possibles.

---

**Exercice 4 : Cas réel avec gros volumes (Data Engineer)**

Dans un contexte Data Engineer, vous travaillez souvent avec des tables contenant des millions de lignes. Cet exercice simule cette situation.

1. Générez un dataset volumineux (10 000 lignes) :

```sql
-- Vider la table si nécessaire
TRUNCATE TABLE orders;

-- Générer 10 000 commandes factices
INSERT INTO orders (customer_id, order_date, amount, status)
SELECT 
    (random() * 1000)::int AS customer_id,
    '2025-01-01'::date + (random() * 365)::int AS order_date,
    (random() * 5000)::numeric(10,2) AS amount,
    CASE WHEN random() > 0.5 THEN 'active' ELSE 'inactive' END AS status
FROM generate_series(1, 10000);

-- Mettre à jour les statistiques
ANALYZE orders;
```

2. Exécutez `EXPLAIN ANALYZE` sur une requête filtrant sur `amount` :

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE amount > 3000;
```

**Questions :**
- Quel type de scan est utilisé ? Pourquoi ?
- Notez le `actual time` et le nombre de `rows` retournées.
- Combien de lignes ont été scannées au total ? (Regardez les `actual rows` dans le plan)

3. Créez un index sur `amount` si ce n'est pas déjà fait :

```sql
CREATE INDEX IF NOT EXISTS idx_orders_amount ON orders(amount);
ANALYZE orders;
```

4. Réexécutez la même requête avec `EXPLAIN ANALYZE` :

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE amount > 3000;
```

**Questions :**
- Le type de scan a-t-il changé ? Si oui, quel est le nouveau type ?
- Comparez les métriques avant/après index :
  - `actual time` : gain de temps ?
  - `cost` : réduction du coût ?
  - `rows` : même nombre de lignes retournées ?
- Calculez le ratio d'amélioration : `(temps_sans_index / temps_avec_index)`
- Dans un contexte ETL avec des millions de lignes, quel serait l'impact de cet index sur un pipeline quotidien ?

5. Testez avec une requête plus sélective (moins de lignes retournées) :

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE amount > 4500;
```

**Questions :**
- Comparez les performances avec la requête précédente (`amount > 3000`).
- Que pouvez-vous conclure sur l'impact de la sélectivité sur le choix du scan ?
- À partir de quel pourcentage de lignes retournées un Seq Scan devient-il préférable à un Index Scan ?

---

---

## 📝 Réponses attendues et explications détaillées

### Exercice 1 : Plan d'exécution simple

#### Plan d'exécution typique avec EXPLAIN

```
Seq Scan on orders  (cost=0.00..18.10 rows=2 width=40)
  Filter: (amount > 700::numeric)
```

#### Analyse détaillée

**1. Type de scan : Seq Scan**

PostgreSQL choisit un **Sequential Scan** car :
- La table ne contient que 3 lignes (trop petite pour qu'un index soit rentable).
- Le coût de lecture de l'index serait supérieur au coût de lecture séquentielle de la table entière.
- Règle générale : sur des tables de moins de ~100 lignes, PostgreSQL préfère souvent un Seq Scan.

**2. Interprétation du coût (`cost=0.00..18.10`)**

- **0.00** (startup cost) : coût avant d'obtenir la première ligne. Ici, très faible car pas de tri ni de préparation complexe.
- **18.10** (total cost) : coût total estimé pour retourner toutes les lignes.
- **Unité** : le coût est relatif, basé sur des paramètres de configuration (`seq_page_cost`, `cpu_tuple_cost`, etc.).
- **Comparaison** : un coût de 18.10 est très faible. En production, on s'inquiète généralement pour des coûts > 1000-10000.

**3. Rows estimées (`rows=2`)**

- PostgreSQL estime que 2 lignes sur 3 correspondent au filtre `amount > 700`.
- Cette estimation est basée sur les statistiques collectées par `ANALYZE`.
- Sur une table de 3 lignes, l'estimation est approximative car les statistiques sont limitées.

**4. Plan avec EXPLAIN ANALYZE**

```
Seq Scan on orders  (cost=0.00..18.10 rows=2 width=40) 
  (actual time=0.012..0.015 rows=2 loops=1)
  Filter: (amount > 700::numeric)
  Rows Removed by Filter: 1
```

**Analyse des métriques réelles :**
- **actual time=0.012..0.015** : 
  - 0.012 ms pour obtenir la première ligne
  - 0.015 ms pour obtenir toutes les lignes
  - Temps très faible car la table est minuscule
- **actual rows=2** : 2 lignes retournées (correspond à l'estimation)
- **loops=1** : le scan a été exécuté une seule fois
- **Rows Removed by Filter: 1** : 1 ligne a été filtrée (celle avec `amount=500`)

**5. Écart estimation/réel**

Sur cette table de 3 lignes :
- L'écart est négligeable car la table est trop petite pour que les statistiques soient significatives.
- **En production** : un écart important entre `rows` et `actual rows` (ex: 1000 estimées vs 10000 réelles) indique :
  - Des statistiques obsolètes → exécuter `ANALYZE table_name`
  - Une distribution de données non uniforme → ajuster `default_statistics_target`
  - Un problème de sélectivité → vérifier les index et les filtres

---

### Exercice 2 : Observation de l'impact des index

#### Plan avant index (Seq Scan)

```
Seq Scan on orders  (cost=0.00..18.10 rows=2 width=40) 
  (actual time=0.012..0.015 rows=2 loops=1)
  Filter: (amount > 700::numeric)
```

#### Plan après index (sur table de 3 lignes)

```
Seq Scan on orders  (cost=0.00..18.10 rows=2 width=40) 
  (actual time=0.011..0.014 rows=2 loops=1)
  Filter: (amount > 700::numeric)
```

**Observation importante :** Sur une table de 3 lignes, PostgreSQL peut **toujours choisir un Seq Scan** même avec un index, car :
- Le coût de lecture de l'index (2-3 pages) + accès à la table est supérieur au coût de lecture séquentielle (1 page).
- C'est un comportement normal et optimal du planner.

#### Plan après index (sur table de 10 000 lignes)

Sur une table plus grande, vous verriez :

**Sans index :**
```
Seq Scan on orders  (cost=0.00..431.00 rows=2000 width=40) 
  (actual time=0.123..45.234 rows=1987 loops=1)
  Filter: (amount > 3000::numeric)
  Rows Removed by Filter: 8013
```

**Avec index :**
```
Index Scan using idx_orders_amount on orders  
  (cost=0.29..85.47 rows=2000 width=40) 
  (actual time=0.045..12.567 rows=1987 loops=1)
  Index Cond: (amount > 3000::numeric)
```

#### Analyse comparative

**1. Changement de scan**

- **Avant** : Seq Scan (lecture de toutes les lignes)
- **Après** : Index Scan (lecture ciblée via l'index)
- **Pourquoi** : sur 10 000 lignes, l'index permet de localiser rapidement les ~2000 lignes concernées sans scanner les 8000 autres.

**2. Gain de performance**

**Métriques comparatives :**
- **Coût estimé** : 431.00 → 85.47 (réduction de ~80%)
- **Temps réel** : 45.234 ms → 12.567 ms (réduction de ~72%)
- **Lignes scannées** : 10 000 → ~2000 (réduction de 80%)

**Calcul du ratio d'amélioration :**
```
Ratio = temps_sans_index / temps_avec_index
Ratio = 45.234 / 12.567 ≈ 3.6x plus rapide
```

**3. Quand un Seq Scan est préférable**

Un Seq Scan devient préférable quand :
- **Plus de 5-10% de la table est retournée** : le coût de lecture séquentielle devient inférieur au coût d'accès indexé.
- **Table très petite** (< 100 lignes) : overhead de l'index non rentable.
- **Pas d'index disponible** : évidemment, PostgreSQL n'a pas le choix.
- **Requête nécessite la majorité des colonnes** : l'Index Scan nécessite des accès supplémentaires à la table.

**Exemple concret :**
```sql
-- Retourne 80% de la table → Seq Scan préféré
SELECT * FROM orders WHERE amount > 100;

-- Retourne 2% de la table → Index Scan préféré
SELECT * FROM orders WHERE amount > 4500;
```

---

### Exercice 3 : JSON pour explain.dalibo.com

#### Plan JSON typique (extrait)

```json
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Relation Name": "orders",
      "Alias": "orders",
      "Startup Cost": 0.00,
      "Total Cost": 431.00,
      "Plan Rows": 2000,
      "Plan Width": 40,
      "Actual Startup Time": 0.123,
      "Actual Total Time": 45.234,
      "Actual Rows": 1987,
      "Actual Loops": 1,
      "Filter": "(amount > 3000::numeric)"
    }
  }
]
```

#### Analyse dans explain.dalibo.com

**1. Identification du nœud coûteux**

- **Nœud racine** : généralement le nœud le plus coûteux (ici, le Seq Scan).
- **Visualisation** : dans l'arbre graphique, les nœuds coûteux sont souvent colorés en rouge/orange.
- **Métrique clé** : regarder le `Total Cost` et `Actual Total Time`.

**2. Corrélation cost vs actual time**

**Cas normal (statistiques à jour) :**
- `Total Cost`: 431.00
- `Actual Total Time`: 45.234 ms
- **Ratio** : cost/time ≈ 9.5 (cohérent)

**Cas problématique (statistiques obsolètes) :**
- `Total Cost`: 431.00
- `Actual Total Time`: 450.234 ms
- **Ratio** : cost/time ≈ 0.96 (écart important → statistiques à mettre à jour)

**Interprétation :**
- Le `cost` est une **estimation** basée sur des modèles mathématiques.
- Le `actual time` est la **réalité mesurée**.
- Un écart important (> 2-3x) indique un problème de statistiques ou de configuration.

**3. Analyse des Buffers**

Avec `BUFFERS` activé, vous verrez :
```json
"Shared Hit Blocks": 25,
"Shared Read Blocks": 0,
"Shared Dirtied Blocks": 0,
"Shared Written Blocks": 0
```

**Interprétation :**
- **Shared Hit Blocks** : lectures depuis le cache (rapide)
- **Shared Read Blocks** : lectures depuis le disque (lent)
- **Ratio Hit/Read** : plus le ratio est élevé, mieux c'est (cache efficace)

**4. Optimisations possibles**

**Optimisation 1 : Créer un index**
```sql
CREATE INDEX idx_orders_amount ON orders(amount);
```
- Impact : transformation d'un Seq Scan en Index Scan
- Gain attendu : 5-10x sur requêtes sélectives

**Optimisation 2 : Améliorer la sélectivité**
- Rendre le filtre plus sélectif si possible
- Exemple : `amount > 3000` au lieu de `amount > 100`

**Optimisation 3 : Mettre à jour les statistiques**
```sql
ANALYZE orders;
```
- Impact : amélioration de la précision des estimations
- Fréquence : après import massif, après modifications importantes

**Optimisation 4 : Index partiel (si applicable)**
```sql
CREATE INDEX idx_orders_high_amount 
ON orders(amount) 
WHERE amount > 1000;
```
- Impact : index plus petit, plus rapide à maintenir
- Cas d'usage : quand une grande partie des données n'est jamais interrogée

---

### Exercice 4 : Cas réel avec gros volumes (Data Engineer)

#### Scénario : Table de 10 000 lignes

**1. Plan sans index**

```
Seq Scan on orders  (cost=0.00..431.00 rows=2000 width=40) 
  (actual time=0.123..45.234 rows=1987 loops=1)
  Filter: (amount > 3000::numeric)
  Rows Removed by Filter: 8013
```

**Analyse détaillée :**
- **Coût** : 431.00 (élevé car scanne toutes les lignes)
- **Temps réel** : 45.234 ms
- **Lignes scannées** : 10 000 (toutes)
- **Lignes retournées** : 1987 (~20% de sélectivité)
- **Efficacité** : 80% des lignes sont filtrées (inefficace)

**2. Plan avec index**

```
Index Scan using idx_orders_amount on orders  
  (cost=0.29..85.47 rows=2000 width=40) 
  (actual time=0.045..12.567 rows=1987 loops=1)
  Index Cond: (amount > 3000::numeric)
```

**Analyse détaillée :**
- **Coût** : 85.47 (réduction de 80%)
- **Temps réel** : 12.567 ms (réduction de 72%)
- **Lignes scannées** : ~2000 (via index, puis accès table)
- **Lignes retournées** : 1987
- **Efficacité** : scanne uniquement les lignes pertinentes

**3. Comparaison quantitative**

| Métrique | Sans index | Avec index | Amélioration |
|----------|------------|------------|--------------|
| **Coût estimé** | 431.00 | 85.47 | -80% |
| **Temps réel** | 45.234 ms | 12.567 ms | -72% |
| **Lignes scannées** | 10 000 | ~2 000 | -80% |
| **Ratio d'amélioration** | 1x | 3.6x | **+260%** |

**4. Impact de la sélectivité**

**Requête peu sélective (`amount > 100` - retourne 80% des lignes) :**

```
Seq Scan on orders  (cost=0.00..431.00 rows=8000 width=40) 
  (actual time=0.123..38.456 rows=7987 loops=1)
  Filter: (amount > 100::numeric)
```

- PostgreSQL choisit **Seq Scan** car 80% de la table est retournée.
- Un Index Scan serait moins efficace (trop d'accès à la table).

**Requête très sélective (`amount > 4500` - retourne 2% des lignes) :**

```
Index Scan using idx_orders_amount on orders  
  (cost=0.29..12.34 rows=200 width=40) 
  (actual time=0.045..2.123 rows=187 loops=1)
  Index Cond: (amount > 4500::numeric)
```

- PostgreSQL choisit **Index Scan** car seulement 2% de la table est retournée.
- Gain encore plus important : 2.123 ms vs 45.234 ms (21x plus rapide).

**5. Seuil de sélectivité**

**Règle empirique :**
- **< 5% de lignes retournées** : Index Scan généralement préféré
- **5-10% de lignes retournées** : dépend de la taille de la table et des index
- **> 10% de lignes retournées** : Seq Scan souvent préféré

**Calcul pour votre cas :**
```sql
-- Sélectivité = lignes retournées / lignes totales
-- amount > 3000 : 1987 / 10000 = 19.87% → Seq Scan peut être préféré
-- amount > 4500 : 187 / 10000 = 1.87% → Index Scan préféré
```

**6. Impact ETL en production**

**Scénario réel : Pipeline quotidien avec 10 millions de lignes**

**Sans index :**
- Temps d'exécution : ~10 minutes
- Ressources : CPU élevé, I/O disque important
- Impact : pipeline ralentit les traitements suivants

**Avec index :**
- Temps d'exécution : ~1 minute
- Ressources : CPU modéré, I/O disque réduit
- Impact : pipeline efficace, autres traitements non impactés

**Calcul du gain :**
```
Gain = (temps_sans_index - temps_avec_index) / temps_sans_index × 100
Gain = (10 - 1) / 10 × 100 = 90%
```

**Coût de maintenance de l'index :**
- Création : ~2-5 minutes (une fois)
- Maintenance : +10-20% sur les INSERT/UPDATE (acceptable)
- **ROI** : gain de 9 minutes par exécution → très rentable sur un pipeline quotidien

**7. Recommandations Data Engineer**

**Quand créer un index :**
- Colonnes fréquemment filtrées dans les requêtes ETL
- Clés de jointure pour optimiser les JOIN
- Colonnes utilisées dans ORDER BY sur gros volumes
- Colonnes avec forte sélectivité (< 10%)

**Quand éviter un index :**
- Colonnes très peu sélectives (ex: booléen avec 50/50)
- Tables très petites (< 1000 lignes)
- Colonnes très fréquemment modifiées (coût de maintenance élevé)
- Tables avec beaucoup d'INSERT (ralentissement des écritures)

**Monitoring en production :**
```sql
-- Vérifier l'utilisation des index
SELECT * FROM pg_stat_user_indexes 
WHERE idx_scan = 0;  -- Index jamais utilisés

-- Vérifier les statistiques
SELECT schemaname, tablename, last_analyze, last_autoanalyze
FROM pg_stat_user_tables
WHERE last_analyze IS NULL OR last_analyze < NOW() - INTERVAL '7 days';
```

---

## 💡 Conseils avancés pour l'autoformation

### 1. Expérimenter avec différents volumes

**Progression recommandée :**
- **100 lignes** : comprendre les bases, voir les plans simples
- **1 000 lignes** : observer les premiers gains avec index
- **10 000 lignes** : voir l'impact réel des optimisations
- **100 000 lignes** : simuler des cas de production
- **1 000 000+ lignes** : comprendre les défis des très gros volumes

**Script pour générer différents volumes :**
```sql
-- Ajuster le nombre dans generate_series(1, N)
INSERT INTO orders (customer_id, order_date, amount, status)
SELECT 
    (random() * 1000)::int,
    '2025-01-01'::date + (random() * 365)::int,
    (random() * 5000)::numeric(10,2),
    CASE WHEN random() > 0.5 THEN 'active' ELSE 'inactive' END
FROM generate_series(1, 100000);  -- Changer le nombre ici
```

### 2. Tester différentes sélectivités

**Stratégie d'expérimentation :**
- **10% de sélectivité** : `WHERE amount > 4500` (sur 10k lignes = ~1000 lignes)
- **50% de sélectivité** : `WHERE amount > 2500` (sur 10k lignes = ~5000 lignes)
- **90% de sélectivité** : `WHERE amount > 500` (sur 10k lignes = ~9000 lignes)

**Observer :**
- Le choix du planner (Seq Scan vs Index Scan)
- Les temps d'exécution
- Les coûts estimés

### 3. Comparer EXPLAIN vs EXPLAIN ANALYZE

**Méthodologie :**
1. Exécuter `EXPLAIN` pour voir les estimations
2. Exécuter `EXPLAIN ANALYZE` pour voir la réalité
3. Comparer `rows` vs `actual rows`
4. Comparer `cost` vs `actual time`

**Signaux d'alerte :**
- Écart `rows` > 2x : statistiques à mettre à jour
- Écart `cost/time` > 3x : problème de configuration ou statistiques
- `actual time` beaucoup plus élevé que prévu : goulot d'étranglement à identifier

### 4. Utiliser \timing dans psql

**Activation :**
```sql
\timing on
```

**Utilisation :**
```sql
SELECT * FROM orders WHERE amount > 3000;
-- Affiche : Time: 45.234 ms
```

**Avantages :**
- Mesure le temps réel d'exécution (inclut le temps réseau, affichage, etc.)
- Plus simple que `EXPLAIN ANALYZE` pour des comparaisons rapides
- Utile pour mesurer l'impact global d'une optimisation

### 5. Bonnes pratiques de diagnostic

**Checklist d'optimisation :**
- [ ] Statistiques à jour (`ANALYZE` récent)
- [ ] Index existants et utilisés (`pg_stat_user_indexes`)
- [ ] Sélectivité appropriée (filtres efficaces)
- [ ] Plans d'exécution analysés (`EXPLAIN ANALYZE`)
- [ ] Métriques comparées (avant/après optimisation)
- [ ] Impact mesuré en production (monitoring)

**Outils complémentaires :**
- `pg_stat_statements` : identifier les requêtes les plus coûteuses
- `explain.dalibo.com` : visualisation graphique des plans
- `pgBadger` : analyse des logs PostgreSQL
- Monitoring (Grafana, Datadog) : suivi des performances en temps réel