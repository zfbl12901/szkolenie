# 3. Types de données et Tables

## 🎯 Objectifs

- Comprendre les types de données ClickHouse
- Créer des tables optimisées
- Choisir le bon moteur de table
- Configurer le partitionnement

## Types de données

### Types entiers

```sql
UInt8, UInt16, UInt32, UInt64  -- Entiers non signés
Int8, Int16, Int32, Int64      -- Entiers signés
```

### Types décimaux

```sql
Float32, Float64               -- Nombres flottants
Decimal32, Decimal64, Decimal128  -- Nombres décimaux précis
```

### Types de chaînes

```sql
String                        -- Chaîne de caractères
FixedString(N)                -- Chaîne de longueur fixe
```

### Types de date/heure

```sql
Date                          -- Date (YYYY-MM-DD)
DateTime                      -- Date et heure
DateTime64                    -- Date et heure avec précision
```

### Types spéciaux

```sql
Array(T)                      -- Tableau de type T
Tuple(T1, T2, ...)            -- Tuple
Nullable(T)                   -- Type nullable
```

## Création de tables

### Table MergeTree (recommandé)

```sql
CREATE TABLE events
(
    id UInt64,
    event_date Date,
    event_time DateTime,
    user_id UInt32,
    event_type String,
    value Float64
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, user_id)
SETTINGS index_granularity = 8192;
```

### Table Memory

```sql
CREATE TABLE temp_data
(
    id UInt64,
    name String
)
ENGINE = Memory;
```

### Table Log

```sql
CREATE TABLE logs
(
    id UInt64,
    message String
)
ENGINE = Log;
```

## Partitionnement

### Par date (recommandé)

```sql
PARTITION BY toYYYYMM(event_date)
```

### Par hash

```sql
PARTITION BY intHash32(user_id) % 10
```

### Par valeur

```sql
PARTITION BY event_type
```

## ORDER BY (clé de tri)

```sql
ORDER BY (event_date, user_id, event_type)
```

**Important :** L'ordre définit l'index primaire

## Moteurs de tables

### MergeTree

- **Usage** : Données persistantes, gros volumes
- **Avantages** : Compression, index, partitions
- **Inconvénients** : Plus complexe

### Memory

- **Usage** : Données temporaires
- **Avantages** : Très rapide
- **Inconvénients** : Perdues au redémarrage

### Log

- **Usage** : Petits volumes, logs
- **Avantages** : Simple
- **Inconvénients** : Pas de partitions

## Exemples complets

### Table de ventes

```sql
CREATE TABLE sales
(
    id UInt64,
    sale_date Date,
    product_id UInt32,
    customer_id UInt32,
    quantity UInt32,
    price Decimal64(2),
    total Decimal64(2) MATERIALIZED quantity * price
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(sale_date)
ORDER BY (sale_date, product_id);
```

### Table avec Array

```sql
CREATE TABLE user_tags
(
    user_id UInt32,
    tags Array(String),
    created_at DateTime
)
ENGINE = MergeTree()
ORDER BY user_id;
```

---

**Prochaine étape :** [Performance et Optimisation](./04-performance/README.md)

