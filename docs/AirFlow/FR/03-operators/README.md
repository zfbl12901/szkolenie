# 3. Opérateurs Airflow

## 🎯 Objectifs

- Comprendre les différents types d'opérateurs
- Utiliser les opérateurs Python, Bash, SQL
- Créer des opérateurs personnalisés
- Gérer les données entre tâches

## 📋 Table des matières

1. [Types d'opérateurs](#types-dopérateurs)
2. [PythonOperator](#pythonoperator)
3. [BashOperator](#bashoperator)
4. [SQL Operators](#sql-operators)
5. [Opérateurs personnalisés](#opérateurs-personnalisés)

---

## Types d'opérateurs

### Opérateurs de base

- **PythonOperator** : Exécute du code Python
- **BashOperator** : Exécute des commandes bash
- **SQLExecuteQueryOperator** : Exécute des requêtes SQL
- **EmailOperator** : Envoie des emails
- **HttpOperator** : Fait des requêtes HTTP

### Opérateurs de transfert

- **FileTransferOperator** : Transfère des fichiers
- **FTPOperator** : Opérations FTP
- **S3FileTransformOperator** : Transforme des fichiers S3

---

## PythonOperator

### Utilisation de base

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG('python_operator', start_date=datetime(2024, 1, 1))

def my_function():
    print("Hello from Python!")
    return "Success"

task = PythonOperator(
    task_id='python_task',
    python_callable=my_function,
    dag=dag,
)
```

### Passer des arguments

```python
def process_data(file_path, output_path):
    print(f"Processing {file_path} -> {output_path}")
    # Traitement...

task = PythonOperator(
    task_id='process',
    python_callable=process_data,
    op_args=['/path/to/input.csv', '/path/to/output.csv'],
    dag=dag,
)
```

### Utiliser XCom pour partager des données

```python
def extract_data(**context):
    data = {'key': 'value'}
    return data  # Retourne automatiquement via XCom

def process_data(**context):
    # Récupérer les données de la tâche précédente
    ti = context['ti']
    data = ti.xcom_pull(task_ids='extract')
    print(f"Received: {data}")

extract = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
    dag=dag,
)

process = PythonOperator(
    task_id='process',
    python_callable=process_data,
    dag=dag,
)

extract >> process
```

---

## BashOperator

### Utilisation de base

```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='bash_task',
    bash_command='echo "Hello from Bash"',
    dag=dag,
)
```

### Utiliser des templates

```python
task = BashOperator(
    task_id='bash_template',
    bash_command='echo "Date: {{ ds }}"',  # ds = execution date
    dag=dag,
)
```

### Variables de template disponibles

- `{{ ds }}` : Date d'exécution (YYYY-MM-DD)
- `{{ ds_nodash }}` : Date sans tirets (YYYYMMDD)
- `{{ ts }}` : Timestamp d'exécution
- `{{ dag }}` : Objet DAG
- `{{ task }}` : Objet Task

---

## SQL Operators

### SQLExecuteQueryOperator

```python
from airflow.providers.postgres.operators.postgres import PostgresOperator

task = PostgresOperator(
    task_id='sql_task',
    postgres_conn_id='my_postgres',
    sql='SELECT * FROM users LIMIT 10;',
    dag=dag,
)
```

### Utiliser des templates SQL

```python
task = PostgresOperator(
    task_id='sql_template',
    postgres_conn_id='my_postgres',
    sql='''
        SELECT * FROM users
        WHERE created_at >= '{{ ds }}'
    ''',
    dag=dag,
)
```

---

## Opérateurs personnalisés

### Créer un opérateur personnalisé

```python
from airflow.models import BaseOperator
from airflow.utils.decorators import apply_defaults

class MyCustomOperator(BaseOperator):
    @apply_defaults
    def __init__(self, my_param, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.my_param = my_param
    
    def execute(self, context):
        print(f"Executing with param: {self.my_param}")
        # Votre logique ici
        return "Success"

# Utilisation
task = MyCustomOperator(
    task_id='custom_task',
    my_param='value',
    dag=dag,
)
```

---

## 📊 Points clés à retenir

1. **PythonOperator** pour logique Python
2. **BashOperator** pour commandes shell
3. **SQL Operators** pour requêtes SQL
4. **XCom** pour partager des données
5. **Templates** pour valeurs dynamiques

## 🔗 Prochain module

Passer au module [4. Sensors](../04-sensors/README.md) pour apprendre à utiliser les sensors.

