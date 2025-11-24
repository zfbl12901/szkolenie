# 4. Filtres et Métadonnées

## 🎯 Objectifs

- Utiliser les filtres avancés
- Gérer les métadonnées (payload)
- Combiner plusieurs conditions
- Optimiser les requêtes filtrées
- Comprendre les types de filtres
- Indexer les champs pour les filtres

## 📋 Table des matières

1. [Introduction aux filtres](#introduction-aux-filtres)
2. [Types de filtres](#types-de-filtres)
3. [Opérateurs de filtrage](#opérateurs-de-filtrage)
4. [Filtres complexes](#filtres-complexes)
5. [Gestion du payload](#gestion-du-payload)
6. [Index de payload](#index-de-payload)
7. [Optimisation des filtres](#optimisation-des-filtres)

---

## Introduction aux filtres

### Qu'est-ce qu'un filtre ?

Un **filtre** permet de restreindre la recherche vectorielle à un sous-ensemble de points basé sur leurs métadonnées (payload) :

- **Avant filtrage** : Recherche sur tous les points
- **Après filtrage** : Recherche uniquement sur les points correspondants

### Pourquoi utiliser des filtres ?

- **Performance** : Réduire l'espace de recherche
- **Pertinence** : Retourner uniquement les résultats pertinents
- **Flexibilité** : Combiner recherche vectorielle et filtres métier

### Exemple simple

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

# Rechercher des produits électroniques similaires
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    query_filter=Filter(
        must=[
            FieldCondition(
                key="category",
                match=MatchValue(value="electronics")
            )
        ]
    ),
    limit=10
)
```

---

## Types de filtres

### Match (valeur exacte)

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

# String exact
filter = Filter(
    must=[
        FieldCondition(
            key="category",
            match=MatchValue(value="electronics")
        )
    ]
)

# Number exact
filter = Filter(
    must=[
        FieldCondition(
            key="product_id",
            match=MatchValue(value=12345)
        )
    ]
)

# Boolean
filter = Filter(
    must=[
        FieldCondition(
            key="in_stock",
            match=MatchValue(value=True)
        )
    ]
)
```

### Match Any (plusieurs valeurs)

```python
from qdrant_client.models import MatchAny

# Correspond à n'importe quelle valeur de la liste
filter = Filter(
    must=[
        FieldCondition(
            key="category",
            match=MatchAny(any=["electronics", "books", "clothing"])
        )
    ]
)
```

### Match Text (recherche de texte)

```python
from qdrant_client.models import MatchText

# Recherche de texte (contient)
filter = Filter(
    must=[
        FieldCondition(
            key="description",
            match=MatchText(text="laptop")
        )
    ]
)
```

### Range (plage de valeurs)

```python
from qdrant_client.models import Range

# Plage numérique
filter = Filter(
    must=[
        FieldCondition(
            key="price",
            range=Range(
                gte=100,  # Greater than or equal
                lte=500   # Less than or equal
            )
        )
    ]
)

# Seulement minimum
filter = Filter(
    must=[
        FieldCondition(
            key="price",
            range=Range(gte=100)
        )
    ]
)

# Seulement maximum
filter = Filter(
    must=[
        FieldCondition(
            key="price",
            range=Range(lte=500)
        )
    ]
)

# Entre deux valeurs
filter = Filter(
    must=[
        FieldCondition(
            key="price",
            range=Range(gt=100, lt=500)  # Strictement entre
        )
    ]
)
```

### Geo (géolocalisation)

```python
from qdrant_client.models import GeoRadius

# Points dans un rayon (coordonnées GPS)
filter = Filter(
    must=[
        FieldCondition(
            key="location",
            geo_radius=GeoRadius(
                center={"lat": 48.8566, "lon": 2.3522},  # Paris
                radius=1000  # 1km en mètres
            )
        )
    ]
)
```

### Is Null / Is Empty

```python
from qdrant_client.models import IsNullCondition

# Vérifier si un champ est null ou absent
filter = Filter(
    must=[
        IsNullCondition(
            is_null=FieldCondition(key="discount")
        )
    ]
)
```

---

## Opérateurs de filtrage

### MUST (ET logique)

Tous les critères doivent être satisfaits :

```python
filter = Filter(
    must=[
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(key="in_stock", match=MatchValue(value=True)),
        FieldCondition(key="price", range=Range(gte=100, lte=500))
    ]
)

# Équivalent à: category="electronics" AND in_stock=True AND price BETWEEN 100 AND 500
```

### SHOULD (OU logique)

Au moins un critère doit être satisfait :

```python
filter = Filter(
    should=[
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(key="category", match=MatchValue(value="books")),
        FieldCondition(key="category", match=MatchValue(value="clothing"))
    ],
    min_should_match=1  # Au moins 1 doit correspondre
)

# Équivalent à: category="electronics" OR category="books" OR category="clothing"
```

### MUST_NOT (NON logique)

Exclure les points correspondants :

```python
filter = Filter(
    must_not=[
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(key="price", range=Range(lt=50))
    ]
)

# Équivalent à: NOT (category="electronics" OR price < 50)
```

### Combinaison complexe

```python
# (category="electronics" OR category="books") 
# AND price BETWEEN 100 AND 500 
# AND NOT (in_stock=False)

filter = Filter(
    must=[
        # ET: prix entre 100 et 500
        FieldCondition(key="price", range=Range(gte=100, lte=500))
    ],
    should=[
        # OU: électronique OU livres
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(key="category", match=MatchValue(value="books"))
    ],
    must_not=[
        # NON: pas en rupture de stock
        FieldCondition(key="in_stock", match=MatchValue(value=False))
    ],
    min_should_match=1  # Au moins une catégorie doit correspondre
)
```

---

## Filtres complexes

### Filtres imbriqués

```python
# (category="electronics" AND price>100) OR (category="books" AND price<50)
filter = Filter(
    should=[
        Filter(
            must=[
                FieldCondition(key="category", match=MatchValue(value="electronics")),
                FieldCondition(key="price", range=Range(gt=100))
            ]
        ),
        Filter(
            must=[
                FieldCondition(key="category", match=MatchValue(value="books")),
                FieldCondition(key="price", range=Range(lt=50))
            ]
        )
    ],
    min_should_match=1
)
```

### Filtres sur tableaux

```python
# Vérifier si un élément est dans un tableau
filter = Filter(
    must=[
        FieldCondition(
            key="tags",
            match=MatchValue(value="gaming")  # "gaming" dans le tableau tags
        )
    ]
)

# Plusieurs tags
filter = Filter(
    must=[
        FieldCondition(
            key="tags",
            match=MatchAny(any=["gaming", "laptop", "professional"])
        )
    ]
)
```

### Filtres sur objets imbriqués

```python
# Payload: {"metadata": {"brand": "Apple", "model": "MacBook"}}
filter = Filter(
    must=[
        FieldCondition(
            key="metadata.brand",  # Accès aux champs imbriqués
            match=MatchValue(value="Apple")
        )
    ]
)
```

---

## Gestion du payload

### Structure du payload

```python
payload = {
    # Types simples
    "name": "Product A",           # String
    "price": 99.99,                # Number (float/int)
    "quantity": 10,                # Integer
    "in_stock": True,              # Boolean
    
    # Tableaux
    "tags": ["electronics", "laptop", "gaming"],
    "sizes": [10, 12, 14],
    
    # Objets
    "metadata": {
        "brand": "BrandX",
        "model": "Model123",
        "specs": {
            "cpu": "Intel i7",
            "ram": "16GB"
        }
    },
    
    # Dates (comme string ISO)
    "created_at": "2024-01-15T10:00:00Z",
    "updated_at": "2024-01-20T15:30:00Z"
}
```

### Insérer avec payload

```python
from qdrant_client.models import PointStruct

point = PointStruct(
    id=1,
    vector=[0.1, 0.2, 0.3, ...],
    payload={
        "name": "Laptop Gaming",
        "category": "electronics",
        "price": 1299.99,
        "in_stock": True,
        "tags": ["gaming", "laptop", "high-performance"],
        "metadata": {
            "brand": "BrandX",
            "model": "GamingPro-2024"
        }
    }
)

client.upsert(collection_name="products", points=[point])
```

### Mettre à jour le payload

```python
# Ajouter/modifier des champs
client.set_payload(
    collection_name="products",
    payload={
        "discount": 0.15,  # Nouveau champ
        "on_sale": True,   # Nouveau champ
        "price": 1099.99   # Mise à jour
    },
    points=[1, 2, 3]  # IDs des points à mettre à jour
)

# Mettre à jour tous les points d'une collection (avec filtre)
client.set_payload(
    collection_name="products",
    payload={"last_updated": "2024-01-20"},
    points=None,  # Tous les points
    key="category",
    match=MatchValue(value="electronics")  # Seulement électronique
)
```

### Supprimer des champs du payload

```python
# Supprimer des clés spécifiques
client.delete_payload(
    collection_name="products",
    keys=["discount", "on_sale"],  # Clés à supprimer
    points=[1, 2, 3]
)

# Supprimer pour tous les points
client.delete_payload(
    collection_name="products",
    keys=["temporary_field"],
    points=None  # Tous les points
)
```

### Récupérer le payload

```python
# Récupérer des points avec leur payload
points = client.retrieve(
    collection_name="products",
    ids=[1, 2, 3],
    with_payload=True,  # Inclure le payload
    with_vectors=False
)

for point in points:
    print(f"ID: {point.id}")
    print(f"Payload: {point.payload}")
    print(f"Name: {point.payload.get('name')}")
    print(f"Price: {point.payload.get('price')}")
```

### Overwrite payload (remplacer complètement)

```python
# Remplacer complètement le payload
client.overwrite_payload(
    collection_name="products",
    payload={
        "name": "New Name",
        "price": 999.99
        # Tous les autres champs sont supprimés
    },
    points=[1]
)
```

---

## Index de payload

### Pourquoi indexer le payload ?

Les **index de payload** accélèrent les filtres :
- **Sans index** : Scan complet de tous les points
- **Avec index** : Recherche rapide dans l'index

### Créer un index

```python
from qdrant_client.models import PayloadSchemaType

# Index sur un champ string
client.create_payload_index(
    collection_name="products",
    field_name="category",
    field_schema=PayloadSchemaType.KEYWORD  # Pour valeurs exactes
)

# Index sur un champ numérique
client.create_payload_index(
    collection_name="products",
    field_name="price",
    field_schema=PayloadSchemaType.FLOAT  # Pour ranges
)

# Index sur un champ integer
client.create_payload_index(
    collection_name="products",
    field_name="product_id",
    field_schema=PayloadSchemaType.INTEGER
)

# Index géospatial
client.create_payload_index(
    collection_name="products",
    field_name="location",
    field_schema=PayloadSchemaType.GEO
)
```

### Types d'index

- **KEYWORD** : Pour valeurs exactes (strings)
- **INTEGER** : Pour nombres entiers
- **FLOAT** : Pour nombres décimaux
- **GEO** : Pour coordonnées géographiques
- **BOOL** : Pour valeurs booléennes

### Vérifier les index

```python
# Obtenir les informations de la collection
collection_info = client.get_collection("products")

# Voir les index de payload
print(collection_info.payload_schema)
```

### Supprimer un index

```python
client.delete_payload_index(
    collection_name="products",
    field_name="category"
)
```

---

## Optimisation des filtres

### Pré-filtrage vs Post-filtrage

**Pré-filtrage (recommandé si peu de résultats après filtrage) :**
```python
# Filtre appliqué AVANT la recherche vectorielle
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    query_filter=filter,  # Filtre d'abord
    limit=10
)
```

**Post-filtrage (si beaucoup de résultats après filtrage) :**
```python
# Recherche vectorielle d'abord, puis filtre
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    limit=100  # Plus de résultats
)

# Filtrer manuellement
filtered_results = [
    r for r in results 
    if r.payload.get("category") == "electronics" 
    and 100 <= r.payload.get("price", 0) <= 500
]
```

### Indexer les champs fréquemment filtrés

```python
# Indexer les champs utilisés dans les filtres
client.create_payload_index(
    collection_name="products",
    field_name="category",
    field_schema=PayloadSchemaType.KEYWORD
)

client.create_payload_index(
    collection_name="products",
    field_name="price",
    field_schema=PayloadSchemaType.FLOAT
)
```

### Ordre des conditions dans MUST

```python
# Mettre les conditions les plus sélectives en premier
filter = Filter(
    must=[
        # 1. Condition la plus sélective (peu de résultats)
        FieldCondition(key="product_id", match=MatchValue(value=12345)),
        # 2. Condition moins sélective
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        # 3. Condition la moins sélective
        FieldCondition(key="in_stock", match=MatchValue(value=True))
    ]
)
```

---

## Exercices pratiques

### Exercice 1 : Filtre complexe

Créer un filtre pour trouver des produits :
- Catégorie "electronics" OU "books"
- Prix entre 50 et 200
- En stock
- Pas de produits avec tag "discontinued"

**Solution :**

```python
filter = Filter(
    must=[
        FieldCondition(key="price", range=Range(gte=50, lte=200)),
        FieldCondition(key="in_stock", match=MatchValue(value=True))
    ],
    should=[
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(key="category", match=MatchValue(value="books"))
    ],
    must_not=[
        FieldCondition(key="tags", match=MatchValue(value="discontinued"))
    ],
    min_should_match=1
)
```

### Exercice 2 : Mise à jour conditionnelle

Mettre à jour le prix de tous les produits électroniques avec une réduction de 10%.

**Solution :**

```python
# 1. Récupérer tous les produits électroniques
scroll_result = client.scroll(
    collection_name="products",
    scroll_filter=Filter(
        must=[
            FieldCondition(key="category", match=MatchValue(value="electronics"))
        ]
    ),
    limit=100,
    with_payload=True,
    with_vectors=False
)

points, next_offset = scroll_result
updated_points = []

for point in points:
    old_price = point.payload.get("price", 0)
    new_price = old_price * 0.9  # Réduction de 10%
    
    # Mettre à jour le payload
    client.set_payload(
        collection_name="products",
        payload={"price": round(new_price, 2)},
        points=[point.id]
    )
```

---

## 🎯 Points clés à retenir

✅ Les filtres restreignent la recherche aux points correspondants  
✅ MUST = ET, SHOULD = OU, MUST_NOT = NON  
✅ Indexer les champs fréquemment filtrés pour de meilleures performances  
✅ Pré-filtrage si peu de résultats, post-filtrage si beaucoup  
✅ Le payload peut contenir des types simples, tableaux et objets imbriqués  

---

**Prochaine étape :** [Intégration Python](./05-python-integration/README.md)
