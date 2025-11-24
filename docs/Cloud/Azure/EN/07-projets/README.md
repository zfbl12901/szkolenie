# 7. Azure Practical Projects

## 🎯 Objectives

- Apply acquired knowledge
- Create complete ETL pipelines
- Integrate with PowerBI
- Create projects for your portfolio
- Use multiple Azure services

## 📋 Table of Contents

1. [Project 1: ETL Pipeline Blob → SQL Database](#project-1--etl-pipeline-blob---sql-database)
2. [Project 2: Data Lake with Synapse](#project-2--data-lake-with-synapse)
3. [Project 3: Analytics with PowerBI](#project-3--analytics-with-powerbi)
4. [Project 4: Complete Automated Pipeline](#project-4--complete-automated-pipeline)
5. [Best Practices for Portfolio](#best-practices-for-portfolio)

---

## Project 1: ETL Pipeline Blob → SQL Database

### Objective

Create an ETL pipeline that loads CSV files from Blob Storage to SQL Database.

### Architecture

```
Blob Storage (CSV) → Data Factory → SQL Database → PowerBI
```

### Steps

#### 1. Prepare Data

**Create a Blob Storage container:**
- Name: `raw-data`
- Upload a test CSV file

**Example CSV data:**
```csv
id,name,email,created_at,status
1,John Doe,john@example.com,2024-01-01,active
2,Jane Smith,jane@example.com,2024-01-02,inactive
```

#### 2. Create a SQL Database

1. Azure Portal → Create SQL Database
2. Configuration:
   - Name: `analytics-db`
   - Server: Create new server
   - Service tier: Basic (free 12 months)
3. Create the database

#### 3. Create the Table in SQL Database

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at DATETIME2,
    status VARCHAR(20)
);
```

#### 4. Create a Data Factory Pipeline

1. Data Factory Studio → "Author" → "Pipelines"
2. Create a new pipeline: `LoadCSVToSQL`
3. Add "Copy Data" activity
4. Configuration:
   - **Source** : Azure Blob Storage (CSV)
   - **Sink** : Azure SQL Database (table users)
5. Publish the pipeline

#### 5. Execute the Pipeline

1. Click on "Trigger now"
2. Check execution in "Monitor"
3. Check data in SQL Database

### Result

- CSV files loaded into SQL Database
- Functional ETL pipeline
- Ready for analytics with PowerBI

---

## Project 2: Data Lake with Synapse

### Objective

Create a complete Data Lake with ingestion, transformation, and analytics.

### Architecture

```
Sources → Data Lake Storage (Raw) → Synapse (Transform) → Data Lake (Processed) → PowerBI
                ↓
        Data Factory (Orchestration)
```

### Steps

#### 1. Data Lake Storage Structure

```
data-lake/
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

#### 2. Create a Synapse Workspace

1. Azure Portal → Create Azure Synapse Analytics
2. Configuration:
   - Workspace name: `my-synapse-workspace`
   - Data Lake Storage: Create new
3. Create the workspace

#### 3. Data Factory Pipelines for Transformation

**Pipeline for users:**

1. Synapse Studio → "Integrate" → "Pipelines"
2. Create pipeline: `TransformUsers`
3. Activities:
   - Source: Data Lake Storage (raw/users/)
   - Data Flow: Transform (filter, clean)
   - Sink: Data Lake Storage (processed/users/)
4. Publish

#### 4. Synapse Tables for Analytics

```sql
-- Create an external table
CREATE EXTERNAL TABLE users_processed (
    id INT,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at DATETIME2
)
WITH (
    LOCATION = 'processed/users/',
    DATA_SOURCE = DataLakeStorage,
    FILE_FORMAT = ParquetFormat
);

-- Analytical query
SELECT 
    YEAR(created_at) AS year,
    COUNT(*) AS user_count
FROM users_processed
GROUP BY YEAR(created_at);
```

#### 5. Automation with Triggers

1. Pipeline → "Add trigger" → "New/Edit"
2. Type: Schedule
3. Recurrence: Daily
4. Start time: 02:00
5. Save

### Result

- Functional Data Lake
- Automated pipeline
- Analytics with Synapse
- Complete project for portfolio

---

## Project 3: Analytics with PowerBI

### Objective

Create a complete analytics system with PowerBI connected to Azure.

### Steps

#### 1. Prepare Data

**In SQL Database or Synapse:**
- Load data
- Create analytical views

#### 2. Connect PowerBI to Azure SQL Database

1. PowerBI Desktop → "Get Data"
2. "Azure" → "Azure SQL Database"
3. Configuration:
   - Server: `my-sql-server.database.windows.net`
   - Database: `analytics-db`
   - Authentication: Database
4. Select tables or views
5. Click on "Load"

#### 3. Create Visualizations

**Example:**
1. Import the `users` table
2. Create a chart: Number of users per month
3. Add filters
4. Create a dashboard

#### 4. Publish to PowerBI Service

1. PowerBI Desktop → "Publish"
2. Select the workspace
3. Publish
4. Access the report on powerbi.com

#### 5. Refresh Data

1. PowerBI Service → Dataset → "Schedule refresh"
2. Configuration:
   - Frequency: Daily
   - Time: 03:00
3. Save

### Result

- Analytics with PowerBI
- Interactive visualizations
- Automatic refresh
- Complete project for portfolio

---

## Project 4: Complete Automated Pipeline

### Objective

Create a completely automated ETL pipeline with multiple Azure services.

### Complete Architecture

```
CSV file uploaded → Blob Storage (raw/)
    ↓ (Event)
Azure Function (Validation)
    ↓
Blob Storage (validated/)
    ↓ (Trigger)
Data Factory Pipeline (Transform CSV → Parquet)
    ↓
Data Lake Storage (processed/)
    ↓
Synapse (Analytics)
    ↓
SQL Database (Results)
    ↓
PowerBI (Visualization)
```

### Implementation

#### 1. Azure Function for Validation

```python
import azure.functions as func
import logging
import csv
from azure.storage.blob import BlobServiceClient

def main(blob: func.InputStream):
    logging.info(f'Processing blob: {blob.name}')
    
    # Read the blob
    content = blob.read().decode('utf-8')
    reader = csv.DictReader(content.splitlines())
    
    # Validate
    valid_rows = []
    for row in reader:
        if row.get('email') and '@' in row['email']:
            valid_rows.append(row)
    
    # Upload validated data
    if valid_rows:
        # Upload to validated/
        # ...
    
    logging.info(f'Validated {len(valid_rows)} rows')
```

#### 2. Data Factory Transformation Pipeline

**Pipeline:**
1. Source: Blob Storage (validated/)
2. Data Flow: Transform (clean, enrich)
3. Sink: Data Lake Storage (processed/parquet/)

#### 3. Synapse for Analytics

```sql
-- Create an analytical view
CREATE VIEW vw_user_analytics AS
SELECT 
    u.id,
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
```

#### 4. PowerBI for Visualization

1. Connect PowerBI to Synapse
2. Use the view `vw_user_analytics`
3. Create visualizations
4. Publish the report

### Result

- Completely automated pipeline
- Automatic validation
- Automatic transformation
- Analytics available immediately
- PowerBI visualizations

---

## Best Practices for Portfolio

### Documentation

**Create a README for each project:**

```markdown
# Project: Azure ETL Pipeline

## Description
Automated ETL pipeline to transform CSV data to Parquet.

## Architecture
- Blob Storage: Storage
- Data Factory: Transformation
- SQL Database: Database
- PowerBI: Visualization

## Results
- 50% cost reduction
- 70% processing time reduction
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
- Data Factory scripts (JSON)
- SQL scripts
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

- [Azure Architecture Center](https://learn.microsoft.com/azure/architecture/)
- [Azure Solutions](https://azure.microsoft.com/solutions/)
- [GitHub Azure Examples](https://github.com/Azure-Samples)

---

**Congratulations!** You have completed the Azure training for Data Analyst. You can now create complete projects on Azure using the available free resources.

