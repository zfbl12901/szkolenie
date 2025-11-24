# 5. Zaawansowane funkcje

## 🎯 Cele

- Opanować funkcje daty/czasu
- Używać funkcji matematycznych
- Manipulować łańcuchami znaków
- Używać zaawansowanych funkcji agregacji

## Funkcje daty i czasu

### Ekstrakcja

```sql
SELECT 
    toYear(event_time) as year,
    toMonth(event_time) as month,
    toDayOfMonth(event_time) as day,
    toHour(event_time) as hour;
```

### Agregacja czasowa

```sql
SELECT 
    toStartOfDay(event_time) as day,
    toStartOfHour(event_time) as hour,
    toStartOfWeek(event_time) as week,
    toStartOfMonth(event_time) as month;
```

## Funkcje matematyczne

### Zaokrąglanie

```sql
SELECT 
    round(value, 2) as rounded,
    floor(value) as floor_val,
    ceil(value) as ceil_val;
```

### Statystyki

```sql
SELECT 
    quantile(0.5)(value) as median,
    quantile(0.95)(value) as p95,
    stddevPop(value) as std_dev;
```

## Funkcje łańcuchowe

### Manipulacja

```sql
SELECT 
    length(name) as name_length,
    upper(name) as upper_name,
    lower(name) as lower_name;
```

### Wyszukiwanie

```sql
SELECT 
    position(name, 'test') as pos,
    match(name, 'test.*') as matches,
    replace(name, 'old', 'new') as replaced;
```

## Zaawansowane funkcje agregacji

### Kwantyle

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

---

**Następny krok :** [Integracja i ETL](./06-integration-etl/README.md)

