# 1. Prise en main ClickHouse

## 🎯 Objectifs

- Comprendre ClickHouse et l'OLAP
- Installer ClickHouse
- Comprendre les concepts de base
- Utiliser ClickHouse Client
- Premières requêtes

## Introduction à ClickHouse

**ClickHouse** = Base de données OLAP orientée colonnes

- **OLAP** : Online Analytical Processing
- **Orientée colonnes** : Stockage par colonnes
- **Haute performance** : Très rapide sur gros volumes
- **Open-source** : Gratuit et open-source

### Pourquoi ClickHouse pour Data Analyst ?

- **Gros volumes** : Gère des milliards de lignes
- **Performance** : Requêtes analytiques ultra-rapides
- **Compression** : Compression efficace
- **SQL** : Langage SQL standard

### Cas d'usage typiques

- Analytics web (logs, événements)
- Data warehousing
- Business Intelligence
- IoT (capteurs)

## Installation

### Option 1 : ClickHouse Playground (recommandé)

1. Allez sur [ClickHouse Playground](https://play.clickhouse.com/)
2. Aucune installation nécessaire

### Option 2 : Docker

```bash
docker run -d --name clickhouse-server -p 8123:8123 -p 9000:9000 clickhouse/clickhouse-server
```

### Option 3 : Installation locale (Linux)

```bash
sudo apt-get install -y apt-transport-https ca-certificates
echo "deb https://repo.clickhouse.com/deb stable main" | sudo tee /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update
sudo apt-get install -y clickhouse-server clickhouse-client
sudo service clickhouse-server start
```

## Concepts de base

### Stockage par colonnes

**Avantages :**
- Compression efficace
- Requêtes analytiques rapides
- Agrégations optimisées

### Moteurs de tables

- **MergeTree** : Pour données persistantes
- **Memory** : Pour données en mémoire
- **Log** : Pour petits volumes

## ClickHouse Client

```bash
clickhouse-client
clickhouse-client --host localhost --port 9000
```

### Commandes de base

```sql
SHOW DATABASES;
USE default;
SHOW TABLES;
DESCRIBE table_name;
```

## Premières requêtes

### Créer une base de données

```sql
CREATE DATABASE IF NOT EXISTS analytics;
USE analytics;
```

### Créer une table

```sql
CREATE TABLE IF NOT EXISTS events
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
ORDER BY (event_date, event_time);
```

### Insérer des données

```sql
INSERT INTO events VALUES
(1, '2024-01-15', '2024-01-15 10:00:00', 100, 'click', 1.5),
(2, '2024-01-15', '2024-01-15 10:01:00', 101, 'view', 2.0);
```

### Requête SELECT

```sql
SELECT * FROM events;

SELECT 
    event_type,
    COUNT(*) as count,
    SUM(value) as total_value
FROM events
WHERE event_date = '2024-01-15'
GROUP BY event_type;
```

---

**Prochaine étape :** [SQL Analytique avec ClickHouse](./02-sql-analytique/README.md)

