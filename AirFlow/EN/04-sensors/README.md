# 4. Airflow Sensors

## 🎯 Objectives

- Understand sensors
- Use FileSensor, SqlSensor, HttpSensor
- Create custom sensors
- Manage timeouts and retries

## 📋 Table of Contents

1. [Introduction to Sensors](#introduction-to-sensors)
2. [FileSensor](#filesensor)
3. [SqlSensor](#sqlsensor)
4. [HttpSensor](#httpsensor)
5. [Custom Sensors](#custom-sensors)

---

## Introduction to Sensors

### What is a Sensor?

**Sensor** = Task that waits for a condition

- **Polling** : Periodically checks a condition
- **Timeout** : Stops after a certain time
- **Poke interval** : Interval between checks
- **Mode** : Resume or poke

### Sensor Types

- **FileSensor** : Waits for a file
- **SqlSensor** : Waits for a SQL condition
- **HttpSensor** : Waits for an HTTP response
- **S3KeySensor** : Waits for an S3 key

---

## FileSensor

### Basic Usage

```python
from airflow.sensors.filesystem import FileSensor

task = FileSensor(
    task_id='wait_for_file',
    filepath='/path/to/file.csv',
    poke_interval=30,  # Check every 30 seconds
    timeout=3600,  # Timeout after 1 hour
    dag=dag,
)
```

### FileSensor with Wildcards

```python
task = FileSensor(
    task_id='wait_for_files',
    filepath='/path/to/data/*.csv',
    poke_interval=60,
    timeout=7200,
    dag=dag,
)
```

---

## SqlSensor

### Basic Usage

```python
from airflow.sensors.sql import SqlSensor

task = SqlSensor(
    task_id='wait_for_data',
    conn_id='my_postgres',
    sql="SELECT COUNT(*) FROM users WHERE status = 'active'",
    poke_interval=60,
    timeout=3600,
    dag=dag,
)
```

### SqlSensor with Condition

```python
task = SqlSensor(
    task_id='wait_for_count',
    conn_id='my_postgres',
    sql="SELECT COUNT(*) as count FROM orders WHERE date = '{{ ds }}'",
    poke_interval=30,
    timeout=1800,
    dag=dag,
)
```

---

## HttpSensor

### Basic Usage

```python
from airflow.sensors.http import HttpSensor

task = HttpSensor(
    task_id='wait_for_api',
    http_conn_id='my_http',
    endpoint='/api/status',
    method='GET',
    poke_interval=30,
    timeout=3600,
    dag=dag,
)
```

### HttpSensor with Response

```python
def check_response(response):
    return response.status_code == 200

task = HttpSensor(
    task_id='wait_for_api_ready',
    http_conn_id='my_http',
    endpoint='/api/health',
    response_check=check_response,
    poke_interval=60,
    timeout=7200,
    dag=dag,
)
```

---

## Custom Sensors

### Create a Custom Sensor

```python
from airflow.sensors.base import BaseSensorOperator
from airflow.utils.decorators import apply_defaults

class MyCustomSensor(BaseSensorOperator):
    @apply_defaults
    def __init__(self, my_param, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.my_param = my_param
    
    def poke(self, context):
        # Check condition
        # Return True if condition met
        # Return False to keep waiting
        return self.check_condition()
    
    def check_condition(self):
        # Your verification logic
        return True  # or False

# Usage
task = MyCustomSensor(
    task_id='custom_sensor',
    my_param='value',
    poke_interval=30,
    timeout=3600,
    dag=dag,
)
```

---

## 📊 Key Takeaways

1. **Sensors** wait for conditions
2. **Polling** checks periodically
3. **Timeout** limits wait time
4. **Poke interval** defines frequency
5. **Mode** controls behavior

## 🔗 Next Module

Proceed to module [5. Hooks](../05-hooks/README.md) to learn how to use hooks.

