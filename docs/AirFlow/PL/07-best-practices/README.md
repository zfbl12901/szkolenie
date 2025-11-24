# 7. Dobre praktyki Airflow

## 🎯 Cele

- Strukturyzować DAGi efektywnie
- Obsługiwać błędy
- Optymalizować wydajność
- Testować i debugować

## 📋 Spis treści

1. [Struktura DAGów](#struktura-dagów)
2. [Obsługa błędów](#obsługa-błędów)
3. [Wydajność](#wydajność)
4. [Testy](#testy)
5. [Debugowanie](#debugowanie)

---

## Struktura DAGów

### Organizacja plików

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

### Dobre praktyki kodu

**1. Zorganizowane importy :**

```python
# Standardowa biblioteka
from datetime import datetime, timedelta

# Zewnętrzne
from airflow import DAG
from airflow.operators.python import PythonOperator

# Lokalne
from utils.helpers import process_data
```

**2. Funkcje reużywalne :**

```python
# utils/helpers.py
def extract_data(source):
    # Logika ekstrakcji
    return data

def transform_data(data):
    # Logika transformacji
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

**3. Konfiguracja scentralizowana :**

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

## Obsługa błędów

### Retry i backoff

```python
default_args = {
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'max_retry_delay': timedelta(hours=1),
}
```

### Obsługa wyjątków

```python
def process_with_error_handling():
    try:
        # Kod który może się nie powieść
        result = risky_operation()
        return result
    except SpecificError as e:
        # Obsłużyć błąd specyficzny
        logger.error(f"Error: {e}")
        raise
    except Exception as e:
        # Obsłużyć inne błędy
        logger.error(f"Unexpected error: {e}")
        raise
```

### Callbacki

```python
def on_failure_callback(context):
    logger.error("Task failed!")
    # Wysłać alert
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

## Wydajność

### Optymalizować DAGi

**1. Unikać niepotrzebnych zależności :**

```python
# Złe : niepotrzebne zależności sekwencyjne
task1 >> task2 >> task3 >> task4

# Dobre : równoległość gdy możliwe
[task1, task2] >> task3 >> task4
```

**2. Używać trybu reschedule dla sensorów :**

```python
sensor = FileSensor(
    task_id='wait_file',
    filepath='/path/to/file',
    mode='reschedule',  # Uwalnia slot worker
    poke_interval=60,
    dag=dag,
)
```

**3. Ograniczać równoległość :**

```python
dag = DAG(
    'my_dag',
    max_active_runs=1,  # Tylko jeden run na raz
    max_active_tasks=10,  # Maksimum 10 zadań równolegle
)
```

---

## Testy

### Testy jednostkowe

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

### Testy integracyjne

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

## Debugowanie

### Logi

```python
import logging

logger = logging.getLogger(__name__)

def my_function():
    logger.info("Starting process")
    logger.debug("Debug information")
    logger.error("Error occurred")
```

### Sprawdzać logi

```bash
# Logi zadania
airflow tasks logs my_dag my_task 2024-01-01

# Logi scheduler
tail -f ~/airflow/logs/scheduler/*.log
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Struktura** : Organizować kod czysto
2. **Błędy** : Obsługiwać z retry i callbackami
3. **Wydajność** : Optymalizować równoległość
4. **Testy** : Testować DAGi
5. **Logi** : Używać logowania efektywnie

## 🔗 Następny moduł

Przejdź do modułu [8. Projekty praktyczne](../08-projets/README.md), aby tworzyć kompletne projekty.

