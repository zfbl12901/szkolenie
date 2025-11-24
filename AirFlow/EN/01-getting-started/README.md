# 1. Airflow Getting Started

## 🎯 Objectives

- Understand Apache Airflow
- Install Airflow locally
- Configure the environment
- Access the web interface
- Create your first DAG

## 📋 Table of Contents

1. [Introduction to Airflow](#introduction-to-airflow)
2. [Installation](#installation)
3. [Basic Configuration](#basic-configuration)
4. [Web Interface](#web-interface)
5. [First DAG](#first-dag)

---

## Introduction to Airflow

### What is Apache Airflow?

**Apache Airflow** = Open-source workflow orchestration platform

- **Workflows** : Complex data pipelines
- **Scheduling** : Automatic scheduling
- **Monitoring** : Real-time monitoring
- **Python** : Defined in Python
- **Scalable** : From simple to very complex

### Why Airflow for Data Analyst?

- **ETL Orchestration** : Coordinate multiple steps
- **Scheduling** : Automate recurring tasks
- **Monitoring** : See pipeline status
- **Retry** : Automatic retry on error
- **Integration** : With databases, APIs, cloud services

### Airflow Components

1. **Web Server** : Web interface (port 8080)
2. **Scheduler** : Schedules and executes DAGs
3. **Metadata Database** : Stores state and metadata
4. **Workers** : Execute tasks (optional)

---

## Installation

### Prerequisites

- **Python 3.8+** : Installed on your system
- **pip** : Python package manager
- **7-8 GB RAM** : Minimum recommended

### Installation with pip

**Step 1: Create a virtual environment**

```bash
# Create a directory
mkdir airflow-project
cd airflow-project

# Create a virtual environment
python -m venv airflow-env

# Activate the environment
# Windows
airflow-env\Scripts\activate
# Linux/Mac
source airflow-env/bin/activate
```

**Step 2: Install Airflow**

```bash
# Install Airflow
pip install apache-airflow

# Install additional providers (optional)
pip install apache-airflow-providers-postgres
pip install apache-airflow-providers-http
```

**Step 3: Initialize the database**

```bash
# Initialize SQLite database (default)
airflow db init
```

**Step 4: Create an admin user**

```bash
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin123
```

### Installation with constraints (recommended)

**To avoid dependency conflicts:**

```bash
# Download constraints
AIRFLOW_VERSION=2.7.0
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

# Install with constraints
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

---

## Basic Configuration

### airflow.cfg file

**Location:**
- Windows: `%USERPROFILE%\airflow\airflow.cfg`
- Linux/Mac: `~/airflow/airflow.cfg`

**Important parameters:**

```ini
[core]
# DAGs directory
dags_folder = ~/airflow/dags

# Logs directory
base_log_folder = ~/airflow/logs

# Timezone
default_timezone = UTC

[webserver]
# Web server port
web_server_port = 8080

# Host (0.0.0.0 for network access)
web_server_host = 0.0.0.0
```

### Environment Variables

**AIRFLOW_HOME:**

```bash
# Windows
set AIRFLOW_HOME=C:\airflow

# Linux/Mac
export AIRFLOW_HOME=~/airflow
```

### Directory Structure

```
airflow/
├── dags/          # Your DAGs
├── logs/          # Execution logs
├── plugins/       # Custom plugins
└── airflow.cfg   # Configuration
```

---

## Web Interface

### Start the web server

```bash
# Activate the virtual environment
source airflow-env/bin/activate  # Linux/Mac
# or
airflow-env\Scripts\activate  # Windows

# Start the web server
airflow webserver --port 8080
```

### Start the scheduler

**In another terminal:**

```bash
# Activate the virtual environment
source airflow-env/bin/activate

# Start the scheduler
airflow scheduler
```

### Access the interface

1. Open a browser
2. Go to: `http://localhost:8080`
3. Log in with:
   - **Username**: `admin`
   - **Password**: `admin123`

### Navigation in the interface

**Main tabs:**
- **DAGs**: List of all DAGs
- **Graph**: Graphical view of a DAG
- **Tree**: Tree view of executions
- **Gantt**: Gantt chart
- **Code**: DAG source code
- **Logs**: Execution logs

---

## First DAG

### Create a simple DAG

**Step 1: Create the DAG file**

```bash
# Create the dags directory
mkdir -p ~/airflow/dags

# Create a DAG file
nano ~/airflow/dags/my_first_dag.py
```

**Step 2: DAG code**

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

# Define default arguments
default_args = {
    'owner': 'data_analyst',
    'depends_on_past': False,
    'email': ['admin@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Create the DAG
dag = DAG(
    'my_first_dag',
    default_args=default_args,
    description='My first Airflow DAG',
    schedule_interval=timedelta(days=1),
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['tutorial'],
)

# Task 1: Print date
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag,
)

# Task 2: Print a message
def print_hello():
    print("Hello from Airflow!")

t2 = PythonOperator(
    task_id='print_hello',
    python_callable=print_hello,
    dag=dag,
)

# Define dependencies
t1 >> t2  # t1 runs before t2
```

**Step 3: Verify the DAG**

```bash
# List DAGs
airflow dags list

# Check syntax
airflow dags list-import-errors

# Test the DAG
airflow dags test my_first_dag 2024-01-01
```

**Step 4: View in web interface**

1. Refresh the web page
2. The DAG `my_first_dag` appears in the list
3. Click "Trigger DAG" to execute it

---

## Practical Examples

### Example 1: DAG with multiple tasks

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data_analyst',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'example_dag',
    default_args=default_args,
    description='Example DAG with multiple tasks',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

# Task 1
extract = BashOperator(
    task_id='extract_data',
    bash_command='echo "Extracting data..."',
    dag=dag,
)

# Task 2
transform = BashOperator(
    task_id='transform_data',
    bash_command='echo "Transforming data..."',
    dag=dag,
)

# Task 3
load = BashOperator(
    task_id='load_data',
    bash_command='echo "Loading data..."',
    dag=dag,
)

# Define dependencies
extract >> transform >> load
```

### Example 2: DAG with branches

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG(
    'branching_dag',
    description='DAG with branches',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

def decide_path():
    # Logic to decide path
    return 'path_a'

decide = PythonOperator(
    task_id='decide',
    python_callable=decide_path,
    dag=dag,
)

path_a = BashOperator(
    task_id='path_a',
    bash_command='echo "Path A"',
    dag=dag,
)

path_b = BashOperator(
    task_id='path_b',
    bash_command='echo "Path B"',
    dag=dag,
)

# Conditional branching
decide >> [path_a, path_b]
```

---

## Useful Commands

### DAG Management

```bash
# List all DAGs
airflow dags list

# Check import errors
airflow dags list-import-errors

# Test a DAG
airflow dags test my_first_dag 2024-01-01

# Pause a DAG
airflow dags pause my_first_dag

# Unpause a DAG
airflow dags unpause my_first_dag

# Delete a DAG
airflow dags delete my_first_dag
```

### Task Management

```bash
# Test a task
airflow tasks test my_first_dag print_date 2024-01-01

# Run a task
airflow tasks run my_first_dag print_date 2024-01-01
```

### Database Management

```bash
# Initialize database
airflow db init

# Upgrade database
airflow db upgrade

# Reset database (WARNING: deletes everything)
airflow db reset
```

---

## Troubleshooting

### Issue: DAG not visible in interface

**Solutions:**
1. Verify file is in `~/airflow/dags/`
2. Check Python syntax
3. Check errors: `airflow dags list-import-errors`
4. Restart scheduler

### Issue: Import error

**Solutions:**
1. Verify all dependencies are installed
2. Check imports in DAG
3. Check Python paths

### Issue: Scheduler won't start

**Solutions:**
1. Verify database is initialized
2. Check logs: `~/airflow/logs/scheduler/`
3. Check permissions

---

## 📊 Key Takeaways

1. **Airflow = Orchestration** of Python workflows
2. **DAGs** define workflows
3. **Tasks** are individual steps
4. **Scheduler** executes DAGs according to schedule
5. **Web interface** allows monitoring and management

## 🔗 Next Module

Proceed to module [2. Fundamental Concepts](../02-concepts/README.md) to deepen Airflow concepts.

