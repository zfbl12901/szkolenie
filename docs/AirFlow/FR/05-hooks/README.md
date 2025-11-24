# 5. Hooks Airflow

## 🎯 Objectifs

- Comprendre les hooks
- Utiliser les hooks de base de données
- Utiliser les hooks cloud
- Créer des hooks personnalisés

## 📋 Table des matières

1. [Introduction aux hooks](#introduction-aux-hooks)
2. [Hooks de base de données](#hooks-de-base-de-données)
3. [Hooks cloud](#hooks-cloud)
4. [Hooks HTTP](#hooks-http)
5. [Hooks personnalisés](#hooks-personnalisés)

---

## Introduction aux hooks

### Qu'est-ce qu'un Hook ?

**Hook** = Interface pour interagir avec des systèmes externes

- **Réutilisable** : Peut être utilisé dans plusieurs tâches
- **Gère les connexions** : Utilise les connexions Airflow
- **Abstraction** : Cache les détails d'implémentation

### Types de hooks

- **Database hooks** : PostgreSQL, MySQL, SQLite
- **Cloud hooks** : AWS, Azure, GCP
- **HTTP hooks** : Requêtes HTTP
- **File hooks** : Opérations sur fichiers

---

## Hooks de base de données

### PostgresHook

```python
from airflow.hooks.postgres import PostgresHook

def query_database():
    hook = PostgresHook(postgres_conn_id='my_postgres')
    
    # Exécuter une requête
    records = hook.get_records("SELECT * FROM users LIMIT 10")
    
    # Exécuter une commande
    hook.run("INSERT INTO logs VALUES ('test')")
    
    # Obtenir pandas DataFrame
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

## Hooks cloud

### S3Hook (AWS)

```python
from airflow.providers.amazon.aws.hooks.s3 import S3Hook

def upload_to_s3():
    hook = S3Hook(aws_conn_id='my_aws')
    
    # Uploader un fichier
    hook.load_file(
        filename='/local/path/file.csv',
        key='s3/path/file.csv',
        bucket_name='my-bucket',
    )
    
    # Télécharger un fichier
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
    
    # Uploader un fichier
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

## Hooks HTTP

### HttpHook

```python
from airflow.providers.http.hooks.http import HttpHook

def call_api():
    hook = HttpHook(http_conn_id='my_api', method='GET')
    
    # Faire une requête GET
    response = hook.run(endpoint='/api/data')
    print(response.json())
    
    # Faire une requête POST
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

## Hooks personnalisés

### Créer un hook personnalisé

```python
from airflow.hooks.base import BaseHook

class MyCustomHook(BaseHook):
    def __init__(self, my_conn_id):
        super().__init__()
        self.conn_id = my_conn_id
        self.conn = self.get_connection(my_conn_id)
    
    def do_something(self):
        # Votre logique
        print(f"Connecting to {self.conn.host}")
        return "Success"

# Utilisation
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

## 📊 Points clés à retenir

1. **Hooks** abstraient les connexions
2. **Réutilisables** dans plusieurs tâches
3. **Gèrent les connexions** via Airflow
4. **Supportent** bases de données, cloud, HTTP
5. **Extensibles** avec hooks personnalisés

## 🔗 Prochain module

Passer au module [6. Variables et Connexions](../06-variables-connections/README.md) pour apprendre à gérer la configuration.

