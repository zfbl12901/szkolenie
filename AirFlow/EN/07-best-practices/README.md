# 7. Airflow Best Practices

## 🎯 Objectives

- Structure DAGs efficiently
- Handle errors
- Optimize performance
- Test and debug

## 📋 Table of Contents

1. [DAG Structure](#dag-structure)
2. [Error Handling](#error-handling)
3. [Performance](#performance)
4. [Testing](#testing)
5. [Debugging](#debugging)

---

## DAG Structure

### File Organization

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

### Code Best Practices

**1. Organized imports:**

```python
# Standard library
from datetime import datetime, timedelta

# Third-party
from airflow import DAG
from airflow.operators.python import PythonOperator

# Local
from utils.helpers import process_data
```

**2. Reusable functions:**

```python
# utils/helpers.py
def extract_data(source):
    # Extraction logic
    return data

def transform_data(data):
    # Transformation logic
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

**3. Centralized configuration:**

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

## Error Handling

### Retry and Backoff

```python
default_args = {
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'max_retry_delay': timedelta(hours=1),
}
```

### Exception Handling

```python
def process_with_error_handling():
    try:
        # Code that may fail
        result = risky_operation()
        return result
    except SpecificError as e:
        # Handle specific error
        logger.error(f"Error: {e}")
        raise
    except Exception as e:
        # Handle other errors
        logger.error(f"Unexpected error: {e}")
        raise
```

### Callbacks

```python
def on_failure_callback(context):
    logger.error("Task failed!")
    # Send an alert
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

### Optimize DAGs

**1. Avoid unnecessary dependencies:**

```python
# Bad: unnecessary sequential dependencies
task1 >> task2 >> task3 >> task4

# Good: parallelism when possible
[task1, task2] >> task3 >> task4
```

**2. Use reschedule mode for sensors:**

```python
sensor = FileSensor(
    task_id='wait_file',
    filepath='/path/to/file',
    mode='reschedule',  # Frees worker slot
    poke_interval=60,
    dag=dag,
)
```

**3. Limit parallelism:**

```python
dag = DAG(
    'my_dag',
    max_active_runs=1,  # Only one run at a time
    max_active_tasks=10,  # Maximum 10 tasks in parallel
)
```

---

## Testing

### Unit Tests

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

### Integration Tests

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

## Debugging

### Logs

```python
import logging

logger = logging.getLogger(__name__)

def my_function():
    logger.info("Starting process")
    logger.debug("Debug information")
    logger.error("Error occurred")
```

### Check Logs

```bash
# Task logs
airflow tasks logs my_dag my_task 2024-01-01

# Scheduler logs
tail -f ~/airflow/logs/scheduler/*.log
```

---

## 📊 Key Takeaways

1. **Structure** : Organize code cleanly
2. **Errors** : Handle with retry and callbacks
3. **Performance** : Optimize parallelism
4. **Tests** : Test DAGs
5. **Logs** : Use logging effectively

## 🔗 Next Module

Proceed to module [8. Practical Projects](../08-projets/README.md) to create complete projects.

