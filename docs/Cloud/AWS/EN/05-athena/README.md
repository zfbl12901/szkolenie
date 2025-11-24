# 5. Amazon Athena - SQL Queries on S3

## 🎯 Objectives

- Understand Amazon Athena and its usage
- Create external tables pointing to S3
- Execute SQL queries on S3 files
- Optimize costs and performance
- Integrate with Glue Data Catalog

## 📋 Table of Contents

1. [Introduction to Athena](#introduction-to-athena)
2. [Create External Tables](#create-external-tables)
3. [Execute Queries](#execute-queries)
4. [Cost Optimization](#cost-optimization)
5. [Integration with Glue](#integration-with-glue)
6. [Best Practices](#best-practices)

---

## Introduction to Athena

### What is Amazon Athena?

**Amazon Athena** = Serverless SQL query service on S3

- **Serverless** : No infrastructure to manage
- **Pay-per-query** : Pay only what you use
- **Standard SQL** : Standard SQL syntax
- **Directly on S3** : No need to load into database

### Use Cases for Data Analyst

- **Data exploration** : Quickly analyze S3 files
- **Data Lake queries** : Queries on data lake
- **Ad-hoc analysis** : One-off analyses
- **Log analysis** : Analyze logs stored in S3

### Athena Free Tier

**Free forever:**
- 10 GB of data scanned/month
- Beyond: $5 per Terabyte scanned

**⚠️ Important:** Costs depend on the amount of data scanned. Optimize queries to reduce costs.

---

## Create External Tables

### Method 1: Via Athena Editor

**Step 1: Access Athena**

1. AWS Console → Search "Athena"
2. Click on "Amazon Athena"
3. First use: Configure S3 result

**Step 2: Configure Result**

1. "Settings" → "Manage"
2. "Query result location": `s3://my-bucket/athena-results/`
3. "Save"

**Step 3: Create a Table**

```sql
-- Table for CSV files
CREATE EXTERNAL TABLE users (
    id INT,
    name STRING,
    email STRING,
    created_at TIMESTAMP
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
    'serialization.format' = ',',
    'field.delim' = ','
)
STORED AS TEXTFILE
LOCATION 's3://my-bucket/data/users/'
TBLPROPERTIES ('skip.header.line.count'='1');
```

### Method 2: Via Glue Data Catalog (Recommended)

**Use tables created by Glue:**

1. Glue → Create a crawler for S3
2. Crawler automatically creates the table
3. Athena uses this table directly

**Advantages:**
- Automatically detected schema
- No need to define manually
- Reusable by other services

### Supported Formats

**CSV:**
```sql
CREATE EXTERNAL TABLE csv_data (
    col1 STRING,
    col2 INT
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS TEXTFILE
LOCATION 's3://bucket/csv/';
```

**JSON:**
```sql
CREATE EXTERNAL TABLE json_data (
    id INT,
    name STRING
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION 's3://bucket/json/';
```

**Parquet (recommended):**
```sql
CREATE EXTERNAL TABLE parquet_data (
    id INT,
    name STRING,
    created_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://bucket/parquet/';
```

---

## Execute Queries

### Basic Queries

**Simple SELECT:**

```sql
SELECT * FROM users LIMIT 10;
```

**Filter:**

```sql
SELECT 
    id,
    name,
    email
FROM users
WHERE created_at > DATE '2024-01-01'
ORDER BY created_at DESC;
```

**Aggregations:**

```sql
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS user_count,
    COUNT(DISTINCT email) AS unique_emails
FROM users
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;
```

### Advanced Queries

**Window functions:**

```sql
SELECT 
    id,
    name,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY DATE_TRUNC('month', created_at) ORDER BY created_at) AS rank
FROM users;
```

**Joins:**

```sql
SELECT 
    u.name,
    o.amount,
    o.created_at
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.created_at > DATE '2024-01-01';
```

### Queries on Partitions

**If data is partitioned:**

```sql
-- Table partitioned by date
CREATE EXTERNAL TABLE sales (
    id INT,
    product_id INT,
    amount DECIMAL(10,2)
)
PARTITIONED BY (sale_date DATE)
STORED AS PARQUET
LOCATION 's3://bucket/sales/';

-- Add partitions
ALTER TABLE sales ADD PARTITION (sale_date='2024-01-01')
LOCATION 's3://bucket/sales/year=2024/month=01/day=01/';

-- Query with partition (faster and cheaper)
SELECT * FROM sales
WHERE sale_date = DATE '2024-01-01';
```

---

## Cost Optimization

### Reduce Data Scanned

**1. Use WHERE to filter early:**

```sql
-- ❌ Bad: Scans everything then filters
SELECT * FROM large_table
WHERE date = '2024-01-01';

-- ✅ Good: Filters from the start (if partitioned)
SELECT * FROM large_table
WHERE date = '2024-01-01';
```

**2. Select only necessary columns:**

```sql
-- ❌ Bad: Scans all columns
SELECT * FROM large_table;

-- ✅ Good: Scans only necessary columns
SELECT id, name FROM large_table;
```

**3. Use LIMIT:**

```sql
-- Limit the number of results
SELECT * FROM large_table LIMIT 100;
```

### Use Parquet

**Parquet is more efficient than CSV:**

- **Compression** : Less data scanned
- **Columns** : Scans only necessary columns
- **Reduced cost** : Up to 90% reduction

**Convert CSV → Parquet with Glue:**

```python
# Glue job to convert
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "csv_data"
)

glueContext.write_dynamic_frame.from_options(
    frame = datasource,
    connection_type = "s3",
    connection_options = {"path": "s3://bucket/parquet/"},
    format = "parquet"
)
```

### Partition Data

**Partition by date (recommended):**

```
s3://bucket/data/
├── year=2024/
│   ├── month=01/
│   │   ├── day=01/
│   │   └── day=02/
│   └── month=02/
```

**Create partitioned table:**

```sql
CREATE EXTERNAL TABLE partitioned_data (
    id INT,
    name STRING
)
PARTITIONED BY (year INT, month INT, day INT)
STORED AS PARQUET
LOCATION 's3://bucket/data/';
```

---

## Integration with Glue

### Use Glue Tables

**Tables created by Glue are automatically available in Athena:**

1. Glue → Crawler creates a table
2. Athena → "Tables" → See all Glue tables
3. Use directly in queries

**Advantages:**
- Automatic schema
- No manual definition
- Automatic synchronization

### Update Partitions

**If new data added:**

```sql
-- Update partitions
MSCK REPAIR TABLE sales;

-- Or add manually
ALTER TABLE sales ADD PARTITION (sale_date='2024-01-02')
LOCATION 's3://bucket/sales/year=2024/month=01/day=02/';
```

---

## Best Practices

### Performance

1. **Use Parquet** instead of CSV
2. **Partition data** by date/category
3. **Select only necessary columns**
4. **Filter early** with WHERE
5. **Use LIMIT** for exploration

### Costs

1. **Monitor scanned data** in results
2. **Optimize queries** to reduce scan
3. **Use Parquet** for compression
4. **Partition** to reduce scan
5. **Cache** frequent results

### Organization

1. **Organize S3** with consistent prefixes
2. **Name tables** clearly
3. **Document schemas**
4. **Use databases** to organize

---

## Practical Examples

### Example 1: Analyze Logs

```sql
-- Table for logs
CREATE EXTERNAL TABLE logs (
    timestamp TIMESTAMP,
    level STRING,
    message STRING,
    user_id INT
)
PARTITIONED BY (date DATE)
STORED AS TEXTFILE
LOCATION 's3://bucket/logs/';

-- Query: Errors per day
SELECT 
    date,
    COUNT(*) AS error_count
FROM logs
WHERE level = 'ERROR'
GROUP BY date
ORDER BY date DESC;
```

### Example 2: Analyze CSV Data

```sql
-- CSV table
CREATE EXTERNAL TABLE sales_csv (
    id INT,
    product_id INT,
    amount DECIMAL(10,2),
    sale_date DATE
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS TEXTFILE
LOCATION 's3://bucket/sales/csv/'
TBLPROPERTIES ('skip.header.line.count'='1');

-- Analysis: Sales per month
SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total_sales,
    COUNT(*) AS transaction_count
FROM sales_csv
GROUP BY DATE_TRUNC('month', sale_date)
ORDER BY month;
```

### Example 3: Join Multiple Tables

```sql
-- Analyze with joins
SELECT 
    p.name AS product_name,
    c.name AS category_name,
    SUM(s.amount) AS total_sales
FROM sales s
JOIN products p ON s.product_id = p.id
JOIN categories c ON p.category_id = c.id
WHERE s.sale_date >= DATE '2024-01-01'
GROUP BY p.name, c.name
ORDER BY total_sales DESC
LIMIT 10;
```

---

## 📊 Key Takeaways

1. **Athena = Serverless SQL** on S3 files
2. **Free Tier: 10 GB/month** of scanned data
3. **Parquet** = most efficient format
4. **Partition** = reduce costs
5. **Glue integration** = automatic schemas

## 🔗 Next Module

Proceed to module [6. AWS Lambda - Serverless Computing](../06-lambda/README.md) to learn how to automate data processing.

