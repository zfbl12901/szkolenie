# 8. Projets Pratiques

## 🎯 Objectifs

- Appliquer les connaissances acquises
- Créer des projets pour votre portfolio
- Résoudre des problèmes réels
- Optimiser les performances

## Projet 1 : Analytics Web

### Objectif

Analyser les logs d'un site web pour comprendre le comportement des utilisateurs.

### Données

```sql
CREATE TABLE web_logs
(
    timestamp DateTime,
    user_id UInt32,
    page String,
    action String,
    duration UInt32
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp, user_id);
```

### Requêtes d'analyse

```sql
-- Pages les plus visitées
SELECT 
    page,
    COUNT(*) as visits,
    AVG(duration) as avg_duration
FROM web_logs
GROUP BY page
ORDER BY visits DESC
LIMIT 10;

-- Utilisateurs actifs par jour
SELECT 
    toStartOfDay(timestamp) as day,
    COUNT(DISTINCT user_id) as active_users
FROM web_logs
GROUP BY day
ORDER BY day;
```

## Projet 2 : Data Warehouse Analytique

### Objectif

Créer un entrepôt de données pour l'analyse des ventes.

### Schéma

```sql
CREATE TABLE sales
(
    sale_id UInt64,
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

### Analyses

```sql
-- Ventes par mois
SELECT 
    toStartOfMonth(sale_date) as month,
    SUM(total) as monthly_revenue,
    COUNT(*) as sales_count
FROM sales
GROUP BY month
ORDER BY month;

-- Top produits
SELECT 
    product_id,
    SUM(quantity) as total_sold,
    SUM(total) as revenue
FROM sales
GROUP BY product_id
ORDER BY revenue DESC
LIMIT 10;
```

## Projet 3 : Dashboard Temps Réel

### Objectif

Créer des métriques en temps réel pour un dashboard.

### Table d'événements

```sql
CREATE TABLE realtime_events
(
    event_time DateTime,
    event_type String,
    value Float64
)
ENGINE = MergeTree()
ORDER BY event_time;
```

### Métriques temps réel

```sql
-- Événements des dernières 24h
SELECT 
    toStartOfHour(event_time) as hour,
    event_type,
    COUNT(*) as count,
    SUM(value) as total
FROM realtime_events
WHERE event_time >= now() - INTERVAL 24 HOUR
GROUP BY hour, event_type
ORDER BY hour DESC;
```

## Projet 4 : Analyse IoT

### Objectif

Analyser les données de capteurs IoT.

### Schéma

```sql
CREATE TABLE sensor_data
(
    timestamp DateTime,
    sensor_id UInt32,
    temperature Float32,
    humidity Float32,
    pressure Float32
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp, sensor_id);
```

### Analyses

```sql
-- Température moyenne par capteur
SELECT 
    sensor_id,
    AVG(temperature) as avg_temp,
    MIN(temperature) as min_temp,
    MAX(temperature) as max_temp
FROM sensor_data
GROUP BY sensor_id;

-- Détection d'anomalies
SELECT 
    sensor_id,
    timestamp,
    temperature
FROM sensor_data
WHERE temperature > (
    SELECT AVG(temperature) + 2 * stddevPop(temperature)
    FROM sensor_data
);
```

## Conseils pour votre portfolio

1. **Documentez vos projets** : Expliquez le problème et la solution
2. **Montrez les résultats** : Visualisations, métriques
3. **Code propre** : Requêtes optimisées et commentées
4. **Performance** : Montrez les optimisations effectuées
5. **GitHub** : Partagez vos projets sur GitHub

---

**Félicitations ! Vous avez terminé la formation ClickHouse pour Data Analyst ! 🎉**

