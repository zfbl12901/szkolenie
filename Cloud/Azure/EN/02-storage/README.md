# 2. Azure Storage - Data Storage

## 🎯 Objectives

- Understand Azure Storage and its usage
- Create storage accounts
- Use Blob Storage and Data Lake Storage
- Upload and manage files
- Organize data

## 📋 Table of Contents

1. [Introduction to Azure Storage](#introduction-to-azure-storage)
2. [Create a Storage Account](#create-a-storage-account)
3. [Blob Storage](#blob-storage)
4. [Data Lake Storage Gen2](#data-lake-storage-gen2)
5. [Upload and Manage Files](#upload-and-manage-files)
6. [Integration with Other Services](#integration-with-other-services)

---

## Introduction to Azure Storage

### What is Azure Storage?

**Azure Storage** = Managed cloud storage service

- **Unlimited storage** : Scalable according to needs
- **High availability** : 99.99% availability
- **Secure** : Encryption by default
- **Integration** : With all Azure services

### Storage Types

1. **Blob Storage** : Files (CSV, JSON, Parquet, etc.)
2. **Data Lake Storage Gen2** : Data Lake with hierarchical file system
3. **File Storage** : File shares
4. **Queue Storage** : Queues
5. **Table Storage** : NoSQL storage

### Azure Storage Free Tier

**Free for 12 months:**
- 5 GB Blob storage
- 5 GB File storage
- 5 GB Table storage
- 5 GB Queue storage

**Free forever:**
- 200 GB outbound data transfer/month

**⚠️ Important:** Beyond these limits, normal billing.

---

## Create a Storage Account

### Step 1: Access Azure Storage

1. Azure Portal → Search "Storage accounts"
2. Click on "Storage accounts"
3. Click on "Create"

### Step 2: Basic Configuration

**Basic Information:**
- **Subscription** : Choose your subscription
- **Resource group** : Create or use existing
- **Storage account name** : Globally unique name (e.g., `mydataanalyststorage`)
- **Region** : Choose the closest region (e.g., `France Central`)

**Performance Options:**
- **Performance** : Standard (recommended to start)
- **Redundancy** : LRS (Locally Redundant Storage) - cheapest

### Step 3: Advanced Options

**Security:**
- **Secure transfer required** : ✅ Enable (recommended)
- **Allow Blob public access** : ❌ Disable (security)

**Data Lake Storage Gen2:**
- **Hierarchical namespace** : ✅ Enable if Data Lake needed

### Step 4: Create the Account

1. Click on "Review + create"
2. Verify configuration
3. Click on "Create"
4. Wait for creation (1-2 minutes)

**⚠️ Important:** Note the storage account name.

---

## Blob Storage

### What is Blob Storage?

**Blob Storage** = Object storage for files

- **Containers** : Organize files (like folders)
- **Blobs** : Individual files
- **Types** : Block blobs, Page blobs, Append blobs

### Create a Container

**Via Azure Portal:**

1. Storage account → "Containers"
2. Click on "+ Container"
3. Name: `raw-data` (or other)
4. Public access level: Private (recommended)
5. Click on "Create"

**Via Azure CLI:**

```bash
az storage container create \
  --name raw-data \
  --account-name mydataanalyststorage \
  --auth-mode login
```

**Via Python:**

```python
from azure.storage.blob import BlobServiceClient

# Connection
connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Create a container
container_client = blob_service_client.create_container("raw-data")
```

### Blob Types

**Block Blobs:**
- Files (CSV, JSON, Parquet, images, etc.)
- Up to 4.75 TB per blob
- Recommended for most cases

**Page Blobs:**
- Virtual disks
- Up to 8 TB

**Append Blobs:**
- Logs
- Append-only data

---

## Data Lake Storage Gen2

### What is Data Lake Storage Gen2?

**Data Lake Storage Gen2** = Blob Storage + hierarchical file system

- **Blob Storage compatible** : Uses the same APIs
- **File system** : Hierarchical organization
- **Big Data optimized** : For analytics and ML
- **Integration** : With Azure Synapse, Databricks, etc.

### Enable Data Lake Storage Gen2

**When creating the account:**
1. In "Advanced" → Enable "Hierarchical namespace"
2. Create the account

**⚠️ Important:** Cannot be enabled after creation.

### Data Lake Structure

```
data-lake/
├── raw/
│   ├── 2024/
│   │   ├── 01/
│   │   └── 02/
├── processed/
│   └── 2024/
└── analytics/
    └── results/
```

### Create Files and Folders

**Via Azure Portal:**

1. Storage account → "Data Lake"
2. Navigate the structure
3. Upload files
4. Create folders

**Via Python:**

```python
from azure.storage.filedatalake import DataLakeServiceClient

# Connection
account_name = "mydataanalyststorage"
account_key = "..."
datalake_service_client = DataLakeServiceClient(
    account_url=f"https://{account_name}.dfs.core.windows.net",
    credential=account_key
)

# Create a file system
file_system_client = datalake_service_client.create_file_system("data-lake")

# Create a directory
directory_client = file_system_client.create_directory("raw/2024")
```

---

## Upload and Manage Files

### Upload a File

**Via Azure Portal:**

1. Container → "Upload"
2. Select the file
3. Click on "Upload"

**Via Azure CLI:**

```bash
az storage blob upload \
  --account-name mydataanalyststorage \
  --container-name raw-data \
  --name data.csv \
  --file ./local-data.csv \
  --auth-mode login
```

**Via Python:**

```python
from azure.storage.blob import BlobServiceClient

blob_service_client = BlobServiceClient.from_connection_string(connection_string)
container_client = blob_service_client.get_container_client("raw-data")

# Upload a file
with open("local-data.csv", "rb") as data:
    container_client.upload_blob(name="data.csv", data=data)
```

### Download a File

**Via Python:**

```python
# Download a blob
blob_client = container_client.get_blob_client("data.csv")
with open("downloaded-data.csv", "wb") as download_file:
    download_file.write(blob_client.download_blob().readall())
```

### List Files

**Via Python:**

```python
# List all blobs in a container
blob_list = container_client.list_blobs()
for blob in blob_list:
    print(f"Name: {blob.name}, Size: {blob.size}")
```

### Delete a File

**Via Python:**

```python
# Delete a blob
blob_client = container_client.get_blob_client("data.csv")
blob_client.delete_blob()
```

---

## Integration with Other Services

### Azure Storage + Data Factory

**Usage:**
- Data source for ETL pipelines
- Destination for transformed data

**Example:**
```json
{
  "type": "AzureBlobStorage",
  "typeProperties": {
    "connectionString": "...",
    "container": "raw-data"
  }
}
```

### Azure Storage + Azure SQL Database

**Usage:**
- Import data from Blob Storage
- Export data to Blob Storage

**SQL Example:**
```sql
-- Import from Blob Storage
BULK INSERT my_table
FROM 'https://mystorageaccount.blob.core.windows.net/raw-data/data.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2
);
```

### Azure Storage + PowerBI

**Usage:**
- Connect PowerBI to Blob Storage
- Analyze files directly

**Configuration:**
1. PowerBI → "Get Data"
2. "Azure Blob Storage"
3. Enter container URL
4. Select files

### Azure Storage + Azure Functions

**Usage:**
- Trigger Functions on upload
- Automatically process files

**Configuration:**
1. Function → "Add trigger"
2. "Azure Blob Storage trigger"
3. Configure container and path

---

## Best Practices

### Organization

1. **Use containers** to organize by project
2. **Name clearly** files and containers
3. **Organize by date** : `raw/2024/01/data.csv`
4. **Separate by type** : `raw/`, `processed/`, `analytics/`

### Performance

1. **Use random names** for blobs (avoid sequences)
2. **Enable CDN** if global distribution needed (paid)
3. **Use block blobs** for most cases
4. **Partition data** to improve performance

### Costs

1. **Monitor usage** in Azure Cost Management
2. **Delete unnecessary files**
3. **Use appropriate storage classes**
4. **Configure lifecycle rules** to automate

### Security

1. **Never make containers public** (except specific need)
2. **Use SAS (Shared Access Signature)** for temporary access
3. **Enable encryption** by default
4. **Use Azure AD** for authentication

---

## Practical Examples

### Example 1: Upload a CSV File

```python
from azure.storage.blob import BlobServiceClient
import pandas as pd

# Connection
connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)
container_client = blob_service_client.get_container_client("raw-data")

# Read a local file
df = pd.read_csv("local-data.csv")

# Upload to Azure Storage
with open("local-data.csv", "rb") as data:
    container_client.upload_blob(name="2024/01/data.csv", data=data)
```

### Example 2: Download and Process

```python
# Download from Azure Storage
blob_client = container_client.get_blob_client("2024/01/data.csv")
with open("downloaded-data.csv", "wb") as download_file:
    download_file.write(blob_client.download_blob().readall())

# Process
df = pd.read_csv("downloaded-data.csv")
# ... processing ...

# Upload result
df.to_csv("processed-data.csv", index=False)
with open("processed-data.csv", "rb") as data:
    container_client.upload_blob(name="processed/2024/01/data.csv", data=data)
```

### Example 3: List and Filter

```python
# List all files in a prefix
blob_list = container_client.list_blobs(name_starts_with="2024/01/")
for blob in blob_list:
    print(f"File: {blob.name}, Size: {blob.size} bytes, Modified: {blob.last_modified}")
```

---

## 📊 Key Takeaways

1. **Azure Storage = Unlimited storage** and highly available
2. **Free Tier: 5 GB** for 12 months
3. **Blob Storage** for files, **Data Lake Gen2** for Big Data
4. **Organize with containers** and prefixes
5. **Native integration** with all Azure data services

## 🔗 Next Module

Proceed to module [3. Azure Data Factory - Cloud ETL](../03-data-factory/README.md) to learn how to create ETL pipelines on Azure.

