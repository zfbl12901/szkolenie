# 6. Azure Databricks - Big Data Analytics

## 🎯 Objectives

- Understand Azure Databricks
- Create a Databricks workspace
- Use Python/SQL notebooks
- Process data with Spark
- Integrate with other Azure services

## 📋 Table of Contents

1. [Introduction to Databricks](#introduction-to-databricks)
2. [Create a Databricks Workspace](#create-a-databricks-workspace)
3. [Create a Cluster](#create-a-cluster)
4. [Python/SQL Notebooks](#pythonsql-notebooks)
5. [Data Processing with Spark](#data-processing-with-spark)
6. [Integration with Other Services](#integration-with-other-services)

---

## Introduction to Databricks

### What is Azure Databricks?

**Azure Databricks** = Big Data platform based on Apache Spark

- **Apache Spark** : Distributed processing engine
- **Notebooks** : Python, SQL, Scala, R
- **Managed** : Microsoft manages the infrastructure
- **Scalable** : Auto-scaling clusters

### Use Cases for Data Analyst

- **Big Data processing** : Process large quantities
- **ETL** : Complex transformations
- **Machine Learning** : Integrated MLlib
- **Data Science** : Interactive notebooks

### Databricks Free Tier

**Free with Azure credit:**
- Use the 200$ free credit (30 days)
- After: normal billing

**⚠️ Important:** Databricks can be expensive. Monitor costs carefully.

---

## Create a Databricks Workspace

### Step 1: Access Databricks

1. Azure Portal → Search "Azure Databricks"
2. Click on "Azure Databricks"
3. Click on "Create"

### Step 2: Basic Configuration

**Basic Information:**
- **Subscription** : Choose your subscription
- **Resource group** : Create or use existing
- **Workspace name** : `my-databricks-workspace`
- **Region** : Choose the region
- **Pricing tier** : Standard (or Premium)

**Networking:**
- **Virtual network** : Create new or use existing
- **Public IP** : ✅ Enable (for easy access)

### Step 3: Create the Workspace

1. Click on "Review + create"
2. Verify configuration
3. Click on "Create"
4. Wait for creation (5-10 minutes)

**⚠️ Important:** Note the workspace URL.

### Step 4: Open Databricks

1. Once created, click on "Launch Workspace"
2. Databricks web interface
3. Sign in with Azure AD

---

## Create a Cluster

### Step 1: Access Clusters

1. Databricks Workspace → "Compute"
2. Click on "Create Cluster"

### Step 2: Cluster Configuration

**Basic Configuration:**
- **Cluster name** : `my-cluster`
- **Cluster mode** : Standard (or Single Node for tests)
- **Databricks runtime version** : Latest LTS (recommended)
- **Python version** : 3.11

**Node Type:**
- **Worker type** : Standard_DS3_v2 (to start)
- **Driver type** : Standard_DS3_v2
- **Min workers** : 0 (to save)
- **Max workers** : 2 (to start)

**⚠️ Important:** Min workers = 0 allows auto-termination when inactive.

### Step 3: Advanced Options

**Auto-termination:**
- ✅ Enable (stops cluster after inactivity)
- **Terminate after** : 30 minutes

**Tags:**
- Add tags for organization

### Step 4: Create the Cluster

1. Click on "Create Cluster"
2. Wait for startup (3-5 minutes)
3. Cluster ready when status = "Running"

**⚠️ Important:** Cluster consumes resources even when inactive. Stop when unused.

---

## Python/SQL Notebooks

### Create a Notebook

**Step 1: Create a Notebook**

1. Databricks Workspace → "Workspace"
2. Right-click → "Create" → "Notebook"
3. Name: `data-processing`
4. Language: Python (or SQL)
5. Cluster: Attach to created cluster

### Step 2: Use the Notebook

**Python Cells:**

```python
# Cell 1: Import libraries
import pandas as pd
from pyspark.sql import SparkSession

# Cell 2: Create a Spark session
spark = SparkSession.builder.appName("DataProcessing").getOrCreate()

# Cell 3: Read data
df = spark.read.csv("dbfs:/FileStore/data/users.csv", header=True, inferSchema=True)

# Cell 4: Display data
df.show()

# Cell 5: Transform
df_filtered = df.filter(df["status"] == "active")
df_filtered.show()
```

**SQL Cells:**

```sql
-- SQL Cell: Create a temporary view
CREATE OR REPLACE TEMPORARY VIEW users AS
SELECT * FROM csv.`dbfs:/FileStore/data/users.csv`

-- SQL Query
SELECT 
    YEAR(created_at) AS year,
    COUNT(*) AS user_count
FROM users
GROUP BY YEAR(created_at)
ORDER BY year;
```

### Execute a Notebook

- **Run cell** : Execute a cell
- **Run all** : Execute all cells
- **Run all above** : Execute all cells above

---

## Data Processing with Spark

### Read Data

**From Data Lake Storage:**

```python
# Read CSV
df = spark.read.csv(
    "abfss://container@account.dfs.core.windows.net/data/users.csv",
    header=True,
    inferSchema=True
)

# Read Parquet
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/data/users.parquet"
)

# Read JSON
df = spark.read.json(
    "abfss://container@account.dfs.core.windows.net/data/users.json"
)
```

**From Azure Blob Storage:**

```python
# Configure access
spark.conf.set(
    "fs.azure.account.key.accountname.blob.core.windows.net",
    "your-account-key"
)

# Read
df = spark.read.csv(
    "wasbs://container@accountname.blob.core.windows.net/data/users.csv",
    header=True
)
```

### Transform Data

**Filter:**

```python
df_filtered = df.filter(df["age"] > 18)
```

**Select Columns:**

```python
df_selected = df.select("id", "name", "email")
```

**Aggregations:**

```python
df_aggregated = df.groupBy("category").agg({
    "amount": "sum",
    "id": "count"
})
```

**Join:**

```python
df_joined = df1.join(df2, df1.id == df2.user_id, "inner")
```

### Write Data

**To Data Lake Storage:**

```python
# Write in Parquet
df.write.mode("overwrite").parquet(
    "abfss://container@account.dfs.core.windows.net/processed/users.parquet"
)

# Write in CSV
df.write.mode("overwrite").csv(
    "abfss://container@account.dfs.core.windows.net/processed/users.csv"
)
```

---

## Integration with Other Services

### Databricks + Data Lake Storage

**Direct Access:**

```python
# Configure access
spark.conf.set(
    "fs.azure.account.auth.type.account.dfs.core.windows.net",
    "OAuth"
)
spark.conf.set(
    "fs.azure.account.oauth.provider.type.account.dfs.core.windows.net",
    "org.apache.hadoop.fs.azurebfs.oauth2.MsiTokenProvider"
)

# Read
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/data/users.parquet"
)
```

### Databricks + Azure SQL Database

**Read from SQL Database:**

```python
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:sqlserver://server.database.windows.net:1433;database=db") \
    .option("dbtable", "users") \
    .option("user", "sqladmin") \
    .option("password", "password") \
    .load()
```

**Write to SQL Database:**

```python
df.write \
    .format("jdbc") \
    .option("url", "jdbc:sqlserver://server.database.windows.net:1433;database=db") \
    .option("dbtable", "users_processed") \
    .option("user", "sqladmin") \
    .option("password", "password") \
    .mode("overwrite") \
    .save()
```

### Databricks + Data Factory

**Data Factory Pipeline:**
1. Source: Azure Blob Storage
2. Activity: Databricks Notebook
3. Sink: Azure SQL Database

**Configuration:**
- Notebook path: `/Workspace/path/to/notebook`
- Parameters: Pass parameters

---

## Best Practices

### Performance

1. **Use cache** to reuse data
2. **Partition** data to improve performance
3. **Optimize transformations** to reduce time
4. **Use the right number of workers**

### Costs

1. **Stop clusters** when unused
2. **Use auto-termination** to save
3. **Monitor costs** in Azure Cost Management
4. **Use smaller clusters** to start

### Organization

1. **Organize notebooks** in folders
2. **Name clearly** notebooks and clusters
3. **Document** code
4. **Version** with Git

### Security

1. **Use Azure AD** for authentication
2. **Limit access** with RBAC
3. **Encrypt data** in transit and at rest
4. **Audit** access

---

## Practical Examples

### Example 1: Complete ETL Pipeline

**Databricks Notebook:**

```python
# 1. Read from Data Lake
df = spark.read.parquet(
    "abfss://container@account.dfs.core.windows.net/raw/users.parquet"
)

# 2. Transform
df_processed = df \
    .filter(df["status"] == "active") \
    .select("id", "name", "email", "created_at") \
    .withColumn("year", year(col("created_at")))

# 3. Write to Data Lake
df_processed.write.mode("overwrite").parquet(
    "abfss://container@account.dfs.core.windows.net/processed/users.parquet"
)
```

### Example 2: Analysis with Spark SQL

```python
# Create a temporary view
df.createOrReplaceTempView("users")

# SQL Query
result = spark.sql("""
    SELECT 
        YEAR(created_at) AS year,
        COUNT(*) AS user_count,
        COUNT(DISTINCT email) AS unique_emails
    FROM users
    GROUP BY YEAR(created_at)
    ORDER BY year
""")

result.show()
```

### Example 3: Integration with Data Factory

1. Create a Databricks notebook
2. In Data Factory, add "Databricks Notebook" activity
3. Configure the notebook
4. Execute the pipeline

---

## 📊 Key Takeaways

1. **Databricks = Big Data** with Apache Spark
2. **Notebooks** Python/SQL for development
3. **Auto-scaling clusters** for performance
4. **Native integration** with Azure services
5. **Paid** : Monitor costs

## 🔗 Next Module

Proceed to module [7. Practical Projects](../07-projets/README.md) to create complete projects with Azure.

