# 1. Rozpoczęcie pracy z ClickHouse

## 🎯 Cele

- Zrozumieć ClickHouse i OLAP
- Zainstalować ClickHouse
- Zrozumieć podstawowe koncepcje
- Używać ClickHouse Client
- Pierwsze zapytania

## Wprowadzenie do ClickHouse

**ClickHouse** = Baza danych OLAP zorientowana na kolumny

- **OLAP** : Online Analytical Processing
- **Zorientowana na kolumny** : Przechowywanie kolumnowe
- **Wysoka wydajność** : Bardzo szybka na dużych wolumenach
- **Open-source** : Darmowa i open-source

### Dlaczego ClickHouse dla Data Analyst?

- **Duże wolumeny** : Obsługuje miliardy wierszy
- **Wydajność** : Ultra-szybkie zapytania analityczne
- **Kompresja** : Efektywna kompresja
- **SQL** : Standardowy język SQL

### Typowe przypadki użycia

- Analityka webowa (logi, zdarzenia)
- Hurtownia danych
- Business Intelligence
- IoT (czujniki)

## Instalacja

### Opcja 1 : ClickHouse Playground (zalecane)

1. Przejdź do [ClickHouse Playground](https://play.clickhouse.com/)
2. Brak potrzeby instalacji

### Opcja 2 : Docker

```bash
docker run -d --name clickhouse-server -p 8123:8123 -p 9000:9000 clickhouse/clickhouse-server
```

### Opcja 3 : Instalacja lokalna (Linux)

```bash
sudo apt-get install -y apt-transport-https ca-certificates
echo "deb https://repo.clickhouse.com/deb stable main" | sudo tee /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update
sudo apt-get install -y clickhouse-server clickhouse-client
sudo service clickhouse-server start
```

## Podstawowe koncepcje

### Przechowywanie kolumnowe

**Zalety :**
- Efektywna kompresja
- Szybkie zapytania analityczne
- Zoptymalizowane agregacje

### Silniki tabel

- **MergeTree** : Do danych trwałych
- **Memory** : Do danych w pamięci
- **Log** : Do małych wolumenów

## ClickHouse Client

```bash
clickhouse-client
clickhouse-client --host localhost --port 9000
```

### Podstawowe polecenia

```sql
SHOW DATABASES;
USE default;
SHOW TABLES;
DESCRIBE table_name;
```

## Pierwsze zapytania

### Utworzenie bazy danych

```sql
CREATE DATABASE IF NOT EXISTS analytics;
USE analytics;
```

### Utworzenie tabeli

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

### Wstawienie danych

```sql
INSERT INTO events VALUES
(1, '2024-01-15', '2024-01-15 10:00:00', 100, 'click', 1.5),
(2, '2024-01-15', '2024-01-15 10:01:00', 101, 'view', 2.0);
```

### Zapytanie SELECT

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

**Następny krok :** [Analityczny SQL z ClickHouse](./02-sql-analytique/README.md)

