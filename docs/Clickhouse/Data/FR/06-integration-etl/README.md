# 6. Intégration et ETL

## 🎯 Objectifs

- Importer des données dans ClickHouse
- Exporter des données depuis ClickHouse
- Intégrer avec Python
- Intégrer avec PowerBI/Tableau

## Import de données

### Depuis CSV

```sql
INSERT INTO events
FROM INFILE '/path/to/file.csv'
FORMAT CSV;
```

### Depuis JSON

```sql
INSERT INTO events
FROM INFILE '/path/to/file.json'
FORMAT JSONEachRow;
```

### Via clickhouse-client

```bash
clickhouse-client --query "INSERT INTO events FORMAT CSV" < data.csv
```

## Export de données

### Vers CSV

```sql
SELECT * FROM events
INTO OUTFILE '/path/to/output.csv'
FORMAT CSV;
```

### Vers JSON

```sql
SELECT * FROM events
INTO OUTFILE '/path/to/output.json'
FORMAT JSONEachRow;
```

### Via clickhouse-client

```bash
clickhouse-client --query "SELECT * FROM events FORMAT CSV" > output.csv
```

## Intégration Python

### Installation

```bash
pip install clickhouse-driver
```

### Connexion

```python
from clickhouse_driver import Client

client = Client(host='localhost', port=9000, database='analytics')
```

### Requêtes

```python
# Exécuter une requête
result = client.execute('SELECT * FROM events LIMIT 10')

# Insérer des données
client.execute('INSERT INTO events VALUES', [
    (1, '2024-01-15', 100, 'click', 1.5),
    (2, '2024-01-15', 101, 'view', 2.0)
])
```

### Pandas

```python
import pandas as pd

# Lire depuis ClickHouse
df = pd.read_sql('SELECT * FROM events', client.connection)

# Écrire vers ClickHouse
df.to_sql('events', client.connection, if_exists='append')
```

## Intégration PowerBI

### Connexion ODBC

1. Installer le driver ODBC ClickHouse
2. Créer une source de données ODBC
3. Se connecter depuis PowerBI

### Requête directe

```sql
SELECT 
    event_date,
    COUNT(*) as count
FROM events
GROUP BY event_date
```

## Intégration Tableau

### Connexion native

1. Sélectionner "ClickHouse" comme source
2. Entrer les paramètres de connexion
3. Créer des visualisations

## ETL avec Python

### Exemple complet

```python
from clickhouse_driver import Client
import pandas as pd

# Connexion
client = Client(host='localhost', port=9000)

# Lire depuis source
df = pd.read_csv('source_data.csv')

# Transformation
df['processed_date'] = pd.to_datetime(df['date'])
df = df[df['value'] > 0]

# Écrire vers ClickHouse
client.execute('INSERT INTO events VALUES', df.values.tolist())
```

---

**Prochaine étape :** [Bonnes Pratiques](./07-best-practices/README.md)

