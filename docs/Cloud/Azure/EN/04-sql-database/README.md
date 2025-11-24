# 4. Azure SQL Database - Database

## 🎯 Objectives

- Understand Azure SQL Database
- Create a SQL Database (free up to 32 GB)
- Migrate data
- Optimize queries
- Integrate with PowerBI

## 📋 Table of Contents

1. [Introduction to SQL Database](#introduction-to-sql-database)
2. [Create a SQL Database](#create-a-sql-database)
3. [Connect to the Database](#connect-to-the-database)
4. [Load Data](#load-data)
5. [SQL Queries](#sql-queries)
6. [Integration with PowerBI](#integration-with-powerbi)

---

## Introduction to SQL Database

### What is Azure SQL Database?

**Azure SQL Database** = Managed cloud SQL database

- **SQL Server compatible** : Standard SQL syntax
- **Managed** : Microsoft manages the infrastructure
- **Scalable** : From a few GB to several TB
- **High availability** : 99.99% availability

### Use Cases for Data Analyst

- **Data Warehouse** : Centralize data
- **Analytics** : Complex queries
- **Business Intelligence** : Source for PowerBI
- **Data Integration** : Central point for ETL

### SQL Database Free Tier

**Free for 12 months:**
- **Basic tier** : Up to 32 GB
- **DTU** : 5 DTU (Database Transaction Units)
- **Backup** : Automatic (7 days)

**⚠️ Important:** After 12 months, normal billing. Monitor costs.

---

## Create a SQL Database

### Step 1: Access SQL Database

1. Azure Portal → Search "SQL databases"
2. Click on "SQL databases"
3. Click on "Create"

### Step 2: Basic Configuration

**Basic Information:**
- **Subscription** : Choose your subscription
- **Resource group** : Create or use existing
- **Database name** : `analytics-db`
- **Server** : Create a new server or use existing

**Create a SQL Server:**
- **Server name** : `my-sql-server-xxxxx` (globally unique)
- **Location** : Choose the region
- **Authentication method** : SQL authentication (or Azure AD)
- **Server admin login** : `sqladmin` (or other)
- **Password** : Strong password
- **Allow Azure services** : ✅ Yes (for Data Factory)

### Step 3: Database Configuration

**Compute + storage:**
- **Service tier** : Basic (for Free Tier)
- **Compute tier** : Serverless (or Provisioned)
- **Storage** : 2 GB (free, extensible up to 32 GB)

**⚠️ Important:** Basic tier = 5 DTU, sufficient to start.

### Step 4: Network Configuration

**Networking:**
- **Public endpoint** : ✅ Enable
- **Firewall rules** :
  - ✅ Allow Azure services and resources
  - Add your IP for local access

### Step 5: Create the Database

1. Click on "Review + create"
2. Verify configuration
3. Click on "Create"
4. Wait for creation (2-3 minutes)

**⚠️ Important:** Note the server name and credentials.

---

## Connect to the Database

### Via Azure Portal (Query Editor)

1. SQL Database → "Query editor"
2. Enter credentials
3. Execute SQL queries

### Via SQL Server Management Studio (SSMS)

**Download SSMS:**
- https://aka.ms/ssmsfullsetup

**Connection:**
- **Server name** : `my-sql-server-xxxxx.database.windows.net`
- **Authentication** : SQL Server Authentication
- **Login** : `sqladmin`
- **Password** : Your password

### Via Azure Data Studio

**Download Azure Data Studio:**
- https://aka.ms/azuredatastudio

**Advantages:**
- Free and open-source
- Modern interface
- Notebook support
- Git integration

### Via Python (pyodbc)

```python
import pyodbc

# Connection
server = 'my-sql-server-xxxxx.database.windows.net'
database = 'analytics-db'
username = 'sqladmin'
password = 'your-password'
driver = '{ODBC Driver 17 for SQL Server}'

conn = pyodbc.connect(
    f'DRIVER={driver};SERVER={server};DATABASE={database};UID={username};PWD={password}'
)

# Execute a query
cursor = conn.cursor()
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()
for row in rows:
    print(row)
```

---

## Load Data

### Method 1: INSERT (Small Quantities)

```sql
INSERT INTO users (id, name, email, created_at)
VALUES (1, 'John Doe', 'john@example.com', '2024-01-01');
```

### Method 2: BULK INSERT from Blob Storage

**Prerequisites:**
- Create a SAS key for Blob Storage
- Create a credential in SQL Database

**Example:**

```sql
-- Create a credential
CREATE DATABASE SCOPED CREDENTIAL BlobCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
SECRET = 'your-sas-token';

-- Create an external data source
CREATE EXTERNAL DATA SOURCE BlobStorage
WITH (
    TYPE = BLOB_STORAGE,
    LOCATION = 'https://mystorageaccount.blob.core.windows.net',
    CREDENTIAL = BlobCredential
);

-- Import from Blob Storage
BULK INSERT users
FROM 'raw-data/users.csv'
WITH (
    DATA_SOURCE = 'BlobStorage',
    FORMAT = 'CSV',
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
);
```

### Method 3: Via Data Factory

**Pipeline:**
1. Source: Azure Blob Storage (CSV)
2. Activity: Copy Data
3. Sink: Azure SQL Database

**Configuration:**
- Source: `raw-data/users.csv`
- Sink: Table `users` in SQL Database
- Mapping: Automatic or manual columns

### Method 4: Via Python (pandas)

```python
import pandas as pd
import pyodbc

# Read a CSV file
df = pd.read_csv('users.csv')

# Connection
conn = pyodbc.connect(connection_string)

# Write to SQL Database
df.to_sql('users', conn, if_exists='append', index=False)
```

---

## SQL Queries

### Basic Queries

**Simple SELECT:**

```sql
SELECT * FROM users LIMIT 10;
```

**Filter:**

```sql
SELECT id, name, email
FROM users
WHERE created_at > '2024-01-01'
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
WHERE o.created_at > '2024-01-01';
```

**CTE (Common Table Expressions):**

```sql
WITH monthly_users AS (
    SELECT 
        DATE_TRUNC('month', created_at) AS month,
        COUNT(*) AS user_count
    FROM users
    GROUP BY DATE_TRUNC('month', created_at)
)
SELECT 
    month,
    user_count,
    LAG(user_count, 1) OVER (ORDER BY month) AS previous_month
FROM monthly_users;
```

---

## Integration with PowerBI

### Direct Connection

**Step 1: In PowerBI Desktop**

1. "Get Data" → "Azure" → "Azure SQL Database"
2. Enter information:
   - **Server** : `my-sql-server-xxxxx.database.windows.net`
   - **Database** : `analytics-db`
   - **Data connectivity mode** : Import (or DirectQuery)

**Step 2: Authentication**

- **Authentication method** : Database
- **Username** : `sqladmin`
- **Password** : Your password

**Step 3: Select Tables**

- Choose tables to import
- Click on "Load"

### DirectQuery vs Import

**Import:**
- ✅ Fast for visualizations
- ✅ Works offline
- ❌ Static data (requires refresh)

**DirectQuery:**
- ✅ Real-time data
- ✅ No size limit
- ❌ Slower (queries on each interaction)

### Create Visualizations

**Example:**
1. Import the `users` table
2. Create a chart: Number of users per month
3. Add filters
4. Publish to PowerBI Service

---

## Best Practices

### Performance

1. **Create indexes** on frequently used columns
2. **Optimize queries** with EXPLAIN
3. **Use views** to simplify
4. **Partition** large tables

### Costs

1. **Monitor usage** in Azure Cost Management
2. **Use Basic tier** to start
3. **Stop the database** if unused (Serverless)
4. **Clean up** unnecessary data

### Security

1. **Use Azure AD** for authentication
2. **Limit access** with firewall rules
3. **Encrypt data** (enabled by default)
4. **Audit access** with SQL Auditing

### Organization

1. **Name clearly** tables and columns
2. **Document** schemas
3. **Use schemas** to organize
4. **Version** SQL scripts (Git)

---

## Practical Examples

### Example 1: Complete Blob → SQL Database Pipeline

**Via Data Factory:**
1. Source: Azure Blob Storage (CSV)
2. Activity: Copy Data
3. Sink: Azure SQL Database
4. Trigger: Schedule (daily)

### Example 2: Analytical Queries

```sql
-- Top 10 users by spending
SELECT TOP 10
    u.name,
    SUM(o.amount) AS total_spent,
    COUNT(o.id) AS order_count
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= DATEADD(month, -3, GETDATE())
GROUP BY u.name
ORDER BY total_spent DESC;
```

### Example 3: Export to PowerBI

1. Create a view for PowerBI
2. Connect PowerBI to the view
3. Create visualizations
4. Publish the report

---

## 📊 Key Takeaways

1. **SQL Database = Cloud SQL database** managed by Microsoft
2. **Free Tier: 32 GB** for 12 months (Basic tier)
3. **SQL Server compatible** : Standard syntax
4. **PowerBI integration** : Direct connection
5. **Scalable** : From Basic to Premium

## 🔗 Next Module

Proceed to module [5. Azure Synapse Analytics - Data Warehouse](../05-synapse/README.md) to learn how to use Synapse for data analysis.

