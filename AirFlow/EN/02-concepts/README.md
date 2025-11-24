# 2. Airflow Fundamental Concepts

## 🎯 Objectives

- Understand DAGs (Directed Acyclic Graphs)
- Master tasks and dependencies
- Understand scheduling
- Use variables and connections
- Manage executions

## 📋 Table of Contents

1. [DAGs (Directed Acyclic Graphs)](#dags-directed-acyclic-graphs)
2. [Tasks and Dependencies](#tasks-and-dependencies)
3. [Scheduling](#scheduling)
4. [Variables and Connections](#variables-and-connections)
5. [Executions and States](#executions-and-states)

---

## DAGs (Directed Acyclic Graphs)

### What is a DAG?

**DAG** = Directed Acyclic Graph

- **Directed** : Tasks have direction (dependencies)
- **Acyclic** : No loops (no circular dependencies)
- **Graph** : Visual representation of workflows

### DAG Structure

```python
from airflow import DAG
from datetime import datetime, timedelta

# Default arguments
default_args = {
    'owner': 'data_analyst',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Create the DAG
dag = DAG(
    'my_dag',
    default_args=default_args,
    description='DAG description',
    schedule_interval='@daily',  # Execution frequency
    start_date=datetime(2024, 1, 1),
    catchup=False,  # Don't execute past runs
    tags=['example'],
)
```

### DAG Properties

**Unique ID:**
- Must be unique in Airflow installation
- Used to identify the DAG

**Schedule interval:**
- `@daily` : Every day
- `@hourly` : Every hour
- `timedelta(days=1)` : Every day
- `'0 2 * * *'` : Cron expression (every day at 2am)
- `None` : Manual trigger only

**Start date:**
- Start date of scheduling
- Format: `datetime(year, month, day)`

**Catchup:**
- `True` : Executes missing runs since start_date
- `False` : Executes only future runs

---

## Tasks and Dependencies

### What is a Task?

**Task** = Individual step in a DAG

- **Operator** : Task type (Python, Bash, SQL, etc.)
- **Unique ID** : Identifier in the DAG
- **Dependencies** : Relationships with other tasks

### Operator Types

**BashOperator:**
```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='bash_task',
    bash_command='echo "Hello"',
    dag=dag,
)
```

**PythonOperator:**
```python
from airflow.operators.python import PythonOperator

def my_function():
    print("Hello from Python")

task = PythonOperator(
    task_id='python_task',
    python_callable=my_function,
    dag=dag,
)
```

### Define Dependencies

**Method 1: Operator >>**

```python
# t1 runs before t2
t1 >> t2

# Multiple dependencies
t1 >> [t2, t3] >> t4
```

**Method 2: set_upstream / set_downstream**

```python
# t1 runs before t2
t1.set_downstream(t2)
# or
t2.set_upstream(t1)
```

**Method 3: bitshift**

```python
# t1 >> t2 is equivalent to
t1 >> t2
```

### Dependency Example

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

dag = DAG('dependencies_example', start_date=datetime(2024, 1, 1))

# Tasks
extract = BashOperator(task_id='extract', bash_command='echo extract', dag=dag)
transform = BashOperator(task_id='transform', bash_command='echo transform', dag=dag)
load = BashOperator(task_id='load', bash_command='echo load', dag=dag)
validate = BashOperator(task_id='validate', bash_command='echo validate', dag=dag)

# Dependencies
extract >> transform >> [load, validate]
```

---

## Scheduling

### Schedule Interval

**Common expressions:**

```python
# Every day at midnight
schedule_interval='@daily'
# or
schedule_interval=timedelta(days=1)

# Every hour
schedule_interval='@hourly'
# or
schedule_interval=timedelta(hours=1)

# Every week
schedule_interval='@weekly'

# Cron expression
schedule_interval='0 2 * * *'  # Every day at 2am
schedule_interval='0 */6 * * *'  # Every 6 hours
schedule_interval='0 0 * * MON'  # Every Monday at midnight
```

### Start Date and Execution Date

**Start Date:**
- Start date of scheduling
- Format: `datetime(2024, 1, 1)`

**Execution Date:**
- Logical execution date
- Format: `YYYY-MM-DDTHH:MM:SS`

**Example:**
```python
dag = DAG(
    'scheduled_dag',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)
```

### Catchup

**Catchup = True:**
- Executes all missing runs since start_date
- Can create many runs

**Catchup = False:**
- Executes only future runs
- Recommended for most cases

---

## Variables and Connections

### Variables

**Variables = Global configuration**

**Create a variable:**

```bash
# Via CLI
airflow variables set my_key "my_value"

# Via web interface
# Admin → Variables → Add
```

**Use a variable:**

```python
from airflow.models import Variable

# Get a variable
my_value = Variable.get("my_key")
my_value_default = Variable.get("my_key", default_var="default")

# In a template
# {{ var.value.my_key }}
```

**Example:**

```python
from airflow import DAG
from airflow.models import Variable
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG('variables_example', start_date=datetime(2024, 1, 1))

def use_variable():
    api_key = Variable.get("api_key")
    print(f"API Key: {api_key}")

task = PythonOperator(
    task_id='use_variable',
    python_callable=use_variable,
    dag=dag,
)
```

### Connections

**Connections = Connection information**

**Create a connection:**

```bash
# Via CLI
airflow connections add 'my_postgres' \
    --conn-type 'postgres' \
    --conn-host 'localhost' \
    --conn-login 'user' \
    --conn-password 'password' \
    --conn-port 5432 \
    --conn-schema 'mydb'
```

**Use a connection:**

```python
from airflow.hooks.base import BaseHook

# Get a connection
conn = BaseHook.get_connection('my_postgres')
print(f"Host: {conn.host}")
print(f"Login: {conn.login}")
print(f"Password: {conn.password}")
```

---

## Executions and States

### Task States

- **None** : Not yet executed
- **Scheduled** : Scheduled
- **Queued** : Waiting
- **Running** : Running
- **Success** : Succeeded
- **Failed** : Failed
- **Skipped** : Skipped
- **Retry** : Retrying
- **Up for retry** : Ready for retry

### DAG States

- **Running** : Running
- **Success** : All tasks succeeded
- **Failed** : At least one task failed

### Manage Executions

**Via web interface:**
- View execution state
- Rerun a task
- Mark as success/failure
- View logs

**Via CLI:**

```bash
# List runs
airflow dags list-runs -d my_dag

# Trigger a DAG
airflow dags trigger my_dag

# Mark a task as success
airflow tasks clear my_dag task_id -s 2024-01-01
```

---

## Practical Examples

### Example 1: DAG with variables

```python
from airflow import DAG
from airflow.models import Variable
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG('variables_dag', start_date=datetime(2024, 1, 1))

def process_data():
    # Get variables
    input_path = Variable.get("input_path")
    output_path = Variable.get("output_path")
    
    print(f"Processing: {input_path} -> {output_path}")

task = PythonOperator(
    task_id='process',
    python_callable=process_data,
    dag=dag,
)
```

### Example 2: DAG with connection

```python
from airflow import DAG
from airflow.hooks.postgres import PostgresHook
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG('connection_dag', start_date=datetime(2024, 1, 1))

def query_database():
    # Use a PostgreSQL connection
    hook = PostgresHook(postgres_conn_id='my_postgres')
    records = hook.get_records("SELECT * FROM users LIMIT 10")
    print(records)

task = PythonOperator(
    task_id='query',
    python_callable=query_database,
    dag=dag,
)
```

---

## 📊 Key Takeaways

1. **DAGs** define workflows
2. **Tasks** are individual steps
3. **Dependencies** define execution order
4. **Scheduling** schedules executions
5. **Variables and Connections** for configuration

## 🔗 Next Module

Proceed to module [3. Operators](../03-operators/README.md) to learn how to use different Airflow operators.

