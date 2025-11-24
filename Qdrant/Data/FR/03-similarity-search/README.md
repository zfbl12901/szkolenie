# 3. Recherche par Similarité

## 🎯 Objectifs

- Effectuer des recherches par similarité
- Comprendre les algorithmes de recherche
- Utiliser les paramètres de recherche
- Optimiser les performances
- Comprendre et interpréter les scores
- Utiliser la recherche batch

## 📋 Table des matières

1. [Recherche simple](#recherche-simple)
2. [Comprendre les scores](#comprendre-les-scores)
3. [Paramètres de recherche](#paramètres-de-recherche)
4. [Recherche avec filtres](#recherche-avec-filtres)
5. [Recherche batch](#recherche-batch)
6. [Recherche par ID](#recherche-par-id)
7. [Recherche recommandée](#recherche-recommandée)
8. [Optimisation](#optimisation)

---

## Recherche simple

### Recherche basique

```python
from qdrant_client import QdrantClient

client = QdrantClient(host="localhost", port=6333)

# Vecteur de requête (même dimension que les vecteurs de la collection)
query_vector = [0.1, 0.2, 0.3, ...]  # Dimension 128

# Recherche
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    limit=10  # Nombre de résultats
)

# Afficher les résultats
for result in results:
    print(f"ID: {result.id}")
    print(f"Score: {result.score:.4f}")
    print(f"Payload: {result.payload}")
    print("---")
```

### Structure des résultats

```python
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    limit=5
)

# Chaque résultat est un ScoredPoint
for result in results:
    # ID du point
    point_id = result.id
    
    # Score de similarité (0-1 pour COSINE, 0-∞ pour EUCLID)
    similarity_score = result.score
    
    # Payload (métadonnées)
    metadata = result.payload
    
    # Vecteur (si demandé)
    vector = result.vector  # None par défaut
```

---

## Comprendre les scores

### Score COSINE

Pour la distance **COSINE**, le score est entre **0 et 1** :
- **1.0** : Vecteurs identiques (même direction)
- **0.9-1.0** : Très similaires
- **0.7-0.9** : Similaires
- **0.5-0.7** : Modérément similaires
- **0.0-0.5** : Peu similaires
- **0.0** : Orthogonaux (perpendiculaires)

```python
# Exemple d'interprétation
for result in results:
    score = result.score
    
    if score > 0.9:
        print(f"Très similaire: {result.id} (score: {score:.4f})")
    elif score > 0.7:
        print(f"Similaire: {result.id} (score: {score:.4f})")
    elif score > 0.5:
        print(f"Modérément similaire: {result.id} (score: {score:.4f})")
    else:
        print(f"Peu similaire: {result.id} (score: {score:.4f})")
```

### Score EUCLID

Pour la distance **EUCLID**, le score est la distance (0 à ∞) :
- **0.0** : Vecteurs identiques
- **Plus petit = plus similaire**
- Pas de limite supérieure

```python
# Pour EUCLID, trier par score croissant
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    limit=10
)

# Le premier résultat a la plus petite distance (plus similaire)
for i, result in enumerate(results):
    print(f"Rang {i+1}: ID={result.id}, Distance={result.score:.4f}")
```

### Score DOT

Pour le **produit scalaire**, le score peut être négatif ou positif :
- **Plus grand = plus similaire**
- Peut être négatif si vecteurs opposés

---

## Paramètres de recherche

### Avec vecteur de requête

```python
results = client.search(
    collection_name="products",
    query_vector=query_vector,  # Vecteur de requête
    limit=10,  # Nombre de résultats
    score_threshold=0.7,  # Score minimum (optionnel)
    with_payload=True,  # Inclure le payload (défaut: True)
    with_vectors=False  # Inclure les vecteurs (défaut: False)
)
```

### Recherche par ID (trouver des similaires)

```python
# Trouver des points similaires à un point existant
results = client.search(
    collection_name="products",
    query_vector=None,  # Pas de vecteur
    query_filter=Filter(
        must=[FieldCondition(key="id", match=MatchValue(value=123))]
    ),
    limit=10,
    using="default"  # Nom du vecteur (si multivector)
)
```

### Score threshold

```python
# Filtrer par score minimum
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    limit=100,  # Chercher jusqu'à 100
    score_threshold=0.8  # Ne retourner que score >= 0.8
)

# Résultats filtrés
print(f"Résultats avec score >= 0.8: {len(results)}")
```

### Avec vecteurs

```python
# Inclure les vecteurs dans les résultats
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    limit=5,
    with_vectors=True  # Inclure les vecteurs
)

for result in results:
    print(f"ID: {result.id}")
    print(f"Vector: {result.vector[:5]}...")  # Afficher les 5 premiers
```

---

## Recherche avec filtres

### Filtre simple

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

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

### Filtres multiples (ET)

```python
from qdrant_client.models import Range

# Tous les critères doivent être satisfaits (ET)
filter = Filter(
    must=[
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(
            key="price",
            range=Range(gte=100, lte=500)
        ),
        FieldCondition(key="in_stock", match=MatchValue(value=True))
    ]
)

results = client.search(
    collection_name="products",
    query_vector=query_vector,
    query_filter=filter,
    limit=10
)
```

### Filtres OU

```python
# Au moins un critère doit être satisfait (OU)
filter = Filter(
    should=[
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(key="category", match=MatchValue(value="books")),
        FieldCondition(key="category", match=MatchValue(value="clothing"))
    ],
    min_should_match=1  # Au moins 1 doit correspondre
)

results = client.search(
    collection_name="products",
    query_vector=query_vector,
    query_filter=filter,
    limit=10
)
```

### Filtres NOT

```python
# Exclure certains points
filter = Filter(
    must_not=[
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(
            key="price",
            range=Range(lt=50)  # Exclure les produits < 50€
        )
    ]
)

results = client.search(
    collection_name="products",
    query_vector=query_vector,
    query_filter=filter,
    limit=10
)
```

### Filtres complexes

```python
# Combinaison complexe
filter = Filter(
    must=[
        # Doit être en stock
        FieldCondition(key="in_stock", match=MatchValue(value=True))
    ],
    should=[
        # OU électronique OU livres
        FieldCondition(key="category", match=MatchValue(value="electronics")),
        FieldCondition(key="category", match=MatchValue(value="books"))
    ],
    must_not=[
        # Mais pas les produits < 20€
        FieldCondition(key="price", range=Range(lt=20))
    ],
    min_should_match=1
)
```

---

## Recherche batch

### Recherche multiple

```python
# Rechercher avec plusieurs vecteurs de requête
query_vectors = [
    [0.1, 0.2, 0.3, ...],  # Requête 1
    [0.4, 0.5, 0.6, ...],  # Requête 2
    [0.7, 0.8, 0.9, ...]   # Requête 3
]

# Recherche batch
batch_results = client.search_batch(
    collection_name="products",
    requests=[
        {
            "vector": query_vector,
            "limit": 10,
            "filter": None  # Optionnel
        }
        for query_vector in query_vectors
    ]
)

# Résultats pour chaque requête
for i, results in enumerate(batch_results):
    print(f"Résultats pour requête {i+1}:")
    for result in results:
        print(f"  ID: {result.id}, Score: {result.score:.4f}")
```

### Recherche batch avec filtres différents

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

batch_results = client.search_batch(
    collection_name="products",
    requests=[
        {
            "vector": query_vector,
            "limit": 10,
            "filter": Filter(
                must=[FieldCondition(key="category", match=MatchValue(value="electronics"))]
            )
        },
        {
            "vector": query_vector,
            "limit": 10,
            "filter": Filter(
                must=[FieldCondition(key="category", match=MatchValue(value="books"))]
            )
        }
    ]
)
```

---

## Recherche recommandée

### Recommandation basique

```python
def recommend_similar_items(item_id, limit=10):
    """Recommander des items similaires à un item donné"""
    
    # 1. Récupérer le vecteur de l'item
    points = client.retrieve(
        collection_name="products",
        ids=[item_id],
        with_vectors=True
    )
    
    if not points:
        return []
    
    item_vector = points[0].vector
    
    # 2. Rechercher des items similaires (exclure l'item original)
    results = client.search(
        collection_name="products",
        query_vector=item_vector,
        query_filter=Filter(
            must_not=[
                FieldCondition(key="id", match=MatchValue(value=item_id))
            ]
        ),
        limit=limit
    )
    
    return results

# Utilisation
recommendations = recommend_similar_items(item_id=123, limit=5)
for rec in recommendations:
    print(f"Recommandé: {rec.payload['name']} (score: {rec.score:.4f})")
```

### Recommandation hybride

```python
def hybrid_recommendation(item_id, user_preferences, limit=10):
    """Recommandation combinant similarité et préférences utilisateur"""
    
    # 1. Vecteur de l'item
    points = client.retrieve(
        collection_name="products",
        ids=[item_id],
        with_vectors=True
    )
    item_vector = points[0].vector
    
    # 2. Recherche avec filtres de préférences
    results = client.search(
        collection_name="products",
        query_vector=item_vector,
        query_filter=Filter(
            must=[
                # Préférences utilisateur
                FieldCondition(
                    key="category",
                    match=MatchValue(value=user_preferences["preferred_category"])
                ),
                FieldCondition(
                    key="price",
                    range=Range(
                        gte=user_preferences["min_price"],
                        lte=user_preferences["max_price"]
                    )
                )
            ],
            must_not=[
                FieldCondition(key="id", match=MatchValue(value=item_id))
            ]
        ),
        limit=limit,
        score_threshold=0.7  # Score minimum
    )
    
    return results
```

---

## Recherche par ID

### Récupérer des points par ID

```python
# Récupérer des points spécifiques
points = client.retrieve(
    collection_name="products",
    ids=[1, 2, 3, 4, 5],
    with_payload=True,
    with_vectors=False
)

for point in points:
    print(f"ID: {point.id}")
    print(f"Payload: {point.payload}")
```

### Scroll (parcourir tous les points)

```python
# Parcourir tous les points (par batches)
scroll_result = client.scroll(
    collection_name="products",
    limit=100,  # Nombre de points par batch
    with_payload=True,
    with_vectors=False
)

points, next_page_offset = scroll_result

# Continuer avec le prochain batch
while next_page_offset is not None:
    scroll_result = client.scroll(
        collection_name="products",
        limit=100,
        offset=next_page_offset,
        with_payload=True,
        with_vectors=False
    )
    points, next_page_offset = scroll_result
```

---

## Optimisation

### Paramètre ef (exactness factor)

```python
# Augmenter la précision de recherche (plus lent)
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    limit=10,
    ef=128  # Plus élevé = plus précis mais plus lent (défaut: auto)
)
```

### Recherche avec index

```python
# Utiliser un index spécifique
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    limit=10,
    using="default"  # Nom du vecteur (pour collections multivector)
)
```

### Pré-filtrage vs Post-filtrage

```python
# Pré-filtrage (recommandé si peu de points après filtrage)
# Filtre d'abord, puis recherche vectorielle
results = client.search(
    collection_name="products",
    query_vector=query_vector,
    query_filter=filter,  # Filtre appliqué AVANT la recherche
    limit=10
)

# Post-filtrage (si beaucoup de points après filtrage)
# Recherche vectorielle d'abord, puis filtre
# Utiliser un limit plus élevé puis filtrer manuellement
```

---

## Exercices pratiques

### Exercice 1 : Recherche avec seuil

Créer une fonction qui recherche des produits similaires avec un score minimum de 0.8.

**Solution :**

```python
def search_high_similarity(query_vector, min_score=0.8, limit=10):
    results = client.search(
        collection_name="products",
        query_vector=query_vector,
        limit=limit,
        score_threshold=min_score
    )
    return results
```

### Exercice 2 : Top-K par catégorie

Pour chaque catégorie, trouver les 5 produits les plus similaires à un vecteur de requête.

**Solution :**

```python
categories = ["electronics", "books", "clothing"]

for category in categories:
    results = client.search(
        collection_name="products",
        query_vector=query_vector,
        query_filter=Filter(
            must=[
                FieldCondition(key="category", match=MatchValue(value=category))
            ]
        ),
        limit=5
    )
    
    print(f"\nTop 5 {category}:")
    for result in results:
        print(f"  {result.payload['name']} (score: {result.score:.4f})")
```

---

## 🎯 Points clés à retenir

✅ Le score COSINE est entre 0 et 1 (1 = identique)  
✅ Utiliser score_threshold pour filtrer par qualité  
✅ Les filtres peuvent être combinés (must, should, must_not)  
✅ La recherche batch est efficace pour plusieurs requêtes  
✅ Le paramètre ef contrôle le compromis précision/vitesse  

---

**Prochaine étape :** [Filtres et Métadonnées](./04-filters-metadata/README.md)
