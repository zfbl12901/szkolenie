# 8. Projekty praktyczne Airflow

## 🎯 Cele

- Tworzyć kompletny pipeline ETL
- Orkiestrować złożone workflows
- Integrować z bazami danych
- Tworzyć projekty do portfolio

## 📋 Spis treści

1. [Projekt 1 : Prosty pipeline ETL](#projekt-1--prosty-pipeline-etl)
2. [Projekt 2 : Orkiestracja workflows](#projekt-2--orkiestracja-workflows)
3. [Projekt 3 : Integracja z bazami danych](#projekt-3--integracja-z-bazami-danych)
4. [Projekt 4 : Kompletny pipeline](#projekt-4--kompletny-pipeline)

---

## Projekt 1 : Prosty pipeline ETL

### Cel

Utworzyć pipeline ETL który ekstrahuje, przekształca i ładuje dane.

### Architektura

```
Extract (API) → Transform (Python) → Load (Plik)
```

### Kod

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
    description='Prosty pipeline ETL',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

def extract_data(**context):
    # Ekstrahować dane z API
    response = requests.get('https://api.example.com/data')
    data = response.json()
    
    # Zapisać tymczasowo
    df = pd.DataFrame(data)
    df.to_csv('/tmp/raw_data.csv', index=False)
    
    return '/tmp/raw_data.csv'

def transform_data(**context):
    # Pobrać plik z poprzedniego zadania
    ti = context['ti']
    input_file = ti.xcom_pull(task_ids='extract')
    
    # Czytać i przekształcać
    df = pd.read_csv(input_file)
    df['processed_at'] = datetime.now()
    df = df.dropna()
    
    # Zapisać
    output_file = '/tmp/transformed_data.csv'
    df.to_csv(output_file, index=False)
    
    return output_file

def load_data(**context):
    # Pobrać przekształcony plik
    ti = context['ti']
    input_file = ti.xcom_pull(task_ids='transform')
    
    # Załadować do miejsca docelowego (np. baza danych)
    df = pd.read_csv(input_file)
    print(f"Loaded {len(df)} rows")
    
    # Tutaj, możesz załadować do bazy danych
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

## Projekt 2 : Orkiestracja workflows

### Cel

Orkiestrować wiele workflows z złożonymi zależnościami.

### Architektura

```
Zbieranie danych → Walidacja → Przetwarzanie → Raportowanie
                      ↓
                  Alertowanie
```

### Kod

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.branch import BranchPythonOperator
from datetime import datetime

dag = DAG(
    'workflow_orchestration',
    description='Orkiestracja workflows',
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

## Projekt 3 : Integracja z bazami danych

### Cel

Integrować Airflow z PostgreSQL dla pipeline'u danych.

### Kod

```python
from airflow import DAG
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG(
    'database_pipeline',
    description='Pipeline z bazą danych',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

# Utworzyć tabelę
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

# Wstawić dane
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

# Zapytanie analityczne
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

## Projekt 4 : Kompletny pipeline

### Cel

Utworzyć kompletny pipeline ETL z wieloma źródłami i miejscami docelowymi.

### Architektura

```
API → Walidacja → Przekształcanie → Baza danych → Raportowanie
  ↓
Plik → Walidacja → Przekształcanie → Baza danych
```

### Kod

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
    description='Kompletny pipeline ETL',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
    default_args={
        'retries': 2,
        'retry_delay': timedelta(minutes=5),
    },
)

# Czekać na plik
wait_file = FileSensor(
    task_id='wait_file',
    filepath='/data/input/file.csv',
    poke_interval=60,
    timeout=3600,
    dag=dag,
)

# Ekstrahować z API
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

# Przekształcić
def transform(**context):
    # Plik API
    api_file = context['ti'].xcom_pull(task_ids='extract_api')
    df_api = pd.read_csv(api_file)
    
    # Plik lokalny
    df_file = pd.read_csv('/data/input/file.csv')
    
    # Połączyć i przekształcić
    df_combined = pd.concat([df_api, df_file])
    df_combined = df_combined.dropna()
    df_combined['processed_at'] = datetime.now()
    
    # Zapisać
    df_combined.to_csv('/tmp/transformed.csv', index=False)
    return '/tmp/transformed.csv'

transform_task = PythonOperator(
    task_id='transform',
    python_callable=transform,
    dag=dag,
)

# Załadować do bazy danych
load_db = PostgresOperator(
    task_id='load_db',
    postgres_conn_id='my_postgres',
    sql='''
        COPY users FROM '/tmp/transformed.csv' 
        WITH (FORMAT csv, HEADER true);
    ''',
    dag=dag,
)

# Wygenerować raport
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

# Zależności
[wait_file, extract_api_task] >> transform_task >> load_db >> report_task
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **ETL** : Extract, Transform, Load
2. **Orkiestracja** : Koordynować wiele workflows
3. **Integracja** : Bazy danych, API, pliki
4. **Monitorowanie** : Monitorować wykonania
5. **Portfolio** : Tworzyć projekty demonstrowalne

## 🔗 Zasoby

- [Przykłady Airflow](https://github.com/apache/airflow/tree/main/airflow/example_dags)
- [Dokumentacja Airflow](https://airflow.apache.org/docs/)

---

**Gratulacje !** Ukończyłeś szkolenie Airflow. Możesz teraz tworzyć złożone pipeline'y danych z Airflow.

