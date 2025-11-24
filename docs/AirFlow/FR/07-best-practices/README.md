# 7. Bonnes pratiques Airflow

## 🎯 Objectifs

- Structurer les DAGs efficacement
- Gérer les erreurs
- Optimiser les performances
- Tester et déboguer

## 📋 Table des matières

1. [Structure des DAGs](#structure-des-dags)
2. [Gestion des erreurs](#gestion-des-erreurs)
3. [Performance](#performance)
4. [Tests](#tests)
5. [Débogage](#débogage)

---

## Structure des DAGs

### Organisation des fichiers

```
airflow/
├── dags/
│   ├── etl/
│   │   ├── __init__.py
│   │   ├── extract.py
│   │   ├── transform.py
│   │   └── load.py
│   └── analytics/
│       └── reports.py
├── plugins/
│   └── custom_operators.py
└── config/
    └── settings.py
```

### Bonnes pratiques de code

**1. Imports organisés :**

```python
# Standard library
from datetime import datetime, timedelta

# Third-party
from airflow import DAG
from airflow.operators.python import PythonOperator

# Local
from utils.helpers import process_data
```

**2. Fonctions réutilisables :**

```python
# utils/helpers.py
def extract_data(source):
    # Logique d'extraction
    return data

def transform_data(data):
    # Logique de transformation
    return transformed_data

# dags/etl_pipeline.py
from utils.helpers import extract_data, transform_data

extract_task = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
    op_args=['source'],
    dag=dag,
)
```

**3. Configuration centralisée :**

```python
# config/settings.py
DEFAULT_ARGS = {
    'owner': 'data_team',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

# dags/my_dag.py
from config.settings import DEFAULT_ARGS

dag = DAG('my_dag', default_args=DEFAULT_ARGS)
```

---

## Gestion des erreurs

### Retry et backoff

```python
default_args = {
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'max_retry_delay': timedelta(hours=1),
}
```

### Gestion d'exceptions

```python
def process_with_error_handling():
    try:
        # Code qui peut échouer
        result = risky_operation()
        return result
    except SpecificError as e:
        # Gérer l'erreur spécifique
        logger.error(f"Error: {e}")
        raise
    except Exception as e:
        # Gérer les autres erreurs
        logger.error(f"Unexpected error: {e}")
        raise
```

### Callbacks

```python
def on_failure_callback(context):
    logger.error("Task failed!")
    # Envoyer une alerte
    send_alert(context)

def on_success_callback(context):
    logger.info("Task succeeded!")

task = PythonOperator(
    task_id='task',
    python_callable=my_function,
    on_failure_callback=on_failure_callback,
    on_success_callback=on_success_callback,
    dag=dag,
)
```

---

## Performance

### Optimiser les DAGs

**1. Éviter les dépendances inutiles :**

```python
# Mauvais : dépendances séquentielles inutiles
task1 >> task2 >> task3 >> task4

# Bon : parallélisme quand possible
[task1, task2] >> task3 >> task4
```

**2. Utiliser le mode reschedule pour les sensors :**

```python
sensor = FileSensor(
    task_id='wait_file',
    filepath='/path/to/file',
    mode='reschedule',  # Libère le slot worker
    poke_interval=60,
    dag=dag,
)
```

**3. Limiter le parallélisme :**

```python
dag = DAG(
    'my_dag',
    max_active_runs=1,  # Un seul run à la fois
    max_active_tasks=10,  # Maximum 10 tâches en parallèle
)
```

---

## Tests

### Tests unitaires

```python
# tests/test_dag.py
import pytest
from airflow.models import DagBag

def test_dag_loaded():
    dagbag = DagBag()
    dag = dagbag.get_dag(dag_id='my_dag')
    assert dag is not None
    assert len(dag.tasks) == 3

def test_dag_structure():
    dagbag = DagBag()
    dag = dagbag.get_dag(dag_id='my_dag')
    assert 'extract' in dag.task_ids
    assert 'transform' in dag.task_ids
```

### Tests d'intégration

```python
from airflow.operators.python import PythonOperator

def test_task_execution():
    task = PythonOperator(
        task_id='test_task',
        python_callable=lambda: "success",
    )
    result = task.execute({})
    assert result == "success"
```

---

## Débogage

### Logs

```python
import logging

logger = logging.getLogger(__name__)

def my_function():
    logger.info("Starting process")
    logger.debug("Debug information")
    logger.error("Error occurred")
```

### Vérifier les logs

```bash
# Logs d'une tâche
airflow tasks logs my_dag my_task 2024-01-01

# Logs du scheduler
tail -f ~/airflow/logs/scheduler/*.log
```

---

## 📊 Points clés à retenir

1. **Structure** : Organiser le code proprement
2. **Erreurs** : Gérer avec retry et callbacks
3. **Performance** : Optimiser le parallélisme
4. **Tests** : Tester les DAGs
5. **Logs** : Utiliser le logging efficacement

## 🔗 Prochain module

Passer au module [8. Projets pratiques](../08-projets/README.md) pour créer des projets complets.

