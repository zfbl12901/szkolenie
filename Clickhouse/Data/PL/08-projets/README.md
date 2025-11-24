# 8. Projekty praktyczne

## 🎯 Cele

- Zastosować zdobytą wiedzę
- Tworzyć projekty do portfolio
- Rozwiązywać rzeczywiste problemy
- Optymalizować wydajność

## Projekt 1 : Analityka webowa

### Cel

Analizować logi strony internetowej, aby zrozumieć zachowanie użytkowników.

### Dane

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

### Zapytania analityczne

```sql
-- Najczęściej odwiedzane strony
SELECT 
    page,
    COUNT(*) as visits,
    AVG(duration) as avg_duration
FROM web_logs
GROUP BY page
ORDER BY visits DESC
LIMIT 10;

-- Aktywni użytkownicy dziennie
SELECT 
    toStartOfDay(timestamp) as day,
    COUNT(DISTINCT user_id) as active_users
FROM web_logs
GROUP BY day
ORDER BY day;
```

## Projekt 2 : Hurtownia danych analityczna

### Cel

Utworzyć hurtownię danych do analizy sprzedaży.

### Schemat

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

### Analizy

```sql
-- Sprzedaż według miesiąca
SELECT 
    toStartOfMonth(sale_date) as month,
    SUM(total) as monthly_revenue,
    COUNT(*) as sales_count
FROM sales
GROUP BY month
ORDER BY month;

-- Top produkty
SELECT 
    product_id,
    SUM(quantity) as total_sold,
    SUM(total) as revenue
FROM sales
GROUP BY product_id
ORDER BY revenue DESC
LIMIT 10;
```

## Projekt 3 : Dashboard czasu rzeczywistego

### Cel

Tworzyć metryki czasu rzeczywistego dla dashboardu.

### Tabela zdarzeń

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

### Metryki czasu rzeczywistego

```sql
-- Zdarzenia z ostatnich 24h
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

## Wskazówki do portfolio

1. **Dokumentuj projekty** : Wyjaśnij problem i rozwiązanie
2. **Pokaż wyniki** : Wizualizacje, metryki
3. **Czysty kod** : Zoptymalizowane i skomentowane zapytania
4. **Wydajność** : Pokaż wykonane optymalizacje
5. **GitHub** : Udostępnij projekty na GitHub

---

**Gratulacje! Ukończyłeś szkolenie ClickHouse dla Data Analyst! 🎉**

