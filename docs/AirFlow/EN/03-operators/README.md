# 3. Airflow Operators

## 🎯 Objectives

- Understand different operator types
- Use Python, Bash, SQL operators
- Create custom operators
- Manage data between tasks

## 📋 Table of Contents

1. [Operator Types](#operator-types)
2. [PythonOperator](#pythonoperator)
3. [BashOperator](#bashoperator)
4. [SQL Operators](#sql-operators)
5. [Custom Operators](#custom-operators)

---

## Operator Types

### Basic Operators

- **PythonOperator** : Executes Python code
- **BashOperator** : Executes bash commands
- **SQLExecuteQueryOperator** : Executes SQL queries
- **EmailOperator** : Sends emails
- **HttpOperator** : Makes HTTP requests

### Transfer Operators

- **FileTransferOperator** : Transfers files
- **FTPOperator** : FTP operations
- **S3FileTransformOperator** : Transforms S3 files

---

## PythonOperator

### Basic Usage

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

### Pass Arguments

```python
def process_data(file_path, output_path):
    print(f"Processing {file_path} -> {output_path}")
    # Processing...

task = PythonOperator(
    task_id='process',
    python_callable=process_data,
    op_args=['/path/to/input.csv', '/path/to/output.csv'],
    dag=dag,
)
```

### Use XCom to Share Data

```python
def extract_data(**context):
    data = {'key': 'value'}
    return data  # Automatically returns via XCom

def process_data(**context):
    # Get data from previous task
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

### Basic Usage

```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='bash_task',
    bash_command='echo "Hello from Bash"',
    dag=dag,
)
```

### Use Templates

```python
task = BashOperator(
    task_id='bash_template',
    bash_command='echo "Date: {{ ds }}"',  # ds = execution date
    dag=dag,
)
```

### Available Template Variables

- `{{ ds }}` : Execution date (YYYY-MM-DD)
- `{{ ds_nodash }}` : Date without dashes (YYYYMMDD)
- `{{ ts }}` : Execution timestamp
- `{{ dag }}` : DAG object
- `{{ task }}` : Task object

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

### Use SQL Templates

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

## Custom Operators

### Create a Custom Operator

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
        # Your logic here
        return "Success"

# Usage
task = MyCustomOperator(
    task_id='custom_task',
    my_param='value',
    dag=dag,
)
```

---

## 📊 Key Takeaways

1. **PythonOperator** for Python logic
2. **BashOperator** for shell commands
3. **SQL Operators** for SQL queries
4. **XCom** to share data
5. **Templates** for dynamic values

## 🔗 Next Module

Proceed to module [4. Sensors](../04-sensors/README.md) to learn how to use sensors.

