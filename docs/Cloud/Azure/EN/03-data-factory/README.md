# 3. Azure Data Factory - Cloud ETL

## 🎯 Objectives

- Understand Azure Data Factory and its role
- Create ETL pipelines
- Use transformation activities
- Integrate with data sources
- Orchestrate workflows

## 📋 Table of Contents

1. [Introduction to Data Factory](#introduction-to-data-factory)
2. [Create a Data Factory](#create-a-data-factory)
3. [Create a Pipeline](#create-a-pipeline)
4. [Transformation Activities](#transformation-activities)
5. [Integration with Data Sources](#integration-with-data-sources)
6. [Orchestration and Scheduling](#orchestration-and-scheduling)

---

## Introduction to Data Factory

### What is Azure Data Factory?

**Azure Data Factory** = Managed cloud ETL service

- **ETL** : Extract, Transform, Load
- **Cloud** : No infrastructure to manage
- **Managed** : Microsoft manages the infrastructure
- **Scalable** : Automatically adapts

### Data Factory Components

1. **Pipelines** : ETL workflows
2. **Activities** : Steps in a pipeline
3. **Datasets** : Data representations
4. **Linked Services** : Connections to sources
5. **Triggers** : Automatic triggering

### Data Factory Free Tier

**Free forever:**
- 5 free pipelines
- Limited activities
- Beyond: usage-based billing

**⚠️ Important:** Monitor costs, especially for transformation activities.

---

## Create a Data Factory

### Step 1: Access Data Factory

1. Azure Portal → Search "Data Factory"
2. Click on "Data factories"
3. Click on "Create"

### Step 2: Basic Configuration

**Basic Information:**
- **Subscription** : Choose your subscription
- **Resource group** : Create or use existing
- **Name** : `my-data-factory` (globally unique)
- **Version** : V2 (recommended)
- **Region** : Choose the closest region

**Git Configuration (optional):**
- **Configure Git later** : To start quickly
- Or configure Git/GitHub for versioning

### Step 3: Create the Data Factory

1. Click on "Review + create"
2. Verify configuration
3. Click on "Create"
4. Wait for creation (2-3 minutes)

**⚠️ Important:** Note the Data Factory name.

### Step 4: Open Data Factory Studio

1. Once created, click on "Open Azure Data Factory Studio"
2. Web interface to create pipelines

---

## Create a Pipeline

### Step 1: Create a Linked Service

**Linked Service = Connection to a data source**

**Example: Azure Blob Storage**

1. Data Factory Studio → "Manage" → "Linked services"
2. Click on "+ New"
3. Search "Azure Blob Storage"
4. Configuration:
   - **Name** : `AzureBlobStorage1`
   - **Storage account name** : Select your account
   - **Authentication method** : Account key (or other)
5. Click on "Create"

### Step 2: Create a Dataset

**Dataset = Data representation**

1. Data Factory Studio → "Author" → "Datasets"
2. Click on "+ New"
3. Choose "Azure Blob Storage"
4. Configuration:
   - **Name** : `CSVData`
   - **Linked service** : `AzureBlobStorage1`
   - **File path** : `raw-data/`
   - **File format** : DelimitedText (CSV)
5. Click on "Create"

### Step 3: Create a Pipeline

1. Data Factory Studio → "Author" → "Pipelines"
2. Click on "+ New pipeline"
3. Name the pipeline: `CopyCSVToParquet`

### Step 4: Add an Activity

**Example: Copy Data**

1. In the pipeline, drag "Copy Data" from "Move & transform"
2. Configure:
   - **Source** : Dataset `CSVData`
   - **Sink (Destination)** : Create a new Parquet dataset
3. Click on "Publish" to save

---

## Transformation Activities

### Copy Data

**Copy data from a source to a destination**

**Configuration:**
- **Source** : Source dataset
- **Sink** : Destination dataset
- **Mapping** : Column mapping

**Example: CSV → Parquet**

```json
{
  "name": "CopyCSVToParquet",
  "type": "Copy",
  "inputs": [{"referenceName": "CSVData"}],
  "outputs": [{"referenceName": "ParquetData"}],
  "typeProperties": {
    "source": {"type": "DelimitedTextSource"},
    "sink": {"type": "ParquetSink"}
  }
}
```

### Data Flow

**Data transformation with graphical interface**

**Steps:**
1. Create a Data Flow
2. Add a source
3. Add transformations:
   - **Select** : Select columns
   - **Filter** : Filter rows
   - **Derived Column** : Create calculated columns
   - **Aggregate** : Aggregations
   - **Join** : Join data
4. Add a sink

**Example transformations:**

```
Source (CSV) 
  → Select (columns)
  → Filter (status = 'active')
  → Derived Column (new column)
  → Aggregate (SUM, COUNT)
  → Sink (Parquet)
```

### Lookup

**Look up values in another source**

**Usage:**
- Validate data
- Enrich data
- Check references

### Stored Procedure

**Execute a SQL stored procedure**

**Usage:**
- Processing in SQL Database
- Complex business logic
- Database-side optimization

---

## Integration with Data Sources

### Azure Blob Storage

**Data source:**

```json
{
  "type": "AzureBlobStorage",
  "typeProperties": {
    "connectionString": "...",
    "container": "raw-data"
  }
}
```

### Azure SQL Database

**Data source:**

```json
{
  "type": "AzureSqlDatabase",
  "typeProperties": {
    "connectionString": "...",
    "tableName": "users"
  }
}
```

### Azure Data Lake Storage Gen2

**Data source:**

```json
{
  "type": "AzureBlobFS",
  "typeProperties": {
    "url": "https://account.dfs.core.windows.net",
    "fileSystem": "data-lake"
  }
}
```

### Local Files (via Self-hosted IR)

**Integration Runtime:**
- Self-hosted IR for local file access
- Install on a local machine
- Connect to Data Factory

---

## Orchestration and Scheduling

### Trigger Manually

1. Data Factory Studio → "Monitor"
2. Select the pipeline
3. Click on "Trigger now"
4. View execution in real-time

### Schedule a Pipeline (Trigger)

**Create a trigger:**

1. Pipeline → "Add trigger" → "New/Edit"
2. Type: "Schedule"
3. Configuration:
   - **Name** : `DailyTrigger`
   - **Type** : Schedule
   - **Recurrence** : Daily
   - **Start time** : 02:00
4. Click on "OK"

**Trigger types:**
- **Schedule** : Scheduled (cron)
- **Event** : Triggered by event
- **Tumbling window** : Sliding window

### Trigger by Event

**Example: New file in Blob Storage**

1. Create a "Storage event" trigger
2. Configure:
   - **Storage account** : Your account
   - **Container** : `raw-data`
   - **Event type** : Blob created
3. Associate with pipeline

---

## Best Practices

### Performance

1. **Use Data Flow** for complex transformations
2. **Optimize activities** to reduce time
3. **Use parallelism** when possible
4. **Choose the right regions** to reduce latency

### Costs

1. **Monitor executions** in Monitor
2. **Use the 5 free pipelines** wisely
3. **Optimize Data Flows** (expensive)
4. **Stop unused pipelines**

### Organization

1. **Name clearly** pipelines and activities
2. **Document** transformations
3. **Version** with Git
4. **Test** before publishing

### Security

1. **Use Key Vault** for secrets
2. **Limit permissions** of Linked Services
3. **Audit** executions
4. **Encrypt** data in transit

---

## Practical Examples

### Example 1: Simple CSV → Parquet Pipeline

**Pipeline:**
1. Source: Azure Blob Storage (CSV)
2. Activity: Copy Data
3. Sink: Azure Blob Storage (Parquet)

**Configuration:**
- Source: `raw-data/data.csv`
- Sink: `processed-data/data.parquet`
- Format: DelimitedText → Parquet

### Example 2: Pipeline with Transformation

**Pipeline:**
1. Source: Azure SQL Database
2. Data Flow:
   - Select columns
   - Filter rows
   - Aggregate
3. Sink: Azure Blob Storage (Parquet)

### Example 3: Orchestrated Pipeline

**Pipeline:**
1. Lookup: Check if new data
2. If Condition: If new data
3. Copy Data: Copy to staging
4. Data Flow: Transform
5. Copy Data: Load into destination

---

## 📊 Key Takeaways

1. **Data Factory = Cloud ETL** managed by Microsoft
2. **Free Tier: 5 pipelines** free
3. **Pipelines** orchestrate activities
4. **Data Flows** for complex transformations
5. **Triggers** enable automation

## 🔗 Next Module

Proceed to module [4. Azure SQL Database - Database](../04-sql-database/README.md) to learn how to use SQL Database on Azure.

