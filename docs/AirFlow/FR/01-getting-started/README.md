# 1. Prise en main Airflow

## 🎯 Objectifs

- Comprendre Apache Airflow
- Installer Airflow localement
- Configurer l'environnement
- Accéder à l'interface web
- Créer votre premier DAG

## 📋 Table des matières

1. [Introduction à Airflow](#introduction-à-airflow)
2. [Installation](#installation)
3. [Configuration de base](#configuration-de-base)
4. [Interface web](#interface-web)
5. [Premier DAG](#premier-dag)

---

## Introduction à Airflow

### Qu'est-ce qu'Apache Airflow ?

**Apache Airflow** = Plateforme open-source d'orchestration de workflows

- **Workflows** : Pipelines de données complexes
- **Scheduling** : Planification automatique
- **Monitoring** : Surveillance en temps réel
- **Python** : Défini en Python
- **Scalable** : De simple à très complexe

### Pourquoi Airflow pour Data Analyst ?

- **Orchestration ETL** : Coordonner plusieurs étapes
- **Scheduling** : Automatiser les tâches récurrentes
- **Monitoring** : Voir l'état des pipelines
- **Retry** : Nouvelle tentative automatique en cas d'erreur
- **Intégration** : Avec bases de données, APIs, services cloud

### Composants Airflow

1. **Web Server** : Interface web (port 8080)
2. **Scheduler** : Planifie et exécute les DAGs
3. **Metadata Database** : Stocke l'état et les métadonnées
4. **Workers** : Exécutent les tâches (optionnel)

---

## Installation

### Prérequis

- **Python 3.8+** : Installé sur votre système
- **pip** : Gestionnaire de paquets Python
- **7-8 Go RAM** : Minimum recommandé

### Installation avec pip

**Étape 1 : Créer un environnement virtuel**

```bash
# Créer un répertoire
mkdir airflow-project
cd airflow-project

# Créer un environnement virtuel
python -m venv airflow-env

# Activer l'environnement
# Windows
airflow-env\Scripts\activate
# Linux/Mac
source airflow-env/bin/activate
```

**Étape 2 : Installer Airflow**

```bash
# Installer Airflow
pip install apache-airflow

# Installer des providers supplémentaires (optionnel)
pip install apache-airflow-providers-postgres
pip install apache-airflow-providers-http
```

**Étape 3 : Initialiser la base de données**

```bash
# Initialiser la base de données SQLite (par défaut)
airflow db init
```

**Étape 4 : Créer un utilisateur admin**

```bash
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin123
```

### Installation avec constraints (recommandé)

**Pour éviter les conflits de dépendances :**

```bash
# Télécharger les constraints
AIRFLOW_VERSION=2.7.0
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

# Installer avec constraints
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

---

## Configuration de base

### Fichier airflow.cfg

**Localisation :**
- Windows : `%USERPROFILE%\airflow\airflow.cfg`
- Linux/Mac : `~/airflow/airflow.cfg`

**Paramètres importants :**

```ini
[core]
# Répertoire des DAGs
dags_folder = ~/airflow/dags

# Répertoire des logs
base_log_folder = ~/airflow/logs

# Fuseau horaire
default_timezone = Europe/Paris

[webserver]
# Port du serveur web
web_server_port = 8080

# Host (0.0.0.0 pour accès réseau)
web_server_host = 0.0.0.0
```

### Variables d'environnement

**AIRFLOW_HOME :**

```bash
# Windows
set AIRFLOW_HOME=C:\airflow

# Linux/Mac
export AIRFLOW_HOME=~/airflow
```

### Structure des répertoires

```
airflow/
├── dags/          # Vos DAGs
├── logs/          # Logs d'exécution
├── plugins/       # Plugins personnalisés
└── airflow.cfg   # Configuration
```

---

## Interface web

### Démarrer le serveur web

```bash
# Activer l'environnement virtuel
source airflow-env/bin/activate  # Linux/Mac
# ou
airflow-env\Scripts\activate  # Windows

# Démarrer le serveur web
airflow webserver --port 8080
```

### Démarrer le scheduler

**Dans un autre terminal :**

```bash
# Activer l'environnement virtuel
source airflow-env/bin/activate

# Démarrer le scheduler
airflow scheduler
```

### Accéder à l'interface

1. Ouvrir un navigateur
2. Aller sur : `http://localhost:8080`
3. Se connecter avec :
   - **Username** : `admin`
   - **Password** : `admin123`

### Navigation dans l'interface

**Onglets principaux :**
- **DAGs** : Liste de tous les DAGs
- **Graph** : Vue graphique d'un DAG
- **Tree** : Vue arborescente des exécutions
- **Gantt** : Diagramme de Gantt
- **Code** : Code source du DAG
- **Logs** : Logs d'exécution

---

## Premier DAG

### Créer un DAG simple

**Étape 1 : Créer le fichier DAG**

```bash
# Créer le répertoire dags
mkdir -p ~/airflow/dags

# Créer un fichier DAG
nano ~/airflow/dags/my_first_dag.py
```

**Étape 2 : Code du DAG**

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

# Définir les arguments par défaut
default_args = {
    'owner': 'data_analyst',
    'depends_on_past': False,
    'email': ['admin@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Créer le DAG
dag = DAG(
    'my_first_dag',
    default_args=default_args,
    description='Mon premier DAG Airflow',
    schedule_interval=timedelta(days=1),
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['tutorial'],
)

# Tâche 1 : Afficher la date
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag,
)

# Tâche 2 : Afficher un message
def print_hello():
    print("Hello from Airflow!")

t2 = PythonOperator(
    task_id='print_hello',
    python_callable=print_hello,
    dag=dag,
)

# Définir les dépendances
t1 >> t2  # t1 s'exécute avant t2
```

**Étape 3 : Vérifier le DAG**

```bash
# Lister les DAGs
airflow dags list

# Vérifier la syntaxe
airflow dags list-import-errors

# Tester le DAG
airflow dags test my_first_dag 2024-01-01
```

**Étape 4 : Voir dans l'interface web**

1. Rafraîchir la page web
2. Le DAG `my_first_dag` apparaît dans la liste
3. Cliquer sur "Trigger DAG" pour l'exécuter

---

## Exemples pratiques

### Exemple 1 : DAG avec plusieurs tâches

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
    description='Exemple de DAG avec plusieurs tâches',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

# Tâche 1
extract = BashOperator(
    task_id='extract_data',
    bash_command='echo "Extracting data..."',
    dag=dag,
)

# Tâche 2
transform = BashOperator(
    task_id='transform_data',
    bash_command='echo "Transforming data..."',
    dag=dag,
)

# Tâche 3
load = BashOperator(
    task_id='load_data',
    bash_command='echo "Loading data..."',
    dag=dag,
)

# Définir les dépendances
extract >> transform >> load
```

### Exemple 2 : DAG avec branches

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator
from datetime import datetime

dag = DAG(
    'branching_dag',
    description='DAG avec branches',
    schedule_interval='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

def decide_path():
    # Logique pour décider du chemin
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

# Branchement conditionnel
decide >> [path_a, path_b]
```

---

## Commandes utiles

### Gestion des DAGs

```bash
# Lister tous les DAGs
airflow dags list

# Vérifier les erreurs d'import
airflow dags list-import-errors

# Tester un DAG
airflow dags test my_first_dag 2024-01-01

# Pauser un DAG
airflow dags pause my_first_dag

# Reprendre un DAG
airflow dags unpause my_first_dag

# Supprimer un DAG
airflow dags delete my_first_dag
```

### Gestion des tâches

```bash
# Tester une tâche
airflow tasks test my_first_dag print_date 2024-01-01

# Exécuter une tâche
airflow tasks run my_first_dag print_date 2024-01-01
```

### Gestion de la base de données

```bash
# Initialiser la base
airflow db init

# Mettre à jour la base
airflow db upgrade

# Réinitialiser la base (ATTENTION : supprime tout)
airflow db reset
```

---

## Dépannage

### Problème : DAG non visible dans l'interface

**Solutions :**
1. Vérifier que le fichier est dans `~/airflow/dags/`
2. Vérifier la syntaxe Python
3. Vérifier les erreurs : `airflow dags list-import-errors`
4. Redémarrer le scheduler

### Problème : Erreur d'import

**Solutions :**
1. Vérifier que toutes les dépendances sont installées
2. Vérifier les imports dans le DAG
3. Vérifier les chemins Python

### Problème : Scheduler ne démarre pas

**Solutions :**
1. Vérifier que la base de données est initialisée
2. Vérifier les logs : `~/airflow/logs/scheduler/`
3. Vérifier les permissions

---

## 📊 Points clés à retenir

1. **Airflow = Orchestration** de workflows Python
2. **DAGs** définissent les workflows
3. **Tasks** sont les étapes individuelles
4. **Scheduler** exécute les DAGs selon le planning
5. **Interface web** permet de monitorer et gérer

## 🔗 Prochain module

Passer au module [2. Concepts fondamentaux](../02-concepts/README.md) pour approfondir les concepts d'Airflow.

