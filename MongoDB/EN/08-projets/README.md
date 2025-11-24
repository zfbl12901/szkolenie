# 8. MongoDB Practical Projects

## 🎯 Objectives

- Create Python application with MongoDB
- Data pipeline with MongoDB
- Data analysis
- Portfolio projects

## 📋 Table of Contents

1. [Project 1 : Python Application](#project-1--python-application)
2. [Project 2 : Data Pipeline](#project-2--data-pipeline)
3. [Project 3 : Data Analysis](#project-3--data-analysis)
4. [Project 4 : REST API](#project-4--rest-api)

---

## Project 1 : Python Application

### Objective

Create a Python application using MongoDB.

### Python Code

```python
from pymongo import MongoClient
from datetime import datetime

# Connection
client = MongoClient('mongodb://localhost:27017/')
db = client['mydb']
collection = db['users']

# Insert user
def create_user(name, email):
    user = {
        'name': name,
        'email': email,
        'created_at': datetime.now()
    }
    result = collection.insert_one(user)
    return result.inserted_id

# Find user
def find_user(email):
    return collection.find_one({'email': email})

# Usage
create_user('John', 'john@example.com')
user = find_user('john@example.com')
print(user)
```

---

## Project 2 : Data Pipeline

### Objective

Create an ETL pipeline with MongoDB.

### Code

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

# Complete pipeline
df = extract()
df = transform(df)
load(df)
```

---

## Project 3 : Data Analysis

### Objective

Analyze data with MongoDB Aggregation.

### Code

```python
from pymongo import MongoClient

client = MongoClient('mongodb://localhost:27017/')
db = client['analytics_db']

# Aggregation pipeline
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

# Execute
results = db.sales.aggregate(pipeline)
for result in results:
    print(result)
```

---

## Project 4 : REST API

### Objective

Create a REST API with Flask and MongoDB.

### Code

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

## 📊 Key Takeaways

1. **Python** : Integration with pymongo
2. **Pipeline** : ETL with MongoDB
3. **Aggregation** : Data analysis
4. **API** : REST with MongoDB
5. **Portfolio** : Demonstrable projects

## 🔗 Resources

- [PyMongo Documentation](https://pymongo.readthedocs.io/)
- [MongoDB Examples](https://github.com/mongodb/mongo-python-driver)

---

**Congratulations!** You have completed the MongoDB training. You can now use MongoDB in your Data Analyst projects.

