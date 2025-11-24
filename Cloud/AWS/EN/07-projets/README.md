# 7. AWS Practical Projects

## 🎯 Objectives

- Apply acquired knowledge
- Create complete ETL pipelines
- Build a Data Lake on AWS
- Create projects for your portfolio
- Integrate multiple AWS services

## 📋 Table of Contents

1. [Project 1: ETL Pipeline S3 → Parquet](#project-1--etl-pipeline-s3---parquet)
2. [Project 2: Data Lake on AWS](#project-2--data-lake-on-aws)
3. [Project 3: Analytics with Athena](#project-3--analytics-with-athena)
4. [Project 4: Complete Automated Pipeline](#project-4--complete-automated-pipeline)
5. [Best Practices for Portfolio](#best-practices-for-portfolio)

---

## Project 1: ETL Pipeline S3 → Parquet

### Objective

Create an ETL pipeline that transforms CSV files from S3 into optimized Parquet format.

### Architecture

```
S3 (raw/) → Glue Crawler → Data Catalog → Glue Job → S3 (processed/parquet/)
```

### Steps

#### 1. Prepare Data

**Create an S3 bucket:**
- Name: `data-analyst-project-1`
- Create a `raw/` folder
- Upload a test CSV file

**Example CSV data:**
```csv
id,name,email,created_at,status
1,John Doe,john@example.com,2024-01-01,active
2,Jane Smith,jane@example.com,2024-01-02,inactive
```

#### 2. Create a Glue Crawler

1. Glue → "Crawlers" → "Add crawler"
2. Name: `csv-crawler`
3. Data source: `s3://data-analyst-project-1/raw/`
4. IAM Role: Create a role with S3 access
5. Database: `project1_db`
6. Execute the crawler

#### 3. Create a Glue Job

1. Glue → "ETL jobs" → "Add job"
2. Name: `csv-to-parquet-job`
3. Type: Spark
4. Script:

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
    database = "project1_db",
    table_name = "raw_data"
)

# Filter active data
filtered = Filter.apply(
    frame = datasource,
    f = lambda x: x["status"] == "active"
)

# Write in Parquet
glueContext.write_dynamic_frame.from_options(
    frame = filtered,
    connection_type = "s3",
    connection_options = {
        "path": "s3://data-analyst-project-1/processed/parquet/"
    },
    format = "parquet"
)

job.commit()
```

#### 4. Execute the Job

1. Select the job
2. "Run job"
3. Check logs
4. Check Parquet files in S3

### Result

- CSV files transformed to Parquet
- Filtered data (only active)
- Ready for analytics with Athena

---

## Project 2: Data Lake on AWS

### Objective

Create a complete Data Lake with ingestion, transformation, and analytics.

### Architecture

```
Sources → S3 (Raw) → Glue (Transform) → S3 (Processed) → Athena (Analytics)
                ↓
            Lambda (Trigger)
```

### Steps

#### 1. S3 Structure

```
data-lake-bucket/
├── raw/
│   ├── users/
│   ├── orders/
│   └── products/
├── processed/
│   ├── users/
│   ├── orders/
│   └── products/
└── analytics/
    └── results/
```

#### 2. Crawlers for Each Source

**Create 3 crawlers:**
- `users-crawler` → `s3://bucket/raw/users/`
- `orders-crawler` → `s3://bucket/raw/orders/`
- `products-crawler` → `s3://bucket/raw/products/`

#### 3. ETL Jobs for Transformation

**Job for users:**
```python
# users-etl-job
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "data_lake_db",
    table_name = "users"
)

# Clean and transform
cleaned = Filter.apply(
    frame = datasource,
    f = lambda x: x["email"] is not None
)

glueContext.write_dynamic_frame.from_options(
    frame = cleaned,
    connection_type = "s3",
    connection_options = {"path": "s3://bucket/processed/users/"},
    format = "parquet"
)
```

#### 4. Athena Tables for Analytics

```sql
-- Table users
CREATE EXTERNAL TABLE users_processed (
    id INT,
    name STRING,
    email STRING,
    created_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://bucket/processed/users/';

-- Table orders
CREATE EXTERNAL TABLE orders_processed (
    id INT,
    user_id INT,
    amount DECIMAL(10,2),
    created_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://bucket/processed/orders/';

-- Analytical query
SELECT 
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM users_processed u
LEFT JOIN orders_processed o ON u.id = o.user_id
GROUP BY u.name
ORDER BY total_spent DESC;
```

#### 5. Automation with Lambda

**Lambda triggered by S3 upload:**

```python
import boto3

glue = boto3.client('glue')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Determine which job to run based on prefix
    if 'users' in key:
        job_name = 'users-etl-job'
    elif 'orders' in key:
        job_name = 'orders-etl-job'
    else:
        job_name = 'products-etl-job'
    
    # Trigger the job
    glue.start_job_run(JobName=job_name)
    
    return {'statusCode': 200}
```

### Result

- Functional Data Lake
- Automated pipeline
- Analytics with Athena
- Complete project for portfolio

---

## Project 3: Analytics with Athena

### Objective

Create a complete analytics system with SQL queries on S3 data.

### Steps

#### 1. Prepare Data

**Upload Parquet files to S3:**
- `s3://analytics-bucket/sales/year=2024/month=01/`
- `s3://analytics-bucket/sales/year=2024/month=02/`

#### 2. Create Partitioned Tables

```sql
CREATE EXTERNAL TABLE sales (
    id INT,
    product_id INT,
    amount DECIMAL(10,2),
    sale_date TIMESTAMP
)
PARTITIONED BY (year INT, month INT)
STORED AS PARQUET
LOCATION 's3://analytics-bucket/sales/';

-- Add partitions
ALTER TABLE sales ADD PARTITION (year=2024, month=1)
LOCATION 's3://analytics-bucket/sales/year=2024/month=01/';

ALTER TABLE sales ADD PARTITION (year=2024, month=2)
LOCATION 's3://analytics-bucket/sales/year=2024/month=02/';
```

#### 3. Analytical Queries

**Sales per month:**
```sql
SELECT 
    year,
    month,
    SUM(amount) AS total_sales,
    COUNT(*) AS transaction_count,
    AVG(amount) AS avg_transaction
FROM sales
WHERE year = 2024
GROUP BY year, month
ORDER BY year, month;
```

**Top products:**
```sql
SELECT 
    product_id,
    SUM(amount) AS total_revenue,
    COUNT(*) AS sales_count
FROM sales
WHERE year = 2024
GROUP BY product_id
ORDER BY total_revenue DESC
LIMIT 10;
```

**Trends:**
```sql
SELECT 
    DATE_TRUNC('week', sale_date) AS week,
    SUM(amount) AS weekly_sales,
    LAG(SUM(amount), 1) OVER (ORDER BY DATE_TRUNC('week', sale_date)) AS previous_week
FROM sales
WHERE year = 2024
GROUP BY DATE_TRUNC('week', sale_date)
ORDER BY week;
```

#### 4. Save Results

**Create a table for results:**
```sql
CREATE EXTERNAL TABLE analytics_results (
    metric_name STRING,
    metric_value DECIMAL(10,2),
    calculated_at TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://analytics-bucket/results/';
```

---

## Project 4: Complete Automated Pipeline

### Objective

Create a completely automated ETL pipeline with multiple AWS services.

### Complete Architecture

```
CSV file uploaded → S3 (raw/)
    ↓ (Event)
Lambda (Validation)
    ↓
S3 (validated/)
    ↓ (Event)
Glue Job (Transform CSV → Parquet)
    ↓
S3 (processed/parquet/)
    ↓
Glue Crawler (Update Catalog)
    ↓
Athena (Analytics)
    ↓
S3 (results/)
```

### Implementation

#### 1. Validation Lambda

```python
import boto3
import csv

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Download and validate
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    reader = csv.DictReader(content.splitlines())
    
    valid_rows = []
    for row in reader:
        if row.get('email') and '@' in row['email']:
            valid_rows.append(row)
    
    # Upload validated data
    if valid_rows:
        validated_key = key.replace('raw/', 'validated/')
        # Convert to CSV and upload
        # ...
    
    return {'statusCode': 200}
```

#### 2. Glue Transformation Job

```python
# Transform validated CSV to Parquet
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "pipeline_db",
    table_name = "validated_data"
)

# Transform
transformed = Map.apply(
    frame = datasource,
    f = lambda x: {
        'id': x['id'],
        'name': x['name'].upper(),
        'email': x['email'].lower(),
        'created_at': x['created_at']
    }
)

# Write in Parquet
glueContext.write_dynamic_frame.from_options(
    frame = transformed,
    connection_type = "s3",
    connection_options = {"path": "s3://bucket/processed/"},
    format = "parquet"
)
```

#### 3. Glue Workflow

**Create a workflow:**
1. Trigger: New file in `validated/`
2. Action: Execute Glue job
3. Next action: Update crawler

### Result

- Completely automated pipeline
- Automatic validation
- Automatic transformation
- Analytics available immediately

---

## Best Practices for Portfolio

### Documentation

**Create a README for each project:**

```markdown
# Project: AWS ETL Pipeline

## Description
Automated ETL pipeline to transform CSV data to Parquet.

## Architecture
- S3: Storage
- Glue: Transformation
- Athena: Analytics

## Results
- 60% cost reduction
- 80% processing time reduction
```

### Visualizations

**Create diagrams:**
- System architecture
- Data flow
- Data schema

**Tools:**
- Draw.io
- Lucidchart
- ASCII diagrams in README

### Metrics

**Include metrics:**
- Execution time before/after
- Costs before/after
- Volume of processed data
- Query performance

### Code

**Best practices:**
- Commented code
- Environment variables for configuration
- Error handling
- Logging

### GitHub

**Create a repository:**
- README with documentation
- Lambda scripts
- Glue scripts
- Configuration
- Diagrams

---

## 📊 Key Takeaways

1. **Practical projects** : Essential for portfolio
2. **Documentation** : Explain architecture and results
3. **Metrics** : Show impact (performance, costs)
4. **Clean code** : Commented and organized
5. **GitHub** : Share your projects

## 🔗 Resources

- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Solutions](https://aws.amazon.com/solutions/)
- [GitHub AWS Examples](https://github.com/aws-samples)

---

**Congratulations!** You have completed the AWS training for Data Analyst. You can now create complete projects on AWS using only free resources.

