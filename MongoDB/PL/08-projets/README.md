# 8. Projekty praktyczne MongoDB

## 🎯 Cele

- Tworzyć aplikację Python z MongoDB
- Pipeline danych z MongoDB
- Analiza danych
- Projekty do portfolio

## 📋 Spis treści

1. [Projekt 1 : Aplikacja Python](#projekt-1--aplikacja-python)
2. [Projekt 2 : Pipeline danych](#projekt-2--pipeline-danych)
3. [Projekt 3 : Analiza danych](#projekt-3--analiza-danych)
4. [Projekt 4 : API REST](#projekt-4--api-rest)

---

## Projekt 1 : Aplikacja Python

### Cel

Utworzyć aplikację Python używającą MongoDB.

### Kod Python

```python
from pymongo import MongoClient
from datetime import datetime

# Połączenie
client = MongoClient('mongodb://localhost:27017/')
db = client['mydb']
collection = db['users']

# Wstawić użytkownika
def create_user(name, email):
    user = {
        'name': name,
        'email': email,
        'created_at': datetime.now()
    }
    result = collection.insert_one(user)
    return result.inserted_id

# Znaleźć użytkownika
def find_user(email):
    return collection.find_one({'email': email})

# Użycie
create_user('John', 'john@example.com')
user = find_user('john@example.com')
print(user)
```

---

## Projekt 2 : Pipeline danych

### Cel

Utworzyć pipeline ETL z MongoDB.

### Kod

```python
from pymongo import MongoClient
import pandas as pd

client = MongoClient('mongodb://localhost:27017/')
db = client['etl_db']

# Extract
def extract():
    df = pd.read_csv('data.csv')
    return df

# Transform
def transform(df):
    df['processed_at'] = pd.Timestamp.now()
    df = df.dropna()
    return df

# Load
def load(df):
    records = df.to_dict('records')
    db.data.insert_many(records)

# Pipeline
df = extract()
df = transform(df)
load(df)
```

---

## Projekt 3 : Analiza danych

### Cel

Analizować dane z MongoDB Aggregation.

### Kod

```python
from pymongo import MongoClient

client = MongoClient('mongodb://localhost:27017/')
db = client['analytics_db']

# Pipeline agregacji
pipeline = [
    {'$match': {'date': {'$gte': datetime(2024, 1, 1)}}},
    {'$group': {
        '_id': '$product',
        'total_sales': {'$sum': '$amount'},
        'count': {'$sum': 1}
    }},
    {'$sort': {'total_sales': -1}},
    {'$limit': 10}
]

# Wykonać
results = db.sales.aggregate(pipeline)
for result in results:
    print(result)
```

---

## Projekt 4 : API REST

### Cel

Utworzyć API REST z Flask i MongoDB.

### Kod

```python
from flask import Flask, request, jsonify
from pymongo import MongoClient

app = Flask(__name__)
client = MongoClient('mongodb://localhost:27017/')
db = client['api_db']
collection = db['items']

@app.route('/items', methods=['GET'])
def get_items():
    items = list(collection.find({}, {'_id': 0}))
    return jsonify(items)

@app.route('/items', methods=['POST'])
def create_item():
    data = request.json
    result = collection.insert_one(data)
    return jsonify({'id': str(result.inserted_id)})

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Python** : Integracja z pymongo
2. **Pipeline** : ETL z MongoDB
3. **Agregacja** : Analiza danych
4. **API** : REST z MongoDB
5. **Portfolio** : Projekty demonstrowalne

## 🔗 Zasoby

- [Dokumentacja PyMongo](https://pymongo.readthedocs.io/)
- [Przykłady MongoDB](https://github.com/mongodb/mongo-python-driver)

---

**Gratulacje !** Ukończyłeś szkolenie MongoDB. Możesz teraz używać MongoDB w projektach Data Analyst.

