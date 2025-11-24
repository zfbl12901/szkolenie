# 5. Hooki Airflow

## 🎯 Cele

- Zrozumieć hooki
- Używać hooków baz danych
- Używać hooków chmurowych
- Tworzyć hooki niestandardowe

## 📋 Spis treści

1. [Wprowadzenie do hooków](#wprowadzenie-do-hooków)
2. [Hooki baz danych](#hooki-baz-danych)
3. [Hooki chmurowe](#hooki-chmurowe)
4. [Hooki HTTP](#hooki-http)
5. [Hooki niestandardowe](#hooki-niestandardowe)

---

## Wprowadzenie do hooków

### Czym jest Hook?

**Hook** = Interfejs do interakcji z systemami zewnętrznymi

- **Reużywalny** : Może być używany w wielu zadaniach
- **Zarządza połączeniami** : Używa połączeń Airflow
- **Abstrakcja** : Ukrywa szczegóły implementacji

### Typy hooków

- **Hooki baz danych** : PostgreSQL, MySQL, SQLite
- **Hooki chmurowe** : AWS, Azure, GCP
- **Hooki HTTP** : Żądania HTTP
- **Hooki plików** : Operacje na plikach

---

## Hooki baz danych

### PostgresHook

```python
from airflow.hooks.postgres import PostgresHook

def query_database():
    hook = PostgresHook(postgres_conn_id='my_postgres')
    
    # Wykonać zapytanie
    records = hook.get_records("SELECT * FROM users LIMIT 10")
    
    # Wykonać polecenie
    hook.run("INSERT INTO logs VALUES ('test')")
    
    # Pobrać pandas DataFrame
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

## Hooki chmurowe

### S3Hook (AWS)

```python
from airflow.providers.amazon.aws.hooks.s3 import S3Hook

def upload_to_s3():
    hook = S3Hook(aws_conn_id='my_aws')
    
    # Przesłać plik
    hook.load_file(
        filename='/local/path/file.csv',
        key='s3/path/file.csv',
        bucket_name='my-bucket',
    )
    
    # Pobrać plik
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
    
    # Przesłać plik
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

## Hooki HTTP

### HttpHook

```python
from airflow.providers.http.hooks.http import HttpHook

def call_api():
    hook = HttpHook(http_conn_id='my_api', method='GET')
    
    # Wykonać żądanie GET
    response = hook.run(endpoint='/api/data')
    print(response.json())
    
    # Wykonać żądanie POST
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

## Hooki niestandardowe

### Utworzyć hook niestandardowy

```python
from airflow.hooks.base import BaseHook

class MyCustomHook(BaseHook):
    def __init__(self, my_conn_id):
        super().__init__()
        self.conn_id = my_conn_id
        self.conn = self.get_connection(my_conn_id)
    
    def do_something(self):
        # Twoja logika
        print(f"Connecting to {self.conn.host}")
        return "Success"

# Użycie
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

## 📊 Kluczowe punkty do zapamiętania

1. **Hooki** abstrahują połączenia
2. **Reużywalne** w wielu zadaniach
3. **Zarządzają połączeniami** przez Airflow
4. **Wspierają** bazy danych, chmurę, HTTP
5. **Rozszerzalne** z hookami niestandardowymi

## 🔗 Następny moduł

Przejdź do modułu [6. Zmienne i Połączenia](../06-variables-connections/README.md), aby nauczyć się zarządzać konfiguracją.

