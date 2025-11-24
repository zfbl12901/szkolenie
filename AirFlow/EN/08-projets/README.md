# 8. Airflow Practical Projects

## 🎯 Objectives

- Create a complete ETL pipeline
- Orchestrate complex workflows
- Integrate with databases
- Create portfolio projects

## 📋 Table of Contents

1. [Project 1: Simple ETL Pipeline](#project-1-simple-etl-pipeline)
2. [Project 2: Workflow Orchestration](#project-2-workflow-orchestration)
3. [Project 3: Database Integration](#project-3-database-integration)
4. [Project 4: Complete Pipeline](#project-4-complete-pipeline)

---

## Project 1: Simple ETL Pipeline

### Objective

Create an ETL pipeline that extracts, transforms, and loads data.

### Architecture

```
Extract (API) → Transform (Python) → Load (File)
```

### Code

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta
import requests
import pandas as pd

default_args = {
    'owner': 'data_analyst',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'etl_pipeline',
    default_args=default_args,
    description='Simple ETL pipeline',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

def extract_data(**context):
    # Extract data from an API
    response = requests.get('https://api.example.com/data')
    data = response.json()
    
    # Save temporarily
    df = pd.DataFrame(data)
    df.to_csv('/tmp/raw_data.csv', index=False)
    
    return '/tmp/raw_data.csv'

def transform_data(**context):
    # Get file from previous task
    ti = context['ti']
    input_file = ti.xcom_pull(task_ids='extract')
    
    # Read and transform
    df = pd.read_csv(input_file)
    df['processed_at'] = datetime.now()
    df = df.dropna()
    
    # Save
    output_file = '/tmp/transformed_data.csv'
    df.to_csv(output_file, index=False)
    
    return output_file

def load_data(**context):
    # Get transformed file
    ti = context['ti']
    input_file = ti.xcom_pull(task_ids='transform')
    
    # Load into destination (e.g., database)
    df = pd.read_csv(input_file)
    print(f"Loaded {len(df)} rows")
    
    # Here, you could load into a database
    # df.to_sql('table', con=engine, if_exists='append')

extract = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
    dag=dag,
)

transform = PythonOperator(
    task_id='transform',
    python_callable=transform_data,
    dag=dag,
)

load = PythonOperator(
    task_id='load',
    python_callable=load_data,
    dag=dag,
)

extract >> transform >> load
```

---

## Project 2: Workflow Orchestration

### Objective

Orchestrate multiple workflows with complex dependencies.

### Architecture

```
Data Collection → Validation → Processing → Reporting
                      ↓
                  Alerting
```

### Code

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.branch import BranchPythonOperator
from datetime import datetime

dag = DAG(
    'workflow_orchestration',
    description='Workflow orchestration',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

def collect_data():
    print("Collecting data...")
    return "success"

def validate_data(**context):
    data = context['ti'].xcom_pull(task_ids='collect')
    if data == "success":
        return "valid"
    return "invalid"

def process_valid():
    print("Processing valid data...")

def process_invalid():
    print("Alerting: Invalid data!")

def generate_report():
    print("Generating report...")

collect = PythonOperator(
    task_id='collect',
    python_callable=collect_data,
    dag=dag,
)

validate = BranchPythonOperator(
    task_id='validate',
    python_callable=validate_data,
    dag=dag,
)

process_valid_task = PythonOperator(
    task_id='process_valid',
    python_callable=process_valid,
    dag=dag,
)

process_invalid_task = PythonOperator(
    task_id='process_invalid',
    python_callable=process_invalid,
    dag=dag,
)

report = PythonOperator(
    task_id='report',
    python_callable=generate_report,
    dag=dag,
)

collect >> validate >> [process_valid_task, process_invalid_task] >> report
```

---

## Project 3: Database Integration

### Objective

Integrate Airflow with PostgreSQL for a data pipeline.

### Code

```python
from airflow import DAG
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG(
    'database_pipeline',
    description='Pipeline with database',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

# Create a table
create_table = PostgresOperator(
    task_id='create_table',
    postgres_conn_id='my_postgres',
    sql='''
        CREATE TABLE IF NOT EXISTS users_processed (
            id SERIAL PRIMARY KEY,
            name VARCHAR(100),
            email VARCHAR(100),
            created_at TIMESTAMP
        );
    ''',
    dag=dag,
)

# Insert data
def insert_data():
    hook = PostgresHook(postgres_conn_id='my_postgres')
    hook.run("""
        INSERT INTO users_processed (name, email, created_at)
        VALUES ('John Doe', 'john@example.com', NOW());
    """)

insert = PythonOperator(
    task_id='insert',
    python_callable=insert_data,
    dag=dag,
)

# Analytical query
analytics_query = PostgresOperator(
    task_id='analytics',
    postgres_conn_id='my_postgres',
    sql='''
        SELECT 
            DATE(created_at) as date,
            COUNT(*) as user_count
        FROM users_processed
        GROUP BY DATE(created_at);
    ''',
    dag=dag,
)

create_table >> insert >> analytics_query
```

---

## Project 4: Complete Pipeline

### Objective

Create a complete ETL pipeline with multiple sources and destinations.

### Architecture

```
API → Validation → Transformation → Database → Reporting
  ↓
File → Validation → Transformation → Database
```

### Code

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.sensors.filesystem import FileSensor
from airflow.providers.postgres.operators.postgres import PostgresOperator
from datetime import datetime, timedelta
import pandas as pd
import requests

dag = DAG(
    'complete_pipeline',
    description='Complete ETL pipeline',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
    default_args={
        'retries': 2,
        'retry_delay': timedelta(minutes=5),
    },
)

# Wait for a file
wait_file = FileSensor(
    task_id='wait_file',
    filepath='/data/input/file.csv',
    poke_interval=60,
    timeout=3600,
    dag=dag,
)

# Extract from API
def extract_api():
    response = requests.get('https://api.example.com/data')
    data = response.json()
    df = pd.DataFrame(data)
    df.to_csv('/tmp/api_data.csv', index=False)
    return '/tmp/api_data.csv'

extract_api_task = PythonOperator(
    task_id='extract_api',
    python_callable=extract_api,
    dag=dag,
)

# Transform
def transform(**context):
    # API file
    api_file = context['ti'].xcom_pull(task_ids='extract_api')
    df_api = pd.read_csv(api_file)
    
    # Local file
    df_file = pd.read_csv('/data/input/file.csv')
    
    # Combine and transform
    df_combined = pd.concat([df_api, df_file])
    df_combined = df_combined.dropna()
    df_combined['processed_at'] = datetime.now()
    
    # Save
    df_combined.to_csv('/tmp/transformed.csv', index=False)
    return '/tmp/transformed.csv'

transform_task = PythonOperator(
    task_id='transform',
    python_callable=transform,
    dag=dag,
)

# Load into database
load_db = PostgresOperator(
    task_id='load_db',
    postgres_conn_id='my_postgres',
    sql='''
        COPY users FROM '/tmp/transformed.csv' 
        WITH (FORMAT csv, HEADER true);
    ''',
    dag=dag,
)

# Generate a report
def generate_report():
    hook = PostgresHook(postgres_conn_id='my_postgres')
    df = hook.get_pandas_df("SELECT * FROM users")
    report = df.describe()
    report.to_csv('/tmp/report.csv')
    print("Report generated!")

report_task = PythonOperator(
    task_id='report',
    python_callable=generate_report,
    dag=dag,
)

# Dependencies
[wait_file, extract_api_task] >> transform_task >> load_db >> report_task
```

---

## 📊 Key Takeaways

1. **ETL** : Extract, Transform, Load
2. **Orchestration** : Coordinate multiple workflows
3. **Integration** : Databases, APIs, files
4. **Monitoring** : Monitor executions
5. **Portfolio** : Create demonstrable projects

## 🔗 Resources

- [Airflow Examples](https://github.com/apache/airflow/tree/main/airflow/example_dags)
- [Airflow Documentation](https://airflow.apache.org/docs/)

---

**Congratulations!** You have completed the Airflow training. You can now create complex data pipelines with Airflow.

