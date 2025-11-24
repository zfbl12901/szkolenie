# 5. Fonctions Avancées

## 🎯 Objectifs

- Maîtriser les fonctions de date/heure
- Utiliser les fonctions mathématiques
- Manipuler les chaînes de caractères
- Utiliser les fonctions d'agrégation avancées

## Fonctions de date et heure

### Extraction

```sql
SELECT 
    toYear(event_time) as year,
    toMonth(event_time) as month,
    toDayOfMonth(event_time) as day,
    toHour(event_time) as hour,
    toMinute(event_time) as minute;
```

### Agrégation temporelle

```sql
SELECT 
    toStartOfDay(event_time) as day,
    toStartOfHour(event_time) as hour,
    toStartOfWeek(event_time) as week,
    toStartOfMonth(event_time) as month,
    toStartOfQuarter(event_time) as quarter,
    toStartOfYear(event_time) as year;
```

### Calculs de dates

```sql
SELECT 
    event_date,
    addDays(event_date, 7) as next_week,
    addMonths(event_date, 1) as next_month,
    dateDiff('day', event_date, today()) as days_ago;
```

## Fonctions mathématiques

### Arrondis

```sql
SELECT 
    round(value, 2) as rounded,
    floor(value) as floor_val,
    ceil(value) as ceil_val,
    trunc(value, 2) as truncated;
```

### Statistiques

```sql
SELECT 
    quantile(0.5)(value) as median,
    quantile(0.95)(value) as p95,
    quantile(0.99)(value) as p99,
    stddevPop(value) as std_dev,
    varPop(value) as variance;
```

### Logarithmes

```sql
SELECT 
    log(value) as ln,
    log2(value) as log2,
    log10(value) as log10,
    exp(value) as exp_val;
```

## Fonctions de chaînes

### Manipulation

```sql
SELECT 
    length(name) as name_length,
    upper(name) as upper_name,
    lower(name) as lower_name,
    reverse(name) as reversed;
```

### Extraction

```sql
SELECT 
    substring(name, 1, 5) as first_5,
    left(name, 3) as left_3,
    right(name, 3) as right_3;
```

### Recherche

```sql
SELECT 
    position(name, 'test') as pos,
    match(name, 'test.*') as matches,
    replace(name, 'old', 'new') as replaced;
```

## Fonctions d'agrégation avancées

### Quantiles

```sql
SELECT 
    quantile(0.5)(value) as median,
    quantiles(0.5, 0.9, 0.99)(value) as quantiles;
```

### TopK

```sql
SELECT 
    topK(10)(event_type) as top_10_types,
    topKWeighted(10)(event_type, value) as top_10_weighted;
```

### ArgMax/ArgMin

```sql
SELECT 
    argMax(user_id, value) as user_with_max_value,
    argMin(user_id, value) as user_with_min_value;
```

## Fonctions conditionnelles

### IF

```sql
SELECT 
    IF(value > 100, 'high', 'low') as category;
```

### CASE

```sql
SELECT 
    CASE 
        WHEN value > 100 THEN 'high'
        WHEN value > 50 THEN 'medium'
        ELSE 'low'
    END as category;
```

### multiIf

```sql
SELECT 
    multiIf(
        value > 100, 'high',
        value > 50, 'medium',
        'low'
    ) as category;
```

---

**Prochaine étape :** [Intégration et ETL](./06-integration-etl/README.md)

