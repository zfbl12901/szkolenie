# 4. Sensors Airflow

## 🎯 Objectifs

- Comprendre les sensors
- Utiliser FileSensor, SqlSensor, HttpSensor
- Créer des sensors personnalisés
- Gérer les timeouts et retries

## 📋 Table des matières

1. [Introduction aux sensors](#introduction-aux-sensors)
2. [FileSensor](#filesensor)
3. [SqlSensor](#sqlsensor)
4. [HttpSensor](#httpsensor)
5. [Sensors personnalisés](#sensors-personnalisés)

---

## Introduction aux sensors

### Qu'est-ce qu'un Sensor ?

**Sensor** = Tâche qui attend une condition

- **Polling** : Vérifie périodiquement une condition
- **Timeout** : Arrête après un certain temps
- **Poke interval** : Intervalle entre les vérifications
- **Mode** : Resume ou poke

### Types de sensors

- **FileSensor** : Attend un fichier
- **SqlSensor** : Attend une condition SQL
- **HttpSensor** : Attend une réponse HTTP
- **S3KeySensor** : Attend une clé S3

---

## FileSensor

### Utilisation de base

```python
from airflow.sensors.filesystem import FileSensor

task = FileSensor(
    task_id='wait_for_file',
    filepath='/path/to/file.csv',
    poke_interval=30,  # Vérifier toutes les 30 secondes
    timeout=3600,  # Timeout après 1 heure
    dag=dag,
)
```

### FileSensor avec wildcards

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

### Utilisation de base

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

### SqlSensor avec condition

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

### Utilisation de base

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

### HttpSensor avec réponse

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

## Sensors personnalisés

### Créer un sensor personnalisé

```python
from airflow.sensors.base import BaseSensorOperator
from airflow.utils.decorators import apply_defaults

class MyCustomSensor(BaseSensorOperator):
    @apply_defaults
    def __init__(self, my_param, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.my_param = my_param
    
    def poke(self, context):
        # Vérifier la condition
        # Retourner True si condition remplie
        # Retourner False pour continuer à attendre
        return self.check_condition()
    
    def check_condition(self):
        # Votre logique de vérification
        return True  # ou False

# Utilisation
task = MyCustomSensor(
    task_id='custom_sensor',
    my_param='value',
    poke_interval=30,
    timeout=3600,
    dag=dag,
)
```

---

## 📊 Points clés à retenir

1. **Sensors** attendent des conditions
2. **Polling** vérifie périodiquement
3. **Timeout** limite le temps d'attente
4. **Poke interval** définit la fréquence
5. **Mode** contrôle le comportement

## 🔗 Prochain module

Passer au module [5. Hooks](../05-hooks/README.md) pour apprendre à utiliser les hooks.

