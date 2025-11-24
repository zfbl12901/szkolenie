# 1. Rozpoczęcie z Airflow

## 🎯 Cele

- Zrozumieć Apache Airflow
- Zainstalować Airflow lokalnie
- Skonfigurować środowisko
- Uzyskać dostęp do interfejsu web
- Utworzyć pierwszy DAG

## 📋 Spis treści

1. [Wprowadzenie do Airflow](#wprowadzenie-do-airflow)
2. [Instalacja](#instalacja)
3. [Podstawowa konfiguracja](#podstawowa-konfiguracja)
4. [Interfejs web](#interfejs-web)
5. [Pierwszy DAG](#pierwszy-dag)

---

## Wprowadzenie do Airflow

### Czym jest Apache Airflow?

**Apache Airflow** = Platforma open-source do orkiestracji przepływów pracy

- **Workflows** : Złożone pipeline'y danych
- **Scheduling** : Automatyczne planowanie
- **Monitorowanie** : Monitorowanie w czasie rzeczywistym
- **Python** : Zdefiniowane w Pythonie
- **Skalowalne** : Od prostych do bardzo złożonych

### Dlaczego Airflow dla Data Analyst?

- **Orkiestracja ETL** : Koordynować wiele kroków
- **Harmonogramowanie** : Automatyzować zadania cykliczne
- **Monitorowanie** : Widzieć status pipeline'ów
- **Retry** : Automatyczne ponowienie przy błędzie
- **Integracja** : Z bazami danych, API, usługami chmurowymi

### Komponenty Airflow

1. **Web Server** : Interfejs web (port 8080)
2. **Scheduler** : Planuje i wykonuje DAGi
3. **Metadata Database** : Przechowuje stan i metadane
4. **Workers** : Wykonują zadania (opcjonalne)

---

## Instalacja

### Wymagania wstępne

- **Python 3.8+** : Zainstalowany w systemie
- **pip** : Menedżer pakietów Python
- **7-8 GB RAM** : Minimum zalecane

### Instalacja z pip

**Krok 1 : Utworzyć środowisko wirtualne**

```bash
# Utworzyć katalog
mkdir airflow-project
cd airflow-project

# Utworzyć środowisko wirtualne
python -m venv airflow-env

# Aktywować środowisko
# Windows
airflow-env\Scripts\activate
# Linux/Mac
source airflow-env/bin/activate
```

**Krok 2 : Zainstalować Airflow**

```bash
# Zainstalować Airflow
pip install apache-airflow

# Zainstalować dodatkowe providers (opcjonalne)
pip install apache-airflow-providers-postgres
pip install apache-airflow-providers-http
```

**Krok 3 : Zainicjalizować bazę danych**

```bash
# Zainicjalizować bazę danych SQLite (domyślnie)
airflow db init
```

**Krok 4 : Utworzyć użytkownika admin**

```bash
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin123
```

### Instalacja z constraints (zalecane)

**Aby uniknąć konfliktów zależności :**

```bash
# Pobrać constraints
AIRFLOW_VERSION=2.7.0
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

# Zainstalować z constraints
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

---

## Podstawowa konfiguracja

### Plik airflow.cfg

**Lokalizacja :**
- Windows : `%USERPROFILE%\airflow\airflow.cfg`
- Linux/Mac : `~/airflow/airflow.cfg`

**Ważne parametry :**

```ini
[core]
# Katalog DAGów
dags_folder = ~/airflow/dags

# Katalog logów
base_log_folder = ~/airflow/logs

# Strefa czasowa
default_timezone = Europe/Warsaw

[webserver]
# Port serwera web
web_server_port = 8080

# Host (0.0.0.0 dla dostępu sieciowego)
web_server_host = 0.0.0.0
```

### Zmienne środowiskowe

**AIRFLOW_HOME :**

```bash
# Windows
set AIRFLOW_HOME=C:\airflow

# Linux/Mac
export AIRFLOW_HOME=~/airflow
```

### Struktura katalogów

```
airflow/
├── dags/          # Twoje DAGi
├── logs/          # Logi wykonania
├── plugins/       # Pluginy niestandardowe
└── airflow.cfg   # Konfiguracja
```

---

## Interfejs web

### Uruchomić serwer web

```bash
# Aktywować środowisko wirtualne
source airflow-env/bin/activate  # Linux/Mac
# lub
airflow-env\Scripts\activate  # Windows

# Uruchomić serwer web
airflow webserver --port 8080
```

### Uruchomić scheduler

**W innym terminalu :**

```bash
# Aktywować środowisko wirtualne
source airflow-env/bin/activate

# Uruchomić scheduler
airflow scheduler
```

### Dostęp do interfejsu

1. Otworzyć przeglądarkę
2. Przejść do : `http://localhost:8080`
3. Zalogować się z :
   - **Username** : `admin`
   - **Password** : `admin123`

### Nawigacja w interfejsie

**Główne zakładki :**
- **DAGs** : Lista wszystkich DAGów
- **Graph** : Widok graficzny DAGa
- **Tree** : Widok drzewa wykonania
- **Gantt** : Diagram Gantta
- **Code** : Kod źródłowy DAGa
- **Logs** : Logi wykonania

---

## Pierwszy DAG

### Utworzyć prosty DAG

**Krok 1 : Utworzyć plik DAG**

```bash
# Utworzyć katalog dags
mkdir -p ~/airflow/dags

# Utworzyć plik DAG
nano ~/airflow/dags/my_first_dag.py
```

**Krok 2 : Kod DAGa**

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

# Zdefiniować argumenty domyślne
default_args = {
    'owner': 'data_analyst',
    'depends_on_past': False,
    'email': ['admin@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Utworzyć DAG
dag = DAG(
    'my_first_dag',
    default_args=default_args,
    description='Mój pierwszy DAG Airflow',
    schedule_interval=timedelta(days=1),
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['tutorial'],
)

# Zadanie 1 : Wyświetlić datę
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag,
)

# Zadanie 2 : Wyświetlić wiadomość
def print_hello():
    print("Hello from Airflow!")

t2 = PythonOperator(
    task_id='print_hello',
    python_callable=print_hello,
    dag=dag,
)

# Zdefiniować zależności
t1 >> t2  # t1 wykonuje się przed t2
```

**Krok 3 : Sprawdzić DAG**

```bash
# Listować DAGi
airflow dags list

# Sprawdzić składnię
airflow dags list-import-errors

# Testować DAG
airflow dags test my_first_dag 2024-01-01
```

**Krok 4 : Zobaczyć w interfejsie web**

1. Odświeżyć stronę web
2. DAG `my_first_dag` pojawia się na liście
3. Kliknąć "Trigger DAG" aby go wykonać

---

## Przykłady praktyczne

### Przykład 1 : DAG z wieloma zadaniami

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
    description='Przykład DAGa z wieloma zadaniami',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

# Zadanie 1
extract = BashOperator(
    task_id='extract_data',
    bash_command='echo "Extracting data..."',
    dag=dag,
)

# Zadanie 2
transform = BashOperator(
    task_id='transform_data',
    bash_command='echo "Transforming data..."',
    dag=dag,
)

# Zadanie 3
load = BashOperator(
    task_id='load_data',
    bash_command='echo "Loading data..."',
    dag=dag,
)

# Zdefiniować zależności
extract >> transform >> load
```

### Przykład 2 : DAG z gałęziami

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG(
    'branching_dag',
    description='DAG z gałęziami',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

def decide_path():
    # Logika do decyzji ścieżki
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

# Rozgałęzienie warunkowe
decide >> [path_a, path_b]
```

---

## Przydatne polecenia

### Zarządzanie DAGami

```bash
# Listować wszystkie DAGi
airflow dags list

# Sprawdzić błędy importu
airflow dags list-import-errors

# Testować DAG
airflow dags test my_first_dag 2024-01-01

# Wstrzymać DAG
airflow dags pause my_first_dag

# Wznowić DAG
airflow dags unpause my_first_dag

# Usunąć DAG
airflow dags delete my_first_dag
```

### Zarządzanie zadaniami

```bash
# Testować zadanie
airflow tasks test my_first_dag print_date 2024-01-01

# Wykonać zadanie
airflow tasks run my_first_dag print_date 2024-01-01
```

### Zarządzanie bazą danych

```bash
# Zainicjalizować bazę
airflow db init

# Zaktualizować bazę
airflow db upgrade

# Zresetować bazę (UWAGA : usuwa wszystko)
airflow db reset
```

---

## Rozwiązywanie problemów

### Problem : DAG niewidoczny w interfejsie

**Rozwiązania :**
1. Sprawdzić że plik jest w `~/airflow/dags/`
2. Sprawdzić składnię Pythona
3. Sprawdzić błędy : `airflow dags list-import-errors`
4. Uruchomić ponownie scheduler

### Problem : Błąd importu

**Rozwiązania :**
1. Sprawdzić że wszystkie zależności są zainstalowane
2. Sprawdzić importy w DAGu
3. Sprawdzić ścieżki Pythona

### Problem : Scheduler nie uruchamia się

**Rozwiązania :**
1. Sprawdzić że baza danych jest zainicjalizowana
2. Sprawdzić logi : `~/airflow/logs/scheduler/`
3. Sprawdzić uprawnienia

---

## 📊 Kluczowe punkty do zapamiętania

1. **Airflow = Orkiestracja** przepływów pracy Python
2. **DAGi** definiują workflows
3. **Zadania** to pojedyncze kroki
4. **Scheduler** wykonuje DAGi według harmonogramu
5. **Interfejs web** umożliwia monitorowanie i zarządzanie

## 🔗 Następny moduł

Przejdź do modułu [2. Podstawowe koncepcje](../02-concepts/README.md), aby pogłębić koncepcje Airflow.

