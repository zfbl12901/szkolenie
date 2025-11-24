# 2. SQL Analytique avec ClickHouse

## 🎯 Objectifs

- Maîtriser les requêtes SELECT avancées
- Utiliser les agrégations et GROUP BY
- Comprendre les fonctions analytiques
- Utiliser les fenêtres (Window Functions)

## Agrégations avancées

### GROUP BY avec plusieurs colonnes

```sql
SELECT 
    event_date,
    event_type,
    COUNT(*) as count,
    SUM(value) as total_value,
    AVG(value) as avg_value
FROM events
GROUP BY event_date, event_type
ORDER BY event_date, event_type;
```

### HAVING pour filtrer les agrégations

```sql
SELECT 
    event_type,
    COUNT(*) as count
FROM events
GROUP BY event_type
HAVING count > 10;
```

## Fonctions d'agrégation

### Fonctions de base

```sql
SELECT 
    COUNT(*) as total_rows,
    COUNT(DISTINCT user_id) as unique_users,
    SUM(value) as total_value,
    AVG(value) as avg_value,
    MIN(value) as min_value,
    MAX(value) as max_value
FROM events;
```

### Fonctions statistiques

```sql
SELECT 
    event_type,
    quantile(0.5)(value) as median,
    quantile(0.95)(value) as p95,
    stddevPop(value) as std_dev
FROM events
GROUP BY event_type;
```

## Window Functions

### ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT 
    user_id,
    event_date,
    value,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY value DESC) as rank
FROM events;
```

### LAG et LEAD

```sql
SELECT 
    event_date,
    value,
    LAG(value) OVER (ORDER BY event_date) as prev_value,
    LEAD(value) OVER (ORDER BY event_date) as next_value
FROM events
ORDER BY event_date;
```

### SUM OVER (cumulative)

```sql
SELECT 
    event_date,
    value,
    SUM(value) OVER (ORDER BY event_date) as cumulative_sum
FROM events
ORDER BY event_date;
```

## Requêtes complexes

### Sous-requêtes

```sql
SELECT 
    event_type,
    COUNT(*) as count
FROM events
WHERE user_id IN (
    SELECT DISTINCT user_id 
    FROM events 
    WHERE value > 100
)
GROUP BY event_type;
```

### JOIN

```sql
SELECT 
    e.event_type,
    u.user_name,
    COUNT(*) as count
FROM events e
JOIN users u ON e.user_id = u.id
GROUP BY e.event_type, u.user_name;
```

### UNION

```sql
SELECT event_type, COUNT(*) as count FROM events GROUP BY event_type
UNION ALL
SELECT 'total' as event_type, COUNT(*) as count FROM events;
```

## Fonctions de date

```sql
SELECT 
    toStartOfDay(event_time) as day,
    toStartOfHour(event_time) as hour,
    toStartOfWeek(event_time) as week,
    toStartOfMonth(event_time) as month,
    COUNT(*) as count
FROM events
GROUP BY day, hour, week, month;
```

## Exercices pratiques

### Exercice 1 : Top 10 des utilisateurs

```sql
SELECT 
    user_id,
    COUNT(*) as event_count,
    SUM(value) as total_value
FROM events
GROUP BY user_id
ORDER BY total_value DESC
LIMIT 10;
```

### Exercice 2 : Évolution quotidienne

```sql
SELECT 
    event_date,
    COUNT(*) as daily_count,
    SUM(value) as daily_total,
    AVG(value) as daily_avg
FROM events
GROUP BY event_date
ORDER BY event_date;
```

---

**Prochaine étape :** [Types de données et Tables](./03-types-tables/README.md)

