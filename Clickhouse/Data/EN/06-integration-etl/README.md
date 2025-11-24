# 6. Integration and ETL

## 🎯 Objectives

- Import data into ClickHouse
- Export data from ClickHouse
- Integrate with Python
- Integrate with PowerBI/Tableau

## Data Import

### From CSV

```sql
INSERT INTO events
FROM INFILE '/path/to/file.csv'
FORMAT CSV;
```

### From JSON

```sql
INSERT INTO events
FROM INFILE '/path/to/file.json'
FORMAT JSONEachRow;
```

### Via clickhouse-client

```bash
clickhouse-client --query "INSERT INTO events FORMAT CSV" < data.csv
```

## Data Export

### To CSV

```sql
SELECT * FROM events
INTO OUTFILE '/path/to/output.csv'
FORMAT CSV;
```

### Via clickhouse-client

```bash
clickhouse-client --query "SELECT * FROM events FORMAT CSV" > output.csv
```

## Python Integration

### Installation

```bash
pip install clickhouse-driver
```

### Connection

```python
from clickhouse_driver import Client

client = Client(host='localhost', port=9000, database='analytics')
```

### Queries

```python
# Execute query
result = client.execute('SELECT * FROM events LIMIT 10')

# Insert data
client.execute('INSERT INTO events VALUES', [
    (1, '2024-01-15', 100, 'click', 1.5),
    (2, '2024-01-15', 101, 'view', 2.0)
])
```

### Pandas

```python
import pandas as pd

# Read from ClickHouse
df = pd.read_sql('SELECT * FROM events', client.connection)

# Write to ClickHouse
df.to_sql('events', client.connection, if_exists='append')
```

## PowerBI Integration

### ODBC Connection

1. Install ClickHouse ODBC driver
2. Create ODBC data source
3. Connect from PowerBI

## ETL with Python

### Complete example

```python
from clickhouse_driver import Client
import pandas as pd

# Connection
client = Client(host='localhost', port=9000)

# Read from source
df = pd.read_csv('source_data.csv')

# Transformation
df['processed_date'] = pd.to_datetime(df['date'])
df = df[df['value'] > 0]

# Write to ClickHouse
client.execute('INSERT INTO events VALUES', df.values.tolist())
```

---

**Next step :** [Best Practices](./07-best-practices/README.md)

