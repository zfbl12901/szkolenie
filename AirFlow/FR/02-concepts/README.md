# 2. Concepts fondamentaux Airflow

## 🎯 Objectifs

- Comprendre les DAGs (Directed Acyclic Graphs)
- Maîtriser les tâches et dépendances
- Comprendre le scheduling
- Utiliser les variables et connexions
- Gérer les exécutions

## 📋 Table des matières

1. [DAGs (Directed Acyclic Graphs)](#dags-directed-acyclic-graphs)
2. [Tasks et dépendances](#tasks-et-dépendances)
3. [Scheduling](#scheduling)
4. [Variables et Connexions](#variables-et-connexions)
5. [Exécutions et états](#exécutions-et-états)

---

## DAGs (Directed Acyclic Graphs)

### Qu'est-ce qu'un DAG ?

**DAG** = Graphe orienté acyclique

- **Oriented** : Les tâches ont un sens (dépendances)
- **Acyclic** : Pas de boucles (pas de dépendances circulaires)
- **Graph** : Représentation visuelle des workflows

### Structure d'un DAG

```python
from airflow import DAG
from datetime import datetime, timedelta

# Arguments par défaut
default_args = {
    'owner': 'data_analyst',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Créer le DAG
dag = DAG(
    'my_dag',
    default_args=default_args,
    description='Description du DAG',
    schedule_interval='@daily',  # Fréquence d'exécution
    start_date=datetime(2024, 1, 1),
    catchup=False,  # Ne pas exécuter les runs passés
    tags=['example'],
)
```

### Propriétés d'un DAG

**ID unique :**
- Doit être unique dans l'installation Airflow
- Utilisé pour identifier le DAG

**Schedule interval :**
- `@daily` : Tous les jours
- `@hourly` : Toutes les heures
- `timedelta(days=1)` : Tous les jours
- `'0 2 * * *'` : Cron expression (tous les jours à 2h)
- `None` : Déclenchement manuel uniquement

**Start date :**
- Date de début du scheduling
- Format : `datetime(année, mois, jour)`

**Catchup :**
- `True` : Exécute les runs manqués depuis start_date
- `False` : N'exécute que les runs futurs

---

## Tasks et dépendances

### Qu'est-ce qu'une Task ?

**Task** = Étape individuelle dans un DAG

- **Opérateur** : Type de tâche (Python, Bash, SQL, etc.)
- **ID unique** : Identifiant dans le DAG
- **Dépendances** : Relations avec d'autres tâches

### Types d'opérateurs

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

### Définir les dépendances

**Méthode 1 : Opérateur >>**

```python
# t1 s'exécute avant t2
t1 >> t2

# Plusieurs dépendances
t1 >> [t2, t3] >> t4
```

**Méthode 2 : set_upstream / set_downstream**

```python
# t1 s'exécute avant t2
t1.set_downstream(t2)
# ou
t2.set_upstream(t1)
```

**Méthode 3 : bitshift**

```python
# t1 >> t2 équivaut à
t1 >> t2
```

### Exemple de dépendances

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

dag = DAG('dependencies_example', start_date=datetime(2024, 1, 1))

# Tâches
extract = BashOperator(task_id='extract', bash_command='echo extract', dag=dag)
transform = BashOperator(task_id='transform', bash_command='echo transform', dag=dag)
load = BashOperator(task_id='load', bash_command='echo load', dag=dag)
validate = BashOperator(task_id='validate', bash_command='echo validate', dag=dag)

# Dépendances
extract >> transform >> [load, validate]
```

---

## Scheduling

### Schedule Interval

**Expressions courantes :**

```python
# Tous les jours à minuit
schedule_interval='@daily'
# ou
schedule_interval=timedelta(days=1)

# Toutes les heures
schedule_interval='@hourly'
# ou
schedule_interval=timedelta(hours=1)

# Toutes les semaines
schedule_interval='@weekly'

# Expression cron
schedule_interval='0 2 * * *'  # Tous les jours à 2h
schedule_interval='0 */6 * * *'  # Toutes les 6 heures
schedule_interval='0 0 * * MON'  # Tous les lundis à minuit
```

### Start Date et Execution Date

**Start Date :**
- Date de début du scheduling
- Format : `datetime(2024, 1, 1)`

**Execution Date :**
- Date logique d'exécution
- Format : `YYYY-MM-DDTHH:MM:SS`

**Exemple :**
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
- Exécute tous les runs manqués depuis start_date
- Peut créer beaucoup de runs

**Catchup = False :**
- N'exécute que les runs futurs
- Recommandé pour la plupart des cas

---

## Variables et Connexions

### Variables

**Variables = Configuration globale**

**Créer une variable :**

```bash
# Via CLI
airflow variables set my_key "my_value"

# Via interface web
# Admin → Variables → Add
```

**Utiliser une variable :**

```python
from airflow.models import Variable

# Récupérer une variable
my_value = Variable.get("my_key")
my_value_default = Variable.get("my_key", default_var="default")

# Dans un template
# {{ var.value.my_key }}
```

**Exemple :**

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

### Connexions

**Connexions = Informations de connexion**

**Créer une connexion :**

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

**Utiliser une connexion :**

```python
from airflow.hooks.base import BaseHook

# Récupérer une connexion
conn = BaseHook.get_connection('my_postgres')
print(f"Host: {conn.host}")
print(f"Login: {conn.login}")
print(f"Password: {conn.password}")
```

---

## Exécutions et états

### États des tâches

- **None** : Pas encore exécutée
- **Scheduled** : Planifiée
- **Queued** : En attente
- **Running** : En cours d'exécution
- **Success** : Réussie
- **Failed** : Échouée
- **Skipped** : Ignorée
- **Retry** : Nouvelle tentative
- **Up for retry** : Prête pour retry

### États des DAGs

- **Running** : En cours d'exécution
- **Success** : Toutes les tâches réussies
- **Failed** : Au moins une tâche échouée

### Gérer les exécutions

**Via l'interface web :**
- Voir l'état des exécutions
- Relancer une tâche
- Marquer comme succès/échec
- Voir les logs

**Via CLI :**

```bash
# Lister les runs
airflow dags list-runs -d my_dag

# Déclencher un DAG
airflow dags trigger my_dag

# Marquer une tâche comme succès
airflow tasks clear my_dag task_id -s 2024-01-01
```

---

## Exemples pratiques

### Exemple 1 : DAG avec variables

```python
from airflow import DAG
from airflow.models import Variable
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG('variables_dag', start_date=datetime(2024, 1, 1))

def process_data():
    # Récupérer des variables
    input_path = Variable.get("input_path")
    output_path = Variable.get("output_path")
    
    print(f"Processing: {input_path} -> {output_path}")

task = PythonOperator(
    task_id='process',
    python_callable=process_data,
    dag=dag,
)
```

### Exemple 2 : DAG avec connexion

```python
from airflow import DAG
from airflow.hooks.postgres import PostgresHook
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG('connection_dag', start_date=datetime(2024, 1, 1))

def query_database():
    # Utiliser une connexion PostgreSQL
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

## 📊 Points clés à retenir

1. **DAGs** définissent les workflows
2. **Tasks** sont les étapes individuelles
3. **Dépendances** définissent l'ordre d'exécution
4. **Scheduling** planifie les exécutions
5. **Variables et Connexions** pour la configuration

## 🔗 Prochain module

Passer au module [3. Opérateurs](../03-operators/README.md) pour apprendre à utiliser les différents opérateurs Airflow.

