# 8. Practical Projects

## 🎯 Objectives

- Apply acquired knowledge
- Create projects for your portfolio
- Solve real problems
- Optimize performance

## Project 1 : Web Analytics

### Objective

Analyze website logs to understand user behavior.

### Data

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

### Analysis Queries

```sql
-- Most visited pages
SELECT 
    page,
    COUNT(*) as visits,
    AVG(duration) as avg_duration
FROM web_logs
GROUP BY page
ORDER BY visits DESC
LIMIT 10;

-- Active users per day
SELECT 
    toStartOfDay(timestamp) as day,
    COUNT(DISTINCT user_id) as active_users
FROM web_logs
GROUP BY day
ORDER BY day;
```

## Project 2 : Analytical Data Warehouse

### Objective

Create a data warehouse for sales analysis.

### Schema

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
-- Sales by month
SELECT 
    toStartOfMonth(sale_date) as month,
    SUM(total) as monthly_revenue,
    COUNT(*) as sales_count
FROM sales
GROUP BY month
ORDER BY month;

-- Top products
SELECT 
    product_id,
    SUM(quantity) as total_sold,
    SUM(total) as revenue
FROM sales
GROUP BY product_id
ORDER BY revenue DESC
LIMIT 10;
```

## Project 3 : Real-Time Dashboard

### Objective

Create real-time metrics for a dashboard.

### Event Table

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

### Real-time Metrics

```sql
-- Events from last 24h
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

## Portfolio Tips

1. **Document your projects** : Explain problem and solution
2. **Show results** : Visualizations, metrics
3. **Clean code** : Optimized and commented queries
4. **Performance** : Show optimizations made
5. **GitHub** : Share projects on GitHub

---

**Congratulations! You have completed the ClickHouse training for Data Analyst! 🎉**

