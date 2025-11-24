# 2. Amazon S3 - Data Storage

## 🎯 Objectives

- Understand Amazon S3 and its usage
- Create and manage S3 buckets
- Upload and organize files
- Understand storage classes
- Integrate S3 with other AWS services

## 📋 Table of Contents

1. [Introduction to S3](#introduction-to-s3)
2. [Create an S3 Bucket](#create-an-s3-bucket)
3. [Upload and Manage Files](#upload-and-manage-files)
4. [Storage Classes](#storage-classes)
5. [Data Organization](#data-organization)
6. [Integration with Other Services](#integration-with-other-services)

---

## Introduction to S3

### What is Amazon S3?

**Amazon S3** (Simple Storage Service) = Object storage service

- Unlimited storage
- High availability (99.99%)
- Secure by default
- Integration with all AWS services

### Use Cases for Data Analyst

- **Data Lake**: Store raw data
- **Backup**: Backup data
- **ETL**: Source/destination for pipelines
- **Analytics**: Data for Athena, Redshift
- **Archive**: Historical data

### S3 Free Tier

**Free forever:**
- 5 GB standard storage
- 20,000 GET requests
- 2,000 PUT requests
- 15 GB data transfer out

**⚠️ Important:** Beyond these limits, normal billing.

---

## Create an S3 Bucket

### Step 1: Access S3

1. AWS Console → Search "S3"
2. Click "Amazon S3"
3. Click "Create bucket"

### Step 2: Bucket Configuration

**Basic information:**
- **Bucket name**: Globally unique name (ex: `my-data-analyst-bucket`)
- **Region**: Choose closest region (ex: `eu-west-3` Paris)

**Configuration options:**

1. **Object Ownership**
   - "ACLs disabled" (recommended)
   - "Bucket owner enforced"

2. **Block Public Access**
   - ✅ **Enable all** (default security)
   - Disable only if specific need

3. **Versioning**
   - Disabled by default (free)
   - Enable if need multiple versions

4. **Tags** (optional)
   - Add tags for organization
   - Ex: `Project: Data-Analyst-Training`

5. **Default encryption**
   - ✅ Enable (recommended)
   - "Amazon S3 managed keys (SSE-S3)" (free)

### Step 3: Create Bucket

1. Click "Create bucket"
2. Bucket created and visible in list
3. Ready to use

**⚠️ Important:** Bucket name must be globally unique in AWS.

---

## Upload and Manage Files

### Upload a File

**Method 1: Web Interface**

1. Click bucket name
2. Click "Upload"
3. "Add files" or "Add folder"
4. Select files
5. Click "Upload"

**Method 2: AWS CLI**

```bash
# Install AWS CLI (if not already)
# Windows: https://aws.amazon.com/cli/
# Linux/Mac: pip install awscli

# Configure credentials
aws configure

# Upload a file
aws s3 cp local-file.csv s3://my-data-analyst-bucket/data/
```

**Method 3: Python SDK (boto3)**

```python
import boto3

# Create S3 client
s3 = boto3.client('s3')

# Upload file
s3.upload_file('local-file.csv', 'my-data-analyst-bucket', 'data/file.csv')
```

### Download a File

**Web interface:**
1. Click file
2. Click "Download"

**AWS CLI:**
```bash
aws s3 cp s3://my-data-analyst-bucket/data/file.csv local-file.csv
```

**Python:**
```python
s3.download_file('my-data-analyst-bucket', 'data/file.csv', 'local-file.csv')
```

### Manage Files

**Available actions:**
- **Download**: Download
- **Open**: Open in browser
- **Copy**: Copy to another location
- **Move**: Move
- **Delete**: Delete
- **Make public**: Make public (security attention)

---

## Storage Classes

### S3 Standard (default)

**Usage:**
- Frequently accessed data
- Production applications

**Characteristics:**
- Fast access
- 99.99% availability
- Cost: ~$0.023 per GB/month

**Free Tier:** 5 GB free

### S3 Intelligent-Tiering

**Usage:**
- Data with variable access
- Automatic cost optimization

**Characteristics:**
- Automatically moves between classes
- No retrieval fees
- Cost: ~$0.023 per GB/month

### S3 Standard-IA (Infrequent Access)

**Usage:**
- Rarely accessed data
- Backup, archives

**Characteristics:**
- Fast access when needed
- Storage cost: ~$0.0125 per GB/month
- Retrieval cost: ~$0.01 per GB

### S3 One Zone-IA

**Usage:**
- Reproducible data
- Secondary backup

**Characteristics:**
- Storage in single zone
- Cost: ~$0.01 per GB/month
- ⚠️ Risk of loss if zone fails

### S3 Glacier

**Usage:**
- Long-term archiving
- Rarely needed data

**Characteristics:**
- Retrieval: 1-5 minutes to several hours
- Cost: ~$0.004 per GB/month
- Retrieval fees by speed

### Choose Storage Class

**For Data Analyst:**
- **S3 Standard**: Active data (frequent analysis)
- **S3 Standard-IA**: Historical data (occasional analysis)
- **S3 Glacier**: Archives (rarely used)

**Automatic transition:**
- Configure transition rules
- Example: Standard → Standard-IA after 30 days

---

## Data Organization

### Recommended Structure

**Organization by project:**
```
bucket-name/
├── raw/              # Raw data
│   ├── 2024/
│   │   ├── 01/
│   │   ├── 02/
│   │   └── ...
├── processed/        # Transformed data
│   ├── 2024/
│   └── ...
├── analytics/        # Data for analysis
│   └── ...
└── archive/          # Archives
    └── ...
```

**Organization by type:**
```
bucket-name/
├── csv/
├── json/
├── parquet/
└── logs/
```

### Prefixes and Folders

**S3 doesn't have "real" folders**, but uses prefixes:

- `data/2024/01/file.csv` = Prefix `data/2024/01/`
- Web interface simulates folders
- Use `/` to organize

**Best practices:**
- Use consistent prefixes
- Include date in path
- Separate by data type

---

## Integration with Other Services

### S3 + AWS Glue

**Usage:**
- S3 as data source
- Glue transforms data
- Result to S3 or other destination

**Example:**
```python
# Glue job reads from S3
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "my_database",
    table_name = "s3_data"
)
```

### S3 + Amazon Athena

**Usage:**
- SQL queries directly on S3 files
- No need to load into database
- Pay-per-query

**Example:**
```sql
-- Create external table pointing to S3
CREATE EXTERNAL TABLE my_table (
    id INT,
    name STRING
)
STORED AS PARQUET
LOCATION 's3://my-bucket/data/';
```

### S3 + Amazon Redshift

**Usage:**
- S3 as source for COPY
- Redshift as data warehouse
- Fast loading of large amounts

**Example:**
```sql
COPY my_table
FROM 's3://my-bucket/data/file.csv'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV;
```

### S3 + AWS Lambda

**Usage:**
- Trigger Lambda on upload
- Automatic file processing
- Transformation, validation, etc.

**Configuration:**
1. S3 → Properties → Event notifications
2. Create notification
3. Trigger: "All object create events"
4. Destination: Lambda function

---

## Best Practices

### Security

1. **Never make buckets public** (except specific need)
2. **Use IAM** to control access
3. **Enable encryption** by default
4. **Use bucket policies** for granular permissions

### Performance

1. **Use prefixes** to distribute load
2. **Avoid sequential names** (ex: file1, file2, file3)
3. **Use Multipart Upload** for large files (>100MB)
4. **Enable Transfer Acceleration** if needed (paid)

### Costs

1. **Monitor usage** regularly
2. **Use right storage classes**
3. **Delete unused files**
4. **Configure automatic transitions**
5. **Use S3 Lifecycle** to automate

### Organization

1. **Name buckets** consistently
2. **Use tags** for organization
3. **Document data structure**
4. **Create naming conventions**

---

## Practical Examples

### Example 1: Upload CSV File

```python
import boto3
import pandas as pd

# Create S3 client
s3 = boto3.client('s3')

# Read local file
df = pd.read_csv('data.csv')

# Upload to S3
s3.upload_file('data.csv', 'my-bucket', 'raw/2024/data.csv')
```

### Example 2: List Files in Prefix

```python
# List all files in a prefix
response = s3.list_objects_v2(
    Bucket='my-bucket',
    Prefix='raw/2024/'
)

for obj in response.get('Contents', []):
    print(obj['Key'], obj['Size'])
```

### Example 3: Download and Process

```python
# Download from S3
s3.download_file('my-bucket', 'raw/data.csv', 'local-data.csv')

# Process
df = pd.read_csv('local-data.csv')
# ... processing ...

# Upload result
df.to_csv('processed-data.csv', index=False)
s3.upload_file('processed-data.csv', 'my-bucket', 'processed/data.csv')
```

---

## 📊 Key Points to Remember

1. **S3 = Unlimited storage** and highly available
2. **Free Tier: 5 GB** always free
3. **Organize with prefixes** for better performance
4. **Choose right storage class** according to usage
5. **S3 integrates** with all AWS data services

## 🔗 Next Module

Go to module [3. AWS Glue - Serverless ETL](../03-glue/README.md) to learn how to transform data with AWS Glue.

