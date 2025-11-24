# 2. Podstawowe koncepcje Airflow

## 🎯 Cele

- Zrozumieć DAGi (Directed Acyclic Graphs)
- Opanować zadania i zależności
- Zrozumieć harmonogramowanie
- Używać zmiennych i połączeń
- Zarządzać wykonaniami

## 📋 Spis treści

1. [DAGi (Directed Acyclic Graphs)](#dagi-directed-acyclic-graphs)
2. [Zadania i zależności](#zadania-i-zależności)
3. [Harmonogramowanie](#harmonogramowanie)
4. [Zmienne i Połączenia](#zmienne-i-połączenia)
5. [Wykonania i stany](#wykonania-i-stany)

---

## DAGi (Directed Acyclic Graphs)

### Czym jest DAG?

**DAG** = Graf skierowany acykliczny

- **Skierowany** : Zadania mają kierunek (zależności)
- **Acykliczny** : Brak pętli (brak zależności cyklicznych)
- **Graf** : Wizualna reprezentacja przepływów pracy

### Struktura DAGa

```python
from airflow import DAG
from datetime import datetime, timedelta

# Argumenty domyślne
default_args = {
    'owner': 'data_analyst',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Utworzyć DAG
dag = DAG(
    'my_dag',
    default_args=default_args,
    description='Opis DAGa',
    schedule_interval='@daily',  # Częstotliwość wykonania
    start_date=datetime(2024, 1, 1),
    catchup=False,  # Nie wykonywać przeszłych runów
    tags=['example'],
)
```

### Właściwości DAGa

**Unikalne ID :**
- Musi być unikalne w instalacji Airflow
- Używane do identyfikacji DAGa

**Schedule interval :**
- `@daily` : Codziennie
- `@hourly` : Co godzinę
- `timedelta(days=1)` : Codziennie
- `'0 2 * * *'` : Wyrażenie cron (codziennie o 2h)
- `None` : Tylko ręczne wyzwalanie

**Start date :**
- Data rozpoczęcia harmonogramowania
- Format : `datetime(rok, miesiąc, dzień)`

**Catchup :**
- `True` : Wykonuje brakujące runy od start_date
- `False` : Wykonuje tylko przyszłe runy

---

## Zadania i zależności

### Czym jest Zadanie?

**Zadanie** = Pojedynczy krok w DAGu

- **Operator** : Typ zadania (Python, Bash, SQL, itp.)
- **Unikalne ID** : Identyfikator w DAGu
- **Zależności** : Relacje z innymi zadaniami

### Typy operatorów

**BashOperator :**
```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='bash_task',
    bash_command='echo "Hello"',
    dag=dag,
)
```

**PythonOperator :**
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

### Zdefiniować zależności

**Metoda 1 : Operator >>**

```python
# t1 wykonuje się przed t2
t1 >> t2

# Wiele zależności
t1 >> [t2, t3] >> t4
```

**Metoda 2 : set_upstream / set_downstream**

```python
# t1 wykonuje się przed t2
t1.set_downstream(t2)
# lub
t2.set_upstream(t1)
```

**Metoda 3 : bitshift**

```python
# t1 >> t2 jest równoważne
t1 >> t2
```

### Przykład zależności

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

dag = DAG('dependencies_example', start_date=datetime(2024, 1, 1))

# Zadania
extract = BashOperator(task_id='extract', bash_command='echo extract', dag=dag)
transform = BashOperator(task_id='transform', bash_command='echo transform', dag=dag)
load = BashOperator(task_id='load', bash_command='echo load', dag=dag)
validate = BashOperator(task_id='validate', bash_command='echo validate', dag=dag)

# Zależności
extract >> transform >> [load, validate]
```

---

## Harmonogramowanie

### Schedule Interval

**Częste wyrażenia :**

```python
# Codziennie o północy
schedule_interval='@daily'
# lub
schedule_interval=timedelta(days=1)

# Co godzinę
schedule_interval='@hourly'
# lub
schedule_interval=timedelta(hours=1)

# Co tydzień
schedule_interval='@weekly'

# Wyrażenie cron
schedule_interval='0 2 * * *'  # Codziennie o 2h
schedule_interval='0 */6 * * *'  # Co 6 godzin
schedule_interval='0 0 * * MON'  # W każdy poniedziałek o północy
```

### Start Date i Execution Date

**Start Date :**
- Data rozpoczęcia harmonogramowania
- Format : `datetime(2024, 1, 1)`

**Execution Date :**
- Logiczna data wykonania
- Format : `YYYY-MM-DDTHH:MM:SS`

**Przykład :**
```python
dag = DAG(
    'scheduled_dag',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)
```

### Catchup

**Catchup = True :**
- Wykonuje wszystkie brakujące runy od start_date
- Może utworzyć wiele runów

**Catchup = False :**
- Wykonuje tylko przyszłe runy
- Zalecane dla większości przypadków

---

## Zmienne i Połączenia

### Zmienne

**Zmienne = Konfiguracja globalna**

**Utworzyć zmienną :**

```bash
# Przez CLI
airflow variables set my_key "my_value"

# Przez interfejs web
# Admin → Variables → Add
```

**Używać zmiennej :**

```python
from airflow.models import Variable

# Pobrać zmienną
my_value = Variable.get("my_key")
my_value_default = Variable.get("my_key", default_var="default")

# W szablonie
# {{ var.value.my_key }}
```

**Przykład :**

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

### Połączenia

**Połączenia = Informacje o połączeniu**

**Utworzyć połączenie :**

```bash
# Przez CLI
airflow connections add 'my_postgres' \
    --conn-type 'postgres' \
    --conn-host 'localhost' \
    --conn-login 'user' \
    --conn-password 'password' \
    --conn-port 5432 \
    --conn-schema 'mydb'
```

**Używać połączenia :**

```python
from airflow.hooks.base import BaseHook

# Pobrać połączenie
conn = BaseHook.get_connection('my_postgres')
print(f"Host: {conn.host}")
print(f"Login: {conn.login}")
print(f"Password: {conn.password}")
```

---

## Wykonania i stany

### Stany zadań

- **None** : Jeszcze nie wykonane
- **Scheduled** : Zaplanowane
- **Queued** : W kolejce
- **Running** : W trakcie wykonania
- **Success** : Zakończone sukcesem
- **Failed** : Nieudane
- **Skipped** : Pominięte
- **Retry** : Ponawianie
- **Up for retry** : Gotowe do ponowienia

### Stany DAGów

- **Running** : W trakcie wykonania
- **Success** : Wszystkie zadania zakończone sukcesem
- **Failed** : Przynajmniej jedno zadanie nieudane

### Zarządzać wykonaniami

**Przez interfejs web :**
- Widzieć stan wykonania
- Uruchomić ponownie zadanie
- Oznaczyć jako sukces/niepowodzenie
- Widzieć logi

**Przez CLI :**

```bash
# Listować runy
airflow dags list-runs -d my_dag

# Wyzwolić DAG
airflow dags trigger my_dag

# Oznaczyć zadanie jako sukces
airflow tasks clear my_dag task_id -s 2024-01-01
```

---

## Przykłady praktyczne

### Przykład 1 : DAG ze zmiennymi

```python
from airflow import DAG
from airflow.models import Variable
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG('variables_dag', start_date=datetime(2024, 1, 1))

def process_data():
    # Pobrać zmienne
    input_path = Variable.get("input_path")
    output_path = Variable.get("output_path")
    
    print(f"Processing: {input_path} -> {output_path}")

task = PythonOperator(
    task_id='process',
    python_callable=process_data,
    dag=dag,
)
```

### Przykład 2 : DAG z połączeniem

```python
from airflow import DAG
from airflow.hooks.postgres import PostgresHook
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG('connection_dag', start_date=datetime(2024, 1, 1))

def query_database():
    # Używać połączenia PostgreSQL
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

## 📊 Kluczowe punkty do zapamiętania

1. **DAGi** definiują workflows
2. **Zadania** to pojedyncze kroki
3. **Zależności** definiują kolejność wykonania
4. **Harmonogramowanie** planuje wykonania
5. **Zmienne i Połączenia** dla konfiguracji

## 🔗 Następny moduł

Przejdź do modułu [3. Operatory](../03-operators/README.md), aby nauczyć się używać różnych operatorów Airflow.

