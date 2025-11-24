# 4. Amazon Redshift - Data Warehouse

## 🎯 Objectives

- Understand Amazon Redshift and its role
- Create a Redshift cluster (free for 2 months)
- Load data into Redshift
- Optimize Redshift queries
- Integrate with S3 and other services

## 📋 Table of Contents

1. [Introduction to Redshift](#introduction-to-redshift)
2. [Create a Redshift Cluster](#create-a-redshift-cluster)
3. [Load Data](#load-data)
4. [Advanced SQL Queries](#advanced-sql-queries)
5. [Optimization](#optimization)
6. [Integration with Other Services](#integration-with-other-services)

---

## Introduction to Redshift

### What is Amazon Redshift?

**Amazon Redshift** = Managed cloud data warehouse

- **OLAP** : Optimized for analysis (not transactions)
- **Columnar** : Column-oriented storage
- **Massively parallel** : Distributed processing
- **Scalable** : From a few GB to several PB

### Use Cases for Data Analyst

- **Data Warehouse** : Centralize data
- **Analytics** : Complex queries on large volumes
- **Business Intelligence** : Dashboards and reports
- **Data Mining** : In-depth analysis

### Redshift Free Tier

**Free for 2 months:**
- 750 hours/month of `dc2.large` cluster
- 32 GB storage per node
- After 2 months: normal billing

**⚠️ Important:** Stop the cluster when not in use to avoid costs.

---

## Create a Redshift Cluster

### Step 1: Access Redshift

1. AWS Console → Search "Redshift"
2. Click on "Amazon Redshift"
3. "Create cluster"

### Step 2: Cluster Configuration

**Basic Configuration:**

1. **Cluster identifier**: `data-analyst-cluster`
2. **Node type**: `dc2.large` (free for 2 months)
3. **Number of nodes**: 1 (sufficient to start)
4. **Database name**: `analytics` (default: `dev`)
5. **Database port**: 5439 (default)
6. **Master username**: `admin` (or other)
7. **Master password**: Strong password

**Network Configuration:**

1. **VPC**: Choose an existing VPC
2. **Subnet group**: Create or use existing
3. **Publicly accessible**: ✅ Yes (for easy access)
4. **Availability zone**: Choose a zone

**Security:**

1. **VPC security groups**: Create a security group
   - Allow port 5439 from your IP
2. **Encryption**: Enable (recommended)

### Step 3: Create the Cluster

1. Click on "Create cluster"
2. Wait 5-10 minutes (creation)
3. Cluster ready when status = "Available"

**⚠️ Important:** Note the cluster endpoint (e.g., `data-analyst-cluster.xxxxx.eu-west-3.redshift.amazonaws.com:5439`)

---

## Load Data

### Method 1: COPY from S3 (Recommended)

**Fastest for large quantities:**

```sql
-- Create a table
CREATE TABLE users (
    id INTEGER,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP
);

-- Load from S3
COPY users
FROM 's3://my-bucket/data/users.csv'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
IGNOREHEADER 1;
```

**IAM Role Configuration:**

1. IAM → "Roles" → "Create role"
2. Type: "Redshift"
3. Attach policy: `AmazonS3ReadOnlyAccess`
4. Name: `RedshiftS3Role`
5. Copy the ARN for COPY

### Method 2: INSERT (Small Quantities)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Method 3: INSERT from Query

```sql
INSERT INTO users_aggregated
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS user_count
FROM users
GROUP BY DATE_TRUNC('month', created_at);
```

### Supported Formats

- **CSV**: CSV files
- **JSON**: JSON files
- **Parquet**: Optimized format (recommended)
- **Avro**: Avro format

---

## Advanced SQL Queries

### Analytical Functions

**Window functions:**

```sql
-- ROW_NUMBER
SELECT 
    id,
    name,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY created_at) AS rank
FROM products;

-- LAG/LEAD
SELECT 
    date,
    sales,
    LAG(sales, 1) OVER (ORDER BY date) AS previous_sales,
    LEAD(sales, 1) OVER (ORDER BY date) AS next_sales
FROM daily_sales;

-- RANK
SELECT 
    user_id,
    total_spent,
    RANK() OVER (ORDER BY total_spent DESC) AS spending_rank
FROM user_totals;
```

### Complex Aggregations

```sql
-- GROUP BY with ROLLUP
SELECT 
    category,
    region,
    SUM(amount) AS total
FROM sales
GROUP BY ROLLUP(category, region);

-- GROUP BY with CUBE
SELECT 
    category,
    region,
    SUM(amount) AS total
FROM sales
GROUP BY CUBE(category, region);
```

### Optimized Joins

```sql
-- Join with distribution key
SELECT 
    u.name,
    o.amount,
    o.created_at
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';
```

---

## Optimization

### Distribution Keys

**Choose the right distribution key:**

```sql
-- Key distribution (for joins)
CREATE TABLE users (
    id INTEGER DISTKEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- ALL distribution (for small tables)
CREATE TABLE categories (
    id INTEGER,
    name VARCHAR(100)
) DISTSTYLE ALL;

-- EVEN distribution (default)
CREATE TABLE logs (
    id INTEGER,
    message TEXT
) DISTSTYLE EVEN;
```

### Sort Keys

**Improve query performance:**

```sql
-- Simple sort key
CREATE TABLE orders (
    id INTEGER,
    user_id INTEGER,
    created_at TIMESTAMP,
    amount DECIMAL(10,2)
) SORTKEY (created_at);

-- Composite sort key
CREATE TABLE sales (
    date DATE,
    region VARCHAR(50),
    amount DECIMAL(10,2)
) SORTKEY (date, region);
```

### Compression

**Reduce storage space:**

```sql
-- Automatic compression
CREATE TABLE users (
    id INTEGER,
    name VARCHAR(100) ENCODE lzo,
    email VARCHAR(100) ENCODE lzo,
    created_at TIMESTAMP ENCODE delta
);
```

### ANALYZE

**Update statistics:**

```sql
-- Analyze a table
ANALYZE users;

-- Analyze all tables
ANALYZE;
```

---

## Integration with Other Services

### Redshift + S3

**Unload to S3:**

```sql
UNLOAD ('SELECT * FROM users WHERE created_at > ''2024-01-01''')
TO 's3://my-bucket/exports/users/'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
PARALLEL OFF;
```

### Redshift + Glue

**Glue can load into Redshift:**

```python
# In a Glue job
glueContext.write_dynamic_frame.from_jdbc_conf(
    frame = transformed_data,
    catalog_connection = "redshift-connection",
    connection_options = {
        "dbtable": "users",
        "database": "analytics"
    }
)
```

### Redshift + QuickSight

**Connect QuickSight to Redshift:**

1. QuickSight → "Data sources"
2. "Redshift"
3. Enter connection information
4. Select tables
5. Create visualizations

---

## Best Practices

### Performance

1. **Use COPY** instead of INSERT for large quantities
2. **Choose the right distribution keys**
3. **Use sort keys** for frequent queries
4. **Compress columns** to save space
5. **VACUUM regularly** to optimize

### Costs

1. **Stop the cluster** when not in use
2. **Use the right node type** according to needs
3. **Monitor storage** usage
4. **Clean up** unnecessary data

### Security

1. **Encrypt data** in transit and at rest
2. **Use VPC** to isolate the cluster
3. **Limit access** with security groups
4. **Audit access** with CloudTrail

---

## Practical Examples

### Example 1: Complete S3 → Redshift Pipeline

```sql
-- 1. Create the table
CREATE TABLE sales (
    id INTEGER,
    product_id INTEGER,
    amount DECIMAL(10,2),
    sale_date DATE
) DISTKEY(product_id) SORTKEY(sale_date);

-- 2. Load from S3
COPY sales
FROM 's3://my-bucket/data/sales/'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV
IGNOREHEADER 1;

-- 3. Analyze
ANALYZE sales;

-- 4. Analytical queries
SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total_sales
FROM sales
GROUP BY DATE_TRUNC('month', sale_date)
ORDER BY month;
```

### Example 2: Aggregations with Window Functions

```sql
-- Top 10 products per month
SELECT 
    product_id,
    month,
    total_sales,
    RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) AS rank
FROM (
    SELECT 
        product_id,
        DATE_TRUNC('month', sale_date) AS month,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY product_id, DATE_TRUNC('month', sale_date)
) monthly_sales
WHERE RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) <= 10;
```

---

## 📊 Key Takeaways

1. **Redshift = Data warehouse** for analytics
2. **Free Tier: 2 months** free (750 hours)
3. **COPY from S3** = fastest method
4. **Distribution and sort keys** = performance keys
5. **Stop the cluster** when not in use

## 🔗 Next Module

Proceed to module [5. Amazon Athena - SQL Queries on S3](../05-athena/README.md) to learn how to query S3 files directly.

