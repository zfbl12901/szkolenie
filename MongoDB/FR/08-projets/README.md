# 8. Projets pratiques MongoDB

## 🎯 Objectifs

- Créer une application Python avec MongoDB
- Pipeline de données avec MongoDB
- Analyse de données
- Projets pour portfolio

## 📋 Table des matières

1. [Projet 1 : Application Python](#projet-1--application-python)
2. [Projet 2 : Pipeline de données](#projet-2--pipeline-de-données)
3. [Projet 3 : Analyse de données](#projet-3--analyse-de-données)
4. [Projet 4 : API REST](#projet-4--api-rest)

---

## Projet 1 : Application Python

### Objectif

Créer une application Python qui utilise MongoDB.

### Structure

```
mongodb-app/
├── requirements.txt
├── app.py
└── config.py
```

### Code Python

```python
from pymongo import MongoClient
from datetime import datetime

# Connexion
client = MongoClient('mongodb://localhost:27017/')
db = client['mydb']
collection = db['users']

# Insérer un utilisateur
def create_user(name, email):
    user = {
        'name': name,
        'email': email,
        'created_at': datetime.now()
    }
    result = collection.insert_one(user)
    return result.inserted_id

# Trouver un utilisateur
def find_user(email):
    return collection.find_one({'email': email})

# Mettre à jour
def update_user(email, updates):
    return collection.update_one(
        {'email': email},
        {'$set': updates}
    )

# Utilisation
create_user('John', 'john@example.com')
user = find_user('john@example.com')
print(user)
```

### requirements.txt

```
pymongo==4.6.0
```

---

## Projet 2 : Pipeline de données

### Objectif

Créer un pipeline ETL avec MongoDB.

### Code

```python
from pymongo import MongoClient
import pandas as pd

client = MongoClient('mongodb://localhost:27017/')
db = client['etl_db']

# Extract : Lire depuis CSV
def extract():
    df = pd.read_csv('data.csv')
    return df

# Transform : Transformer les données
def transform(df):
    df['processed_at'] = pd.Timestamp.now()
    df = df.dropna()
    return df

# Load : Charger dans MongoDB
def load(df):
    records = df.to_dict('records')
    db.data.insert_many(records)

# Pipeline complet
df = extract()
df = transform(df)
load(df)
```

---

## Projet 3 : Analyse de données

### Objectif

Analyser des données avec MongoDB Aggregation.

### Code

```python
from pymongo import MongoClient

client = MongoClient('mongodb://localhost:27017/')
db = client['analytics_db']

# Pipeline d'agrégation
pipeline = [
    # Filtrer par date
    {
        '$match': {
            'date': {
                '$gte': datetime(2024, 1, 1),
                '$lt': datetime(2024, 2, 1)
            }
        }
    },
    # Grouper par produit
    {
        '$group': {
            '_id': '$product',
            'total_sales': {'$sum': '$amount'},
            'count': {'$sum': 1},
            'average': {'$avg': '$amount'}
        }
    },
    # Trier
    {
        '$sort': {'total_sales': -1}
    },
    # Limiter
    {
        '$limit': 10
    }
]

# Exécuter
results = db.sales.aggregate(pipeline)
for result in results:
    print(result)
```

---

## Projet 4 : API REST

### Objectif

Créer une API REST avec Flask et MongoDB.

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

@app.route('/items/<item_id>', methods=['GET'])
def get_item(item_id):
    item = collection.find_one({'_id': ObjectId(item_id)}, {'_id': 0})
    return jsonify(item)

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 📊 Points clés à retenir

1. **Python** : Intégration avec pymongo
2. **Pipeline** : ETL avec MongoDB
3. **Agrégation** : Analyse de données
4. **API** : REST avec MongoDB
5. **Portfolio** : Projets démontrables

## 🔗 Ressources

- [PyMongo Documentation](https://pymongo.readthedocs.io/)
- [MongoDB Examples](https://github.com/mongodb/mongo-python-driver)

---

**Félicitations !** Vous avez terminé la formation MongoDB. Vous pouvez maintenant utiliser MongoDB dans vos projets de Data Analyst.

