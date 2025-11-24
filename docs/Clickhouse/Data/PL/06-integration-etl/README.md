# 6. Integracja i ETL

## 🎯 Cele

- Importować dane do ClickHouse
- Eksportować dane z ClickHouse
- Integrować z Python
- Integrować z PowerBI/Tableau

## Import danych

### Z CSV

```sql
INSERT INTO events
FROM INFILE '/path/to/file.csv'
FORMAT CSV;
```

### Z JSON

```sql
INSERT INTO events
FROM INFILE '/path/to/file.json'
FORMAT JSONEachRow;
```

### Przez clickhouse-client

```bash
clickhouse-client --query "INSERT INTO events FORMAT CSV" < data.csv
```

## Eksport danych

### Do CSV

```sql
SELECT * FROM events
INTO OUTFILE '/path/to/output.csv'
FORMAT CSV;
```

### Przez clickhouse-client

```bash
clickhouse-client --query "SELECT * FROM events FORMAT CSV" > output.csv
```

## Integracja Python

### Instalacja

```bash
pip install clickhouse-driver
```

### Połączenie

```python
from clickhouse_driver import Client

client = Client(host='localhost', port=9000, database='analytics')
```

### Zapytania

```python
# Wykonaj zapytanie
result = client.execute('SELECT * FROM events LIMIT 10')

# Wstaw dane
client.execute('INSERT INTO events VALUES', [
    (1, '2024-01-15', 100, 'click', 1.5),
    (2, '2024-01-15', 101, 'view', 2.0)
])
```

### Pandas

```python
import pandas as pd

# Czytaj z ClickHouse
df = pd.read_sql('SELECT * FROM events', client.connection)

# Zapisz do ClickHouse
df.to_sql('events', client.connection, if_exists='append')
```

## Integracja PowerBI

### Połączenie ODBC

1. Zainstaluj sterownik ODBC ClickHouse
2. Utwórz źródło danych ODBC
3. Połącz z PowerBI

## ETL z Python

### Kompletny przykład

```python
from clickhouse_driver import Client
import pandas as pd

# Połączenie
client = Client(host='localhost', port=9000)

# Czytaj ze źródła
df = pd.read_csv('source_data.csv')

# Transformacja
df['processed_date'] = pd.to_datetime(df['date'])
df = df[df['value'] > 0]

# Zapisz do ClickHouse
client.execute('INSERT INTO events VALUES', df.values.tolist())
```

---

**Następny krok :** [Najlepsze praktyki](./07-best-practices/README.md)

