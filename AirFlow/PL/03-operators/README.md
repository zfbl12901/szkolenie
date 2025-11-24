# 3. Operatory Airflow

## 🎯 Cele

- Zrozumieć różne typy operatorów
- Używać operatorów Python, Bash, SQL
- Tworzyć operatory niestandardowe
- Zarządzać danymi między zadaniami

## 📋 Spis treści

1. [Typy operatorów](#typy-operatorów)
2. [PythonOperator](#pythonoperator)
3. [BashOperator](#bashoperator)
4. [Operatory SQL](#operatory-sql)
5. [Operatory niestandardowe](#operatory-niestandardowe)

---

## Typy operatorów

### Operatory podstawowe

- **PythonOperator** : Wykonuje kod Python
- **BashOperator** : Wykonuje polecenia bash
- **SQLExecuteQueryOperator** : Wykonuje zapytania SQL
- **EmailOperator** : Wysyła emaile
- **HttpOperator** : Wykonuje żądania HTTP

### Operatory transferu

- **FileTransferOperator** : Transferuje pliki
- **FTPOperator** : Operacje FTP
- **S3FileTransformOperator** : Przekształca pliki S3

---

## PythonOperator

### Podstawowe użycie

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

### Przekazywać argumenty

```python
def process_data(file_path, output_path):
    print(f"Processing {file_path} -> {output_path}")
    # Przetwarzanie...

task = PythonOperator(
    task_id='process',
    python_callable=process_data,
    op_args=['/path/to/input.csv', '/path/to/output.csv'],
    dag=dag,
)
```

### Używać XCom do dzielenia danych

```python
def extract_data(**context):
    data = {'key': 'value'}
    return data  # Automatycznie zwraca przez XCom

def process_data(**context):
    # Pobrać dane z poprzedniego zadania
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

### Podstawowe użycie

```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='bash_task',
    bash_command='echo "Hello from Bash"',
    dag=dag,
)
```

### Używać szablonów

```python
task = BashOperator(
    task_id='bash_template',
    bash_command='echo "Date: {{ ds }}"',  # ds = data wykonania
    dag=dag,
)
```

### Dostępne zmienne szablonu

- `{{ ds }}` : Data wykonania (YYYY-MM-DD)
- `{{ ds_nodash }}` : Data bez myślników (YYYYMMDD)
- `{{ ts }}` : Timestamp wykonania
- `{{ dag }}` : Obiekt DAG
- `{{ task }}` : Obiekt Task

---

## Operatory SQL

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

### Używać szablonów SQL

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

## Operatory niestandardowe

### Utworzyć operator niestandardowy

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
        # Twoja logika tutaj
        return "Success"

# Użycie
task = MyCustomOperator(
    task_id='custom_task',
    my_param='value',
    dag=dag,
)
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **PythonOperator** dla logiki Python
2. **BashOperator** dla poleceń shell
3. **Operatory SQL** dla zapytań SQL
4. **XCom** do dzielenia danych
5. **Szablony** dla wartości dynamicznych

## 🔗 Następny moduł

Przejdź do modułu [4. Sensory](../04-sensors/README.md), aby nauczyć się używać sensorów.

