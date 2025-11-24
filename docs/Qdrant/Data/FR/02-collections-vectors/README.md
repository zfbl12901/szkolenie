# 2. Collections et Vecteurs

## 🎯 Objectifs

- Créer et gérer des collections
- Comprendre les configurations de collections
- Insérer des vecteurs efficacement
- Comprendre les distances et métriques
- Gérer les métadonnées (payload)
- Optimiser les collections

## 📋 Table des matières

1. [Collections](#collections)
2. [Configuration des collections](#configuration-des-collections)
3. [Types de distances](#types-de-distances)
4. [Insertion de vecteurs](#insertion-de-vecteurs)
5. [Gestion des collections](#gestion-des-collections)
6. [Métadonnées (Payload)](#métadonnées-payload)
7. [Optimisation](#optimisation)

---

## Collections

### Qu'est-ce qu'une collection ?

Une **collection** est un conteneur pour des points (vecteurs) similaires :
- Même dimension de vecteurs
- Même type de distance
- Partage des mêmes index

### Structure d'une collection

```
Collection "products"
├── Configuration
│   ├── Dimension: 128
│   ├── Distance: COSINE
│   └── Index: HNSW
└── Points
    ├── Point 1: [vector, payload]
    ├── Point 2: [vector, payload]
    └── ...
```

---

## Configuration des collections

### Création basique

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams

client = QdrantClient(host="localhost", port=6333)

# Collection simple
client.create_collection(
    collection_name="products",
    vectors_config=VectorParams(
        size=128,  # Dimension des vecteurs
        distance=Distance.COSINE  # Type de distance
    )
)
```

### Configuration avancée

```python
from qdrant_client.models import (
    Distance, VectorParams, HnswConfigDiff, 
    OptimizersConfigDiff, QuantizationConfig
)

client.create_collection(
    collection_name="products_advanced",
    vectors_config=VectorParams(
        size=384,  # Dimension (ex: sentence-transformers)
        distance=Distance.COSINE
    ),
    # Configuration HNSW (index)
    hnsw_config=HnswConfigDiff(
        m=16,  # Nombre de connexions (plus = plus précis, plus lent)
        ef_construct=100,  # Précision de construction
        full_scan_threshold=10000  # Seuil pour scan complet
    ),
    # Configuration des optimiseurs
    optimizers_config=OptimizersConfigDiff(
        indexing_threshold=20000,  # Seuil pour indexation
        memmap_threshold=50000  # Seuil pour memmap
    ),
    # Quantisation (compression)
    quantization_config=QuantizationConfig(
        scalar=ScalarQuantization(
            type=ScalarType.INT8,
            quantile=0.99,
            always_ram=True
        )
    )
)
```

### Collections avec plusieurs vecteurs

```python
# Collection avec plusieurs vecteurs par point
client.create_collection(
    collection_name="multivector",
    vectors_config={
        "image": VectorParams(size=512, distance=Distance.COSINE),
        "text": VectorParams(size=384, distance=Distance.COSINE)
    }
)
```

---

## Types de distances

### Distance COSINE (recommandé pour textes)

**Utilisation :** Embeddings normalisés, textes, recherche sémantique

```python
VectorParams(size=128, distance=Distance.COSINE)
```

**Caractéristiques :**
- Mesure l'angle entre vecteurs
- Score entre 0 et 1 (1 = identique)
- Insensible à la magnitude
- Idéal pour embeddings normalisés

**Exemple :**
```python
# Vecteurs normalisés
v1 = [0.5, 0.5, 0.5, 0.5]  # Norme = 1
v2 = [1.0, 1.0, 1.0, 1.0]  # Norme = 2
# Similarité cosinus = 1.0 (même direction)
```

### Distance EUCLID

**Utilisation :** Données numériques, coordonnées, features brutes

```python
VectorParams(size=128, distance=Distance.EUCLID)
```

**Caractéristiques :**
- Distance géométrique dans l'espace
- Score entre 0 et ∞ (0 = identique)
- Sensible à la magnitude
- Idéal pour données numériques

**Exemple :**
```python
# Coordonnées 2D
point1 = [0, 0]
point2 = [3, 4]
# Distance euclidienne = 5.0
```

### Distance DOT (produit scalaire)

**Utilisation :** Vecteurs non normalisés, scores pondérés

```python
VectorParams(size=128, distance=Distance.DOT)
```

**Caractéristiques :**
- Produit scalaire entre vecteurs
- Score entre -∞ et +∞
- Sensible à la magnitude
- Idéal pour scores pondérés

**Quand utiliser chaque distance ?**

| Distance | Cas d'usage | Exemple |
|----------|-------------|---------|
| **COSINE** | Textes, embeddings normalisés | Recherche sémantique |
| **EUCLID** | Coordonnées, features numériques | Classification d'images |
| **DOT** | Scores pondérés, recommandations | Système de scoring |

---

## Insertion de vecteurs

### Insertion simple

```python
from qdrant_client.models import PointStruct

# Un seul point
point = PointStruct(
    id=1,
    vector=[0.1, 0.2, 0.3, ...],  # Vecteur de dimension 128
    payload={
        "name": "Product A",
        "category": "electronics",
        "price": 99.99
    }
)

client.upsert(
    collection_name="products",
    points=[point]
)
```

### Insertion multiple (batch)

```python
import numpy as np
from qdrant_client.models import PointStruct

# Générer des vecteurs (exemple)
vectors = np.random.rand(1000, 128).tolist()

# Créer les points
points = [
    PointStruct(
        id=i,
        vector=vectors[i],
        payload={
            "name": f"Product {i}",
            "category": ["electronics", "books", "clothing"][i % 3],
            "price": round(10 + i * 0.5, 2)
        }
    )
    for i in range(1000)
]

# Insertion batch
client.upsert(
    collection_name="products",
    points=points
)
```

### Insertion avec wait

```python
# Attendre que l'insertion soit confirmée
client.upsert(
    collection_name="products",
    points=points,
    wait=True  # Attendre la confirmation
)
```

### Insertion avec ordering

```python
# Garantir l'ordre d'insertion
client.upsert(
    collection_name="products",
    points=points,
    ordering=WriteOrdering.STRONG  # Ordre garanti
)
```

### Gestion des IDs

```python
# IDs numériques
PointStruct(id=1, vector=..., payload=...)

# IDs UUID
import uuid
PointStruct(id=str(uuid.uuid4()), vector=..., payload=...)

# IDs personnalisés
PointStruct(id="product_123", vector=..., payload=...)
```

---

## Gestion des collections

### Lister les collections

```python
# Obtenir toutes les collections
collections = client.get_collections()
for collection in collections.collections:
    print(f"Collection: {collection.name}")
```

### Informations d'une collection

```python
# Obtenir les informations détaillées
collection_info = client.get_collection("products")

print(f"Points count: {collection_info.points_count}")
print(f"Vectors count: {collection_info.vectors_count}")
print(f"Indexed vectors: {collection_info.indexed_vectors_count}")
print(f"Status: {collection_info.status}")
```

### Vérifier l'existence

```python
# Vérifier si une collection existe
collections = client.get_collections()
collection_names = [c.name for c in collections.collections]

if "products" in collection_names:
    print("Collection existe")
else:
    print("Collection n'existe pas")
```

### Supprimer une collection

```python
# Supprimer une collection
client.delete_collection("products")

# Vérifier la suppression
collections = client.get_collections()
print("products" not in [c.name for c in collections.collections])
```

### Renommer une collection

```python
# Qdrant ne supporte pas directement le renommage
# Solution : créer une nouvelle collection et copier les données

# 1. Créer nouvelle collection
client.create_collection(
    collection_name="products_new",
    vectors_config=VectorParams(size=128, distance=Distance.COSINE)
)

# 2. Récupérer tous les points (par batches)
# 3. Insérer dans la nouvelle collection
# 4. Supprimer l'ancienne
```

---

## Métadonnées (Payload)

### Qu'est-ce qu'un payload ?

Le **payload** contient les métadonnées associées à un vecteur :
- Informations structurées
- Filtres possibles
- Pas utilisé pour la recherche vectorielle

### Types de payload

```python
PointStruct(
    id=1,
    vector=[...],
    payload={
        # String
        "name": "Product A",
        
        # Number
        "price": 99.99,
        "quantity": 10,
        
        # Boolean
        "in_stock": True,
        
        # Array
        "tags": ["electronics", "laptop", "gaming"],
        
        # Nested object
        "metadata": {
            "brand": "BrandX",
            "model": "Model123"
        },
        
        # Date (comme string ISO)
        "created_at": "2024-01-15T10:00:00Z"
    }
)
```

### Mettre à jour le payload

```python
# Mettre à jour le payload d'un point
client.set_payload(
    collection_name="products",
    payload={
        "discount": 0.1,
        "on_sale": True
    },
    points=[1, 2, 3]  # IDs des points
)
```

### Supprimer des champs du payload

```python
# Supprimer des clés du payload
client.delete_payload(
    collection_name="products",
    keys=["discount", "on_sale"],
    points=[1, 2, 3]
)
```

### Récupérer le payload

```python
# Récupérer des points avec leur payload
points = client.retrieve(
    collection_name="products",
    ids=[1, 2, 3],
    with_payload=True,
    with_vectors=False
)

for point in points:
    print(f"ID: {point.id}")
    print(f"Payload: {point.payload}")
```

---

## Optimisation

### Index HNSW

**HNSW** (Hierarchical Navigable Small World) est l'algorithme d'indexation :

```python
from qdrant_client.models import HnswConfigDiff

client.create_collection(
    collection_name="optimized",
    vectors_config=VectorParams(size=128, distance=Distance.COSINE),
    hnsw_config=HnswConfigDiff(
        m=16,  # Plus de connexions = plus précis mais plus lent
        ef_construct=100,  # Précision lors de la construction
        full_scan_threshold=10000  # Scan complet si < 10000 points
    )
)
```

### Paramètres HNSW

- **m** : Nombre de connexions (défaut: 16)
  - Plus élevé = plus précis, plus lent
  - Recommandé: 16-64
  
- **ef_construct** : Précision de construction (défaut: 100)
  - Plus élevé = meilleure qualité, plus lent
  - Recommandé: 100-200

### Quantisation

La **quantisation** réduit la taille mémoire :

```python
from qdrant_client.models import QuantizationConfig, ScalarQuantization, ScalarType

client.create_collection(
    collection_name="quantized",
    vectors_config=VectorParams(size=128, distance=Distance.COSINE),
    quantization_config=QuantizationConfig(
        scalar=ScalarQuantization(
            type=ScalarType.INT8,  # Réduction de 4x (float32 → int8)
            quantile=0.99,
            always_ram=True
        )
    )
)
```

**Avantages :**
- Réduction mémoire (4x)
- Recherche plus rapide
- Légère perte de précision

---

## Exercices pratiques

### Exercice 1 : Créer une collection optimisée

Créer une collection pour des embeddings de texte (384 dimensions) avec :
- Distance COSINE
- HNSW optimisé (m=32, ef_construct=200)
- Quantisation INT8

**Solution :**

```python
from qdrant_client.models import (
    Distance, VectorParams, HnswConfigDiff,
    QuantizationConfig, ScalarQuantization, ScalarType
)

client.create_collection(
    collection_name="text_embeddings",
    vectors_config=VectorParams(size=384, distance=Distance.COSINE),
    hnsw_config=HnswConfigDiff(m=32, ef_construct=200),
    quantization_config=QuantizationConfig(
        scalar=ScalarQuantization(
            type=ScalarType.INT8,
            quantile=0.99,
            always_ram=True
        )
    )
)
```

### Exercice 2 : Insérer 1000 produits

Créer 1000 points avec :
- IDs séquentiels
- Vecteurs aléatoires de dimension 128
- Payload avec name, category, price

**Solution :**

```python
import random
from qdrant_client.models import PointStruct

categories = ["electronics", "books", "clothing", "food", "sports"]
points = []

for i in range(1000):
    points.append(
        PointStruct(
            id=i,
            vector=[random.random() for _ in range(128)],
            payload={
                "name": f"Product {i}",
                "category": random.choice(categories),
                "price": round(random.uniform(10, 1000), 2)
            }
        )
    )

# Insertion par batches de 100
batch_size = 100
for i in range(0, len(points), batch_size):
    batch = points[i:i+batch_size]
    client.upsert(collection_name="products", points=batch)
    print(f"Inserted batch {i//batch_size + 1}")
```

---

## 🎯 Points clés à retenir

✅ Une collection regroupe des vecteurs de même dimension et distance  
✅ COSINE pour textes, EUCLID pour coordonnées, DOT pour scores  
✅ L'insertion batch est plus efficace que point par point  
✅ Le payload contient les métadonnées filtrables  
✅ HNSW et quantisation optimisent performances et mémoire  

---

**Prochaine étape :** [Recherche par Similarité](./03-similarity-search/README.md)

