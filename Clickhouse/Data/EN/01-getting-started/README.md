# 1. Getting Started with ClickHouse

## 🎯 Objectives

- Understand ClickHouse and OLAP
- Install ClickHouse
- Understand basic concepts
- Use ClickHouse Client
- First queries

## Introduction to ClickHouse

**ClickHouse** = Column-oriented OLAP database

- **OLAP** : Online Analytical Processing
- **Column-oriented** : Column storage
- **High performance** : Very fast on large volumes
- **Open-source** : Free and open-source

### Why ClickHouse for Data Analyst?

- **Large volumes** : Handles billions of rows
- **Performance** : Ultra-fast analytical queries
- **Compression** : Efficient compression
- **SQL** : Standard SQL language

### Typical Use Cases

- Web analytics (logs, events)
- Data warehousing
- Business Intelligence
- IoT (sensors)

## Installation

### Option 1 : ClickHouse Playground (recommended)

1. Go to [ClickHouse Playground](https://play.clickhouse.com/)
2. No installation needed

### Option 2 : Docker

```bash
docker run -d --name clickhouse-server -p 8123:8123 -p 9000:9000 clickhouse/clickhouse-server
```

### Option 3 : Local Installation (Linux)

```bash
sudo apt-get install -y apt-transport-https ca-certificates
echo "deb https://repo.clickhouse.com/deb stable main" | sudo tee /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update
sudo apt-get install -y clickhouse-server clickhouse-client
sudo service clickhouse-server start
```

## Basic Concepts

### Column Storage

**Advantages :**
- Efficient compression
- Fast analytical queries
- Optimized aggregations

### Table Engines

- **MergeTree** : For persistent data
- **Memory** : For in-memory data
- **Log** : For small volumes

## ClickHouse Client

```bash
clickhouse-client
clickhouse-client --host localhost --port 9000
```

### Basic Commands

```sql
SHOW DATABASES;
USE default;
SHOW TABLES;
DESCRIBE table_name;
```

## First Queries

### Create Database

```sql
CREATE DATABASE IF NOT EXISTS analytics;
USE analytics;
```

### Create Table

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

### Insert Data

```sql
INSERT INTO events VALUES
(1, '2024-01-15', '2024-01-15 10:00:00', 100, 'click', 1.5),
(2, '2024-01-15', '2024-01-15 10:01:00', 101, 'view', 2.0);
```

### SELECT Query

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

**Next step :** [Analytical SQL with ClickHouse](./02-sql-analytique/README.md)

