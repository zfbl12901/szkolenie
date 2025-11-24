# 3. AWS Glue - Serverless ETL

## 🎯 Objectives

- Understand AWS Glue and its role in ETL
- Create crawlers to discover data
- Create ETL jobs with Glue
- Transform data with PySpark
- Integrate Glue with S3 and other services

## 📋 Table of Contents

1. [Introduction to AWS Glue](#introduction-to-aws-glue)
2. [Create a Data Catalog](#create-a-data-catalog)
3. [Crawlers - Discover Data](#crawlers---discover-data)
4. [Create an ETL Job](#create-an-etl-job)
5. [Data Transformation](#data-transformation)
6. [Orchestration and Scheduling](#orchestration-and-scheduling)

---

## Introduction to AWS Glue

### What is AWS Glue?

**AWS Glue** = Managed serverless ETL service

- **ETL** : Extract, Transform, Load
- **Serverless** : No servers to manage
- **Managed** : AWS manages the infrastructure
- **Scalable** : Automatically adapts

### Glue Components

1. **Data Catalog** : Metadata catalog
2. **Crawlers** : Automatically discover schemas
3. **ETL Jobs** : Transformation scripts (Python/PySpark)
4. **Triggers** : Automatic triggering
5. **Workflows** : Orchestration of multiple jobs

### Glue Free Tier

**Free forever:**
- 10,000 objects/month in Data Catalog
- 1 million queries/month to Data Catalog
- $0.44 per DPU-hour (first million free)

**⚠️ Important:** Glue jobs consume DPU (Data Processing Units). Monitor costs.

---

## Create a Data Catalog

### What is the Data Catalog?

**Data Catalog** = Centralized metadata catalog

- Data schemas
- Locations (S3, databases)
- Data types
- Partitions

### Data Catalog Structure

- **Databases** : Groups of tables
- **Tables** : Data metadata
- **Partitions** : Data organization

### Create a Database

1. AWS Console → Glue → "Databases"
2. "Add database"
3. Name: `data_analyst_db`
4. Description (optional)
5. "Create"

**Usage:**
- Organize tables by project
- Example: `raw_data_db`, `processed_data_db`

---

## Crawlers - Discover Data

### What is a Crawler?

**Crawler** = Service that scans data and automatically creates tables

- Analyzes files in S3
- Automatically detects schema
- Creates tables in Data Catalog
- Supports: CSV, JSON, Parquet, etc.

### Create a Crawler

**Step 1: Basic Configuration**

1. Glue → "Crawlers" → "Add crawler"
2. Name: `s3-csv-crawler`
3. Description (optional)

**Step 2: Data Source**

1. "Add a data source"
2. Type: "S3"
3. S3 path: `s3://my-bucket/raw/`
4. Include subfolders (optional)

**Step 3: IAM Role**

1. Create a new role or use existing
2. Name: `AWSGlueServiceRole-default`
3. Permissions: S3 and Glue access

**Step 4: Output**

1. Database: `data_analyst_db`
2. Table prefix (optional)

**Step 5: Execute**

1. "Run crawler now" or schedule
2. Wait for completion (a few minutes)
3. Check created tables

### Crawler Result

**Automatically created table:**
- Detected columns
- Inferred data types
- S3 location
- File format

**Example of created table:**
```
Table: raw_data
Columns:
  - id (bigint)
  - name (string)
  - created_at (timestamp)
Location: s3://my-bucket/raw/
Format: csv
```

---

## Create an ETL Job

### Glue Job Types

1. **Spark** : PySpark jobs (recommended)
2. **Python shell** : Simple Python scripts
3. **Ray** : Advanced distributed processing

### Create a Spark Job

**Step 1: Configuration**

1. Glue → "ETL jobs" → "Add job"
2. Name: `transform-csv-job`
3. IAM Role: `AWSGlueServiceRole-default`
4. Type: "Spark"
5. Glue version: "4.0" (recommended)
6. DPU: 2 (minimum, adjustable)

**Step 2: Data Source**

1. "Data source": Select a table from Data Catalog
2. Or: Direct S3 path

**Step 3: Destination**

1. "Data target": S3
2. Format: Parquet (recommended for analytics)
3. Path: `s3://my-bucket/processed/`

**Step 4: Script**

1. Generate an automatic script
2. Or: Write a custom script

### Basic ETL Script

**Automatically generated script:**

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Read from Data Catalog
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_data"
)

# Transform (example: filter)
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# Write to S3
glueContext.write_dynamic_frame.from_options(
    frame = filtered,
    connection_type = "s3",
    connection_options = {"path": "s3://my-bucket/processed/"},
    format = "parquet"
)

job.commit()
```

---

## Data Transformation

### Common Transformations

#### 1. Filter Rows

```python
from awsglue.transforms import Filter

filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["age"] > 18
)
```

#### 2. Select Columns

```python
from awsglue.transforms import SelectFields

selected = SelectFields.apply(
    frame = datasource,
    paths = ["id", "name", "email"]
)
```

#### 3. Rename Columns

```python
from awsglue.transforms import RenameField

renamed = RenameField.apply(
    frame = datasource,
    old_name = "old_column",
    new_name = "new_column"
)
```

#### 4. Join Data

```python
joined = Join.apply(
    frame1 = datasource1,
    frame2 = datasource2,
    keys1 = ["id"],
    keys2 = ["user_id"]
)
```

#### 5. Aggregations

```python
# Convert to Spark DataFrame for aggregations
df = datasource.toDF()

aggregated = df.groupBy("category").agg({
    "amount": "sum",
    "id": "count"
})

# Convert back to DynamicFrame
from awsglue.dynamicframe import DynamicFrame
result = DynamicFrame.fromDF(aggregated, glueContext, "result")
```

### Complete Example: CSV → Parquet Transformation

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# 1. Read from S3 (via Data Catalog)
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_data"
)

# 2. Filter data
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# 3. Select columns
selected = SelectFields.apply(
    frame = filtered,
    paths = ["id", "name", "email", "created_at"]
)

# 4. Convert to DataFrame for advanced transformations
df = selected.toDF()

# 5. Add a calculated column
from pyspark.sql.functions import col, year
df = df.withColumn("year", year(col("created_at")))

# 6. Convert back to DynamicFrame
from awsglue.dynamicframe import DynamicFrame
result = DynamicFrame.fromDF(df, glueContext, "result")

# 7. Write to S3 in Parquet (partitioned by year)
glueContext.write_dynamic_frame.from_options(
    frame = result,
    connection_type = "s3",
    connection_options = {
        "path": "s3://my-bucket/processed/",
        "partitionKeys": ["year"]
    },
    format = "parquet"
)

job.commit()
```

---

## Orchestration and Scheduling

### Manually Trigger a Job

1. Glue → "ETL jobs"
2. Select the job
3. "Run job"
4. View real-time logs

### Schedule a Job (Trigger)

**Create a trigger:**

1. Glue → "Triggers" → "Add trigger"
2. Name: `daily-etl-trigger`
3. Type: "Scheduled"
4. Frequency: "Cron expression"
   - Example: `cron(0 2 * * ? *)` = Every day at 2 AM
5. Actions: Select the job to execute
6. "Add"

**Trigger types:**
- **On-demand** : Manual triggering
- **Scheduled** : Scheduled (cron)
- **Event-driven** : Triggered by event (e.g., new S3 file)

### Workflows (Complex Orchestration)

**Create a workflow:**

1. Glue → "Workflows" → "Add workflow"
2. Name: `etl-pipeline-workflow`
3. Add steps:
   - Crawler → ETL Job → Another Job
4. Define dependencies
5. Trigger the workflow

**Example workflow:**
```
1. S3 Crawler → Discovers new files
2. ETL Job 1 → Transforms raw data
3. ETL Job 2 → Aggregates data
4. ETL Job 3 → Loads into Redshift
```

---

## Best Practices

### Performance

1. **Use Parquet** instead of CSV (faster)
2. **Partition data** (improves performance)
3. **Adjust DPU** according to data size
4. **Use Spark cache** to reuse data

### Costs

1. **Monitor DPU-hours** used
2. **Optimize scripts** to reduce execution time
3. **Use appropriate S3 classes** (Standard-IA for archives)
4. **Stop jobs** that fail quickly

### Organization

1. **Name jobs** consistently
2. **Document transformations**
3. **Version scripts** (Git)
4. **Test locally** before deploying

---

## Practical Examples

### Example 1: Transform CSV → Parquet

```python
# Read CSV from S3
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "raw_csv_data"
)

# Write in Parquet
glueContext.write_dynamic_frame.from_options(
    frame = datasource,
    connection_type = "s3",
    connection_options = {"path": "s3://my-bucket/parquet/"},
    format = "parquet"
)
```

### Example 2: Clean and Validate

```python
# Filter invalid rows
cleaned = Filter.apply(
    frame = datasource,
    f = lambda x: x["email"] is not None and "@" in x["email"]
)

# Remove duplicates (via DataFrame)
df = cleaned.toDF()
df = df.dropDuplicates(["id"])

result = DynamicFrame.fromDF(df, glueContext, "result")
```

### Example 3: Join Multiple Sources

```python
# Read two tables
users = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "users"
)

orders = glueContext.create_dynamic_frame.from_catalog(
    database = "data_analyst_db",
    table_name = "orders"
)

# Join
joined = Join.apply(
    frame1 = users,
    frame2 = orders,
    keys1 = ["id"],
    keys2 = ["user_id"]
)
```

---

## 📊 Key Takeaways

1. **Glue = Serverless ETL** managed by AWS
2. **Crawlers** automatically discover schemas
3. **ETL Jobs** use PySpark for transformations
4. **Data Catalog** centralizes metadata
5. **Triggers** enable automation

## 🔗 Next Module

Proceed to module [4. Amazon Redshift - Data Warehouse](../04-redshift/README.md) to learn how to use Redshift for data analysis.

