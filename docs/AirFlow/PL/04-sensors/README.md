# 4. Sensory Airflow

## 🎯 Cele

- Zrozumieć sensory
- Używać FileSensor, SqlSensor, HttpSensor
- Tworzyć sensory niestandardowe
- Zarządzać timeoutami i ponowieniami

## 📋 Spis treści

1. [Wprowadzenie do sensorów](#wprowadzenie-do-sensorów)
2. [FileSensor](#filesensor)
3. [SqlSensor](#sqlsensor)
4. [HttpSensor](#httpsensor)
5. [Sensory niestandardowe](#sensory-niestandardowe)

---

## Wprowadzenie do sensorów

### Czym jest Sensor?

**Sensor** = Zadanie które czeka na warunek

- **Polling** : Sprawdza okresowo warunek
- **Timeout** : Zatrzymuje się po określonym czasie
- **Poke interval** : Interwał między sprawdzeniami
- **Mode** : Resume lub poke

### Typy sensorów

- **FileSensor** : Czeka na plik
- **SqlSensor** : Czeka na warunek SQL
- **HttpSensor** : Czeka na odpowiedź HTTP
- **S3KeySensor** : Czeka na klucz S3

---

## FileSensor

### Podstawowe użycie

```python
from airflow.sensors.filesystem import FileSensor

task = FileSensor(
    task_id='wait_for_file',
    filepath='/path/to/file.csv',
    poke_interval=30,  # Sprawdzać co 30 sekund
    timeout=3600,  # Timeout po 1 godzinie
    dag=dag,
)
```

### FileSensor z wildcards

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

### Podstawowe użycie

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

### SqlSensor z warunkiem

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

### Podstawowe użycie

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

### HttpSensor z odpowiedzią

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

## Sensory niestandardowe

### Utworzyć sensor niestandardowy

```python
from airflow.sensors.base import BaseSensorOperator
from airflow.utils.decorators import apply_defaults

class MyCustomSensor(BaseSensorOperator):
    @apply_defaults
    def __init__(self, my_param, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.my_param = my_param
    
    def poke(self, context):
        # Sprawdzić warunek
        # Zwrócić True jeśli warunek spełniony
        # Zwrócić False aby kontynuować oczekiwanie
        return self.check_condition()
    
    def check_condition(self):
        # Twoja logika weryfikacji
        return True  # lub False

# Użycie
task = MyCustomSensor(
    task_id='custom_sensor',
    my_param='value',
    poke_interval=30,
    timeout=3600,
    dag=dag,
)
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Sensory** czekają na warunki
2. **Polling** sprawdza okresowo
3. **Timeout** ogranicza czas oczekiwania
4. **Poke interval** definiuje częstotliwość
5. **Mode** kontroluje zachowanie

## 🔗 Następny moduł

Przejdź do modułu [5. Hooki](../05-hooks/README.md), aby nauczyć się używać hooków.

