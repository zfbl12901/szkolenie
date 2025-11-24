# 5. Azure Synapse Analytics - Data Warehouse

## 🎯 Objectives

- Understand Azure Synapse Analytics
- Create a Synapse workspace
- Load data
- Execute advanced SQL queries
- Integrate with PowerBI

## 📋 Table of Contents

1. [Introduction to Synapse](#introduction-to-synapse)
2. [Create a Synapse Workspace](#create-a-synapse-workspace)
3. [Load Data](#load-data)
4. [Advanced SQL Queries](#advanced-sql-queries)
5. [Integration with PowerBI](#integration-with-powerbi)
6. [Best Practices](#best-practices)

---

## Introduction to Synapse

### What is Azure Synapse Analytics?

**Azure Synapse Analytics** = Unified analytics platform

- **Data Warehouse** : Data storage and analysis
- **Big Data** : Large-scale processing
- **SQL** : Standard SQL queries
- **Spark** : Distributed processing
- **Integration** : With all Azure services

### Synapse Components

1. **SQL Pool** : SQL data warehouse (formerly SQL Data Warehouse)
2. **Spark Pool** : Spark clusters for Big Data
3. **Synapse Studio** : Unified web interface
4. **Pipelines** : Integrated ETL
5. **Notebooks** : Python, SQL, Scala

### Synapse Free Tier

**Free with Azure credit:**
- Use the 200$ free credit (30 days)
- After: normal billing

**⚠️ Important:** Synapse can be expensive. Monitor costs carefully.

---

## Create a Synapse Workspace

### Step 1: Access Synapse

1. Azure Portal → Search "Azure Synapse Analytics"
2. Click on "Azure Synapse Analytics"
3. Click on "Create"

### Step 2: Basic Configuration

**Basic Information:**
- **Subscription** : Choose your subscription
- **Resource group** : Create or use existing
- **Workspace name** : `my-synapse-workspace`
- **Region** : Choose the region
- **Data Lake Storage Gen2** : Create new or use existing

**SQL Administrator:**
- **SQL admin name** : `sqladmin`
- **Password** : Strong password

### Step 3: SQL Pool Configuration

**SQL Pool:**
- **Create a SQL pool** : ✅ Yes (to start)
- **Performance level** : DW100c (cheapest)
- **Or** : Create later (Serverless SQL)

**⚠️ Important:** Serverless SQL = pay-per-query, more economical to start.

### Step 4: Create the Workspace

1. Click on "Review + create"
2. Verify configuration
3. Click on "Create"
4. Wait for creation (5-10 minutes)

**⚠️ Important:** Note SQL credentials.

---

## Load Data

### Method 1: COPY from Data Lake Storage

**Fastest for large quantities:**

```sql
-- Create a table
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at DATETIME2
)
WITH (
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
);

-- Load from Data Lake Storage
COPY INTO users
FROM 'https://mystorageaccount.dfs.core.windows.net/data-lake/raw/users.csv'
WITH (
    FILE_TYPE = 'CSV',
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
);
```

### Method 2: Via Synapse Pipelines

**Integrated pipeline:**

1. Synapse Studio → "Integrate" → "Pipelines"
2. Create a new pipeline
3. Add "Copy Data" activity
4. Source: Azure Blob Storage or Data Lake
5. Sink: SQL Pool
6. Execute the pipeline

### Method 3: INSERT (Small Quantities)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Method 4: Via PolyBase (External Tables)

**Create an external table:**

```sql
-- Create a credential
CREATE DATABASE SCOPED CREDENTIAL BlobCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
SECRET = 'your-sas-token';

-- Create an external data source
CREATE EXTERNAL DATA SOURCE BlobStorage
WITH (
    TYPE = HADOOP,
    LOCATION = 'wasbs://container@account.blob.core.windows.net',
    CREDENTIAL = BlobCredential
);

-- Create an external file format
CREATE EXTERNAL FILE FORMAT CSVFormat
WITH (
    FORMAT_TYPE = DELIMITEDTEXT,
    FORMAT_OPTIONS (FIELD_TERMINATOR = ',')
);

-- Create an external table
CREATE EXTERNAL TABLE users_external (
    id INT,
    name VARCHAR(100),
    email VARCHAR(100)
)
WITH (
    LOCATION = 'raw/users.csv',
    DATA_SOURCE = BlobStorage,
    FILE_FORMAT = CSVFormat
);

-- Load into internal table
INSERT INTO users
SELECT * FROM users_external;
```

---

## Advanced SQL Queries

### Basic Queries

**Simple SELECT:**

```sql
SELECT TOP 100 * FROM users;
```

**Aggregations:**

```sql
SELECT 
    YEAR(created_at) AS year,
    MONTH(created_at) AS month,
    COUNT(*) AS user_count
FROM users
GROUP BY YEAR(created_at), MONTH(created_at)
ORDER BY year, month;
```

### Window Functions

**ROW_NUMBER:**

```sql
SELECT 
    id,
    name,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY YEAR(created_at) ORDER BY created_at) AS rank
FROM users;
```

**LAG/LEAD:**

```sql
SELECT 
    date,
    sales,
    LAG(sales, 1) OVER (ORDER BY date) AS previous_sales,
    LEAD(sales, 1) OVER (ORDER BY date) AS next_sales
FROM daily_sales;
```

### Distribution and Performance

**Distribution keys:**

```sql
-- HASH distribution (for joins)
CREATE TABLE users (
    id INT,
    name VARCHAR(100)
)
WITH (
    DISTRIBUTION = HASH(id),
    CLUSTERED COLUMNSTORE INDEX
);

-- ROUND_ROBIN distribution (default)
CREATE TABLE logs (
    id INT,
    message VARCHAR(MAX)
)
WITH (
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
);
```

**Clustered Columnstore Index:**
- Optimized for analytics
- High compression
- Fast queries on large tables

---

## Integration with PowerBI

### Direct Connection

**Step 1: In PowerBI Desktop**

1. "Get Data" → "Azure" → "Azure Synapse Analytics SQL"
2. Enter information:
   - **Server** : `my-synapse-workspace-ondemand.sql.azuresynapse.net` (Serverless)
   - **Database** : Database name
   - **Data connectivity mode** : DirectQuery (recommended)

**Step 2: Authentication**

- **Authentication method** : Database
- **Username** : `sqladmin`
- **Password** : Your password

**Step 3: Select Tables**

- Choose tables or views
- Click on "Load"

### Create Views for PowerBI

**Optimized view:**

```sql
CREATE VIEW vw_user_analytics AS
SELECT 
    u.id,
    u.name,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name, u.email;
```

**Use the view in PowerBI:**
- Simpler for users
- Centralized business logic
- Optimized performance

---

## Best Practices

### Performance

1. **Use Columnstore Index** for analytics
2. **Choose the right distribution keys**
3. **Partition** large tables
4. **Optimize queries** with EXPLAIN

### Costs

1. **Use Serverless SQL** to start (pay-per-query)
2. **Pause the SQL Pool** when unused
3. **Monitor costs** in Azure Cost Management
4. **Use the right pool sizes**

### Organization

1. **Create schemas** to organize
2. **Name clearly** tables and views
3. **Document** schemas
4. **Use views** to simplify

### Security

1. **Use Azure AD** for authentication
2. **Limit access** with firewall rules
3. **Encrypt data** (enabled by default)
4. **Audit access**

---

## Practical Examples

### Example 1: Complete Data Lake → Synapse Pipeline

**Synapse Pipeline:**
1. Source: Data Lake Storage (Parquet)
2. Activity: Copy Data
3. Sink: SQL Pool
4. Trigger: Schedule (daily)

### Example 2: Complex Analytical Queries

```sql
-- Sales analysis with window functions
WITH monthly_sales AS (
    SELECT 
        YEAR(sale_date) AS year,
        MONTH(sale_date) AS month,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY YEAR(sale_date), MONTH(sale_date)
)
SELECT 
    year,
    month,
    total_sales,
    LAG(total_sales, 1) OVER (ORDER BY year, month) AS previous_month,
    (total_sales - LAG(total_sales, 1) OVER (ORDER BY year, month)) / 
        LAG(total_sales, 1) OVER (ORDER BY year, month) * 100 AS growth_percent
FROM monthly_sales
ORDER BY year, month;
```

### Example 3: Export to PowerBI

1. Create an analytical view
2. Connect PowerBI to the view
3. Create visualizations
4. Publish the report

---

## 📊 Key Takeaways

1. **Synapse = Unified analytics** platform
2. **SQL Pool** for data warehouse
3. **Serverless SQL** for pay-per-query
4. **Native PowerBI integration**
5. **Scalable** from a few GB to several PB

## 🔗 Next Module

Proceed to module [6. Azure Databricks - Big Data Analytics](../06-databricks/README.md) to learn how to use Databricks for Big Data.

