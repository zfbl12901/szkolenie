# 8. Projets pratiques Airflow

## 🎯 Objectifs

- Créer un pipeline ETL complet
- Orchestrer des workflows complexes
- Intégrer avec bases de données
- Créer des projets pour portfolio

## 📋 Table des matières

1. [Projet 1 : Pipeline ETL simple](#projet-1--pipeline-etl-simple)
2. [Projet 2 : Orchestration de workflows](#projet-2--orchestration-de-workflows)
3. [Projet 3 : Intégration avec bases de données](#projet-3--intégration-avec-bases-de-données)
4. [Projet 4 : Pipeline complet](#projet-4--pipeline-complet)

---

## Projet 1 : Pipeline ETL simple

### Objectif

Créer un pipeline ETL qui extrait, transforme et charge des données.

### Architecture

```
Extract (API) → Transform (Python) → Load (Fichier)
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
    description='Pipeline ETL simple',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

def extract_data(**context):
    # Extraire des données depuis une API
    response = requests.get('https://api.example.com/data')
    data = response.json()
    
    # Sauvegarder temporairement
    df = pd.DataFrame(data)
    df.to_csv('/tmp/raw_data.csv', index=False)
    
    return '/tmp/raw_data.csv'

def transform_data(**context):
    # Récupérer le fichier de la tâche précédente
    ti = context['ti']
    input_file = ti.xcom_pull(task_ids='extract')
    
    # Lire et transformer
    df = pd.read_csv(input_file)
    df['processed_at'] = datetime.now()
    df = df.dropna()
    
    # Sauvegarder
    output_file = '/tmp/transformed_data.csv'
    df.to_csv(output_file, index=False)
    
    return output_file

def load_data(**context):
    # Récupérer le fichier transformé
    ti = context['ti']
    input_file = ti.xcom_pull(task_ids='transform')
    
    # Charger dans destination (ex: base de données)
    df = pd.read_csv(input_file)
    print(f"Loaded {len(df)} rows")
    
    # Ici, vous pourriez charger dans une base de données
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

## Projet 2 : Orchestration de workflows

### Objectif

Orchestrer plusieurs workflows avec dépendances complexes.

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
    description='Orchestration de workflows',
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

## Projet 3 : Intégration avec bases de données

### Objectif

Intégrer Airflow avec PostgreSQL pour un pipeline de données.

### Code

```python
from airflow import DAG
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG(
    'database_pipeline',
    description='Pipeline avec base de données',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

# Créer une table
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

# Insérer des données
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

# Requête analytique
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

## Projet 4 : Pipeline complet

### Objectif

Créer un pipeline ETL complet avec plusieurs sources et destinations.

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
    description='Pipeline ETL complet',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
    default_args={
        'retries': 2,
        'retry_delay': timedelta(minutes=5),
    },
)

# Attendre un fichier
wait_file = FileSensor(
    task_id='wait_file',
    filepath='/data/input/file.csv',
    poke_interval=60,
    timeout=3600,
    dag=dag,
)

# Extraire depuis API
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

# Transformer
def transform(**context):
    # Fichier API
    api_file = context['ti'].xcom_pull(task_ids='extract_api')
    df_api = pd.read_csv(api_file)
    
    # Fichier local
    df_file = pd.read_csv('/data/input/file.csv')
    
    # Combiner et transformer
    df_combined = pd.concat([df_api, df_file])
    df_combined = df_combined.dropna()
    df_combined['processed_at'] = datetime.now()
    
    # Sauvegarder
    df_combined.to_csv('/tmp/transformed.csv', index=False)
    return '/tmp/transformed.csv'

transform_task = PythonOperator(
    task_id='transform',
    python_callable=transform,
    dag=dag,
)

# Charger dans base de données
load_db = PostgresOperator(
    task_id='load_db',
    postgres_conn_id='my_postgres',
    sql='''
        COPY users FROM '/tmp/transformed.csv' 
        WITH (FORMAT csv, HEADER true);
    ''',
    dag=dag,
)

# Générer un rapport
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

# Dépendances
[wait_file, extract_api_task] >> transform_task >> load_db >> report_task
```

---

## 📊 Points clés à retenir

1. **ETL** : Extract, Transform, Load
2. **Orchestration** : Coordonner plusieurs workflows
3. **Intégration** : Bases de données, APIs, fichiers
4. **Monitoring** : Surveiller les exécutions
5. **Portfolio** : Créer des projets démontrables

## 🔗 Ressources

- [Exemples Airflow](https://github.com/apache/airflow/tree/main/airflow/example_dags)
- [Documentation Airflow](https://airflow.apache.org/docs/)

---

**Félicitations !** Vous avez terminé la formation Airflow. Vous pouvez maintenant créer des pipelines de données complexes avec Airflow.

