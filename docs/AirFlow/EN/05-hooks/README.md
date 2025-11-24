# 5. Airflow Hooks

## 🎯 Objectives

- Understand hooks
- Use database hooks
- Use cloud hooks
- Create custom hooks

## 📋 Table of Contents

1. [Introduction to Hooks](#introduction-to-hooks)
2. [Database Hooks](#database-hooks)
3. [Cloud Hooks](#cloud-hooks)
4. [HTTP Hooks](#http-hooks)
5. [Custom Hooks](#custom-hooks)

---

## Introduction to Hooks

### What is a Hook?

**Hook** = Interface to interact with external systems

- **Reusable** : Can be used in multiple tasks
- **Manages connections** : Uses Airflow connections
- **Abstraction** : Hides implementation details

### Hook Types

- **Database hooks** : PostgreSQL, MySQL, SQLite
- **Cloud hooks** : AWS, Azure, GCP
- **HTTP hooks** : HTTP requests
- **File hooks** : File operations

---

## Database Hooks

### PostgresHook

```python
from airflow.hooks.postgres import PostgresHook

def query_database():
    hook = PostgresHook(postgres_conn_id='my_postgres')
    
    # Execute a query
    records = hook.get_records("SELECT * FROM users LIMIT 10")
    
    # Execute a command
    hook.run("INSERT INTO logs VALUES ('test')")
    
    # Get pandas DataFrame
    df = hook.get_pandas_df("SELECT * FROM users")

task = PythonOperator(
    task_id='query_db',
    python_callable=query_database,
    dag=dag,
)
```

### MySqlHook

```python
from airflow.providers.mysql.hooks.mysql import MySqlHook

def query_mysql():
    hook = MySqlHook(mysql_conn_id='my_mysql')
    records = hook.get_records("SELECT * FROM orders")

task = PythonOperator(
    task_id='query_mysql',
    python_callable=query_mysql,
    dag=dag,
)
```

---

## Cloud Hooks

### S3Hook (AWS)

```python
from airflow.providers.amazon.aws.hooks.s3 import S3Hook

def upload_to_s3():
    hook = S3Hook(aws_conn_id='my_aws')
    
    # Upload a file
    hook.load_file(
        filename='/local/path/file.csv',
        key='s3/path/file.csv',
        bucket_name='my-bucket',
    )
    
    # Download a file
    hook.download_file(
        key='s3/path/file.csv',
        bucket_name='my-bucket',
        local_path='/local/path/file.csv',
    )

task = PythonOperator(
    task_id='s3_upload',
    python_callable=upload_to_s3,
    dag=dag,
)
```

### Azure Blob Storage Hook

```python
from airflow.providers.microsoft.azure.hooks.wasb import WasbHook

def upload_to_azure():
    hook = WasbHook(wasb_conn_id='my_azure')
    
    # Upload a file
    hook.upload(
        container_name='my-container',
        blob_name='file.csv',
        file_path='/local/path/file.csv',
    )

task = PythonOperator(
    task_id='azure_upload',
    python_callable=upload_to_azure,
    dag=dag,
)
```

---

## HTTP Hooks

### HttpHook

```python
from airflow.providers.http.hooks.http import HttpHook

def call_api():
    hook = HttpHook(http_conn_id='my_api', method='GET')
    
    # Make a GET request
    response = hook.run(endpoint='/api/data')
    print(response.json())
    
    # Make a POST request
    hook = HttpHook(http_conn_id='my_api', method='POST')
    response = hook.run(
        endpoint='/api/data',
        data={'key': 'value'},
    )

task = PythonOperator(
    task_id='call_api',
    python_callable=call_api,
    dag=dag,
)
```

---

## Custom Hooks

### Create a Custom Hook

```python
from airflow.hooks.base import BaseHook

class MyCustomHook(BaseHook):
    def __init__(self, my_conn_id):
        super().__init__()
        self.conn_id = my_conn_id
        self.conn = self.get_connection(my_conn_id)
    
    def do_something(self):
        # Your logic
        print(f"Connecting to {self.conn.host}")
        return "Success"

# Usage
def use_custom_hook():
    hook = MyCustomHook(my_conn_id='my_connection')
    result = hook.do_something()

task = PythonOperator(
    task_id='use_hook',
    python_callable=use_custom_hook,
    dag=dag,
)
```

---

## 📊 Key Takeaways

1. **Hooks** abstract connections
2. **Reusable** in multiple tasks
3. **Manage connections** via Airflow
4. **Support** databases, cloud, HTTP
5. **Extensible** with custom hooks

## 🔗 Next Module

Proceed to module [6. Variables and Connections](../06-variables-connections/README.md) to learn how to manage configuration.

