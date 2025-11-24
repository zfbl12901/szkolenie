# 7. Bonnes Pratiques

## 🎯 Objectifs

- Optimiser les performances
- Gérer la dimension des vecteurs
- Choisir la bonne distance
- Structurer les métadonnées
- Gérer la mémoire
- Monitoring et maintenance

## 📋 Table des matières

1. [Dimension des vecteurs](#dimension-des-vecteurs)
2. [Choix de la distance](#choix-de-la-distance)
3. [Configuration HNSW](#configuration-hnsw)
4. [Structure des métadonnées](#structure-des-métadonnées)
5. [Gestion de la mémoire](#gestion-de-la-mémoire)
6. [Performance](#performance)
7. [Sécurité](#sécurité)
8. [Monitoring](#monitoring)

---

## Dimension des vecteurs

### Choisir la bonne dimension

La dimension affecte :
- **Performance** : Plus grande = plus lent
- **Qualité** : Plus grande = généralement meilleure
- **Mémoire** : Plus grande = plus de mémoire

### Recommandations par cas d'usage

| Cas d'usage | Dimension | Modèle exemple |
|-------------|-----------|----------------|
| **Textes courts** | 128-256 | MiniLM |
| **Textes moyens** | 384-512 | sentence-transformers |
| **Textes longs** | 768 | BERT base |
| **Textes avancés** | 1536 | OpenAI ada-002 |
| **Images** | 512-768 | CLIP, ResNet |
| **Multimodal** | 512-768 | CLIP |

### Impact sur les performances

```python
# Dimension 128 : Rapide, moins précis
VectorParams(size=128, distance=Distance.COSINE)

# Dimension 384 : Équilibré (recommandé)
VectorParams(size=384, distance=Distance.COSINE)

# Dimension 1536 : Lent, très précis
VectorParams(size=1536, distance=Distance.COSINE)
```

### Règle générale

- **< 1M points** : 384-512 dimensions
- **1M - 10M points** : 256-384 dimensions
- **> 10M points** : 128-256 dimensions

---

## Choix de la distance

### Distance COSINE (recommandé pour textes)

**Utilisation :**
- Embeddings normalisés
- Textes et recherche sémantique
- Recommandations

**Avantages :**
- Insensible à la magnitude
- Score entre 0 et 1 (facile à interpréter)
- Idéal pour embeddings normalisés

```python
VectorParams(size=384, distance=Distance.COSINE)
```

### Distance EUCLID

**Utilisation :**
- Coordonnées géographiques
- Features numériques brutes
- Classification d'images

**Avantages :**
- Distance géométrique intuitive
- Bon pour données numériques

```python
VectorParams(size=128, distance=Distance.EUCLID)
```

### Distance DOT

**Utilisation :**
- Vecteurs non normalisés
- Scores pondérés
- Recommandations avec poids

**Avantages :**
- Prend en compte la magnitude
- Bon pour scores pondérés

```python
VectorParams(size=128, distance=Distance.DOT)
```

### Tableau de décision

| Type de données | Distance recommandée |
|-----------------|---------------------|
| Textes (embeddings normalisés) | **COSINE** |
| Textes (embeddings non normalisés) | **DOT** |
| Coordonnées GPS | **EUCLID** |
| Features numériques | **EUCLID** |
| Images | **COSINE** ou **EUCLID** |
| Recommandations | **COSINE** |

---

## Configuration HNSW

### Paramètres HNSW

```python
from qdrant_client.models import HnswConfigDiff

hnsw_config = HnswConfigDiff(
    m=16,  # Nombre de connexions (défaut: 16)
    ef_construct=100,  # Précision de construction (défaut: 100)
    full_scan_threshold=10000  # Seuil pour scan complet
)
```

### Paramètre m (connexions)

- **m=8-16** : Rapide, moins précis (petites collections)
- **m=16-32** : Équilibré (recommandé)
- **m=32-64** : Lent, très précis (grandes collections)

```python
# Collection rapide (petite)
HnswConfigDiff(m=8, ef_construct=50)

# Collection équilibrée (moyenne)
HnswConfigDiff(m=16, ef_construct=100)

# Collection précise (grande)
HnswConfigDiff(m=32, ef_construct=200)
```

### Paramètre ef_construct

- **ef_construct=50-100** : Construction rapide
- **ef_construct=100-200** : Équilibré
- **ef_construct=200+** : Construction lente, meilleure qualité

### Paramètre ef (recherche)

```python
# Recherche rapide (moins précis)
results = client.search(
    collection_name="products",
    query_vector=vector,
    limit=10,
    ef=32  # Plus petit = plus rapide
)

# Recherche précise (plus lent)
results = client.search(
    collection_name="products",
    query_vector=vector,
    limit=10,
    ef=128  # Plus grand = plus précis
)
```

---

## Structure des métadonnées

### Bonnes pratiques pour le payload

```python
# ✅ Bon : Structure claire et typée
payload = {
    "title": "Product Name",  # String
    "category": "electronics",  # String (indexable)
    "price": 99.99,  # Float (indexable)
    "quantity": 10,  # Integer
    "in_stock": True,  # Boolean
    "tags": ["laptop", "gaming"],  # Array
    "created_at": "2024-01-15T10:00:00Z",  # ISO date string
    "metadata": {  # Objet imbriqué
        "brand": "BrandX",
        "model": "Model123"
    }
}

# ❌ Moins bon : Structure incohérente
payload = {
    "title": "Product Name",
    "Category": "electronics",  # Incohérence de casse
    "price": "99.99",  # String au lieu de number
    "tags": "laptop,gaming",  # String au lieu d'array
    "created": "15/01/2024"  # Format date non standard
}
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

client.create_payload_index(
    collection_name="products",
    field_name="created_at",
    field_schema=PayloadSchemaType.KEYWORD  # Pour dates
)
```

### Éviter les payloads trop volumineux

```python
# ✅ Bon : Payload concis
payload = {
    "id": 123,
    "title": "Product",
    "category": "electronics"
}

# ❌ Moins bon : Payload trop volumineux
payload = {
    "id": 123,
    "title": "Product",
    "full_description": "..." * 1000,  # Texte très long
    "high_res_image": base64_image,  # Image encodée
    "full_specs": {...}  # Objet très volumineux
}
```

---

## Gestion de la mémoire

### Quantisation

La **quantisation** réduit la mémoire utilisée :

```python
from qdrant_client.models import QuantizationConfig, ScalarQuantization, ScalarType

# Quantisation INT8 (réduction 4x)
quantization_config = QuantizationConfig(
    scalar=ScalarQuantization(
        type=ScalarType.INT8,
        quantile=0.99,
        always_ram=True
    )
)

client.create_collection(
    collection_name="products",
    vectors_config=VectorParams(size=384, distance=Distance.COSINE),
    quantization_config=quantization_config
)
```

**Avantages :**
- Réduction mémoire : 4x (float32 → int8)
- Recherche plus rapide
- Légère perte de précision

### Memmap (mémoire mappée)

Pour très grandes collections :

```python
from qdrant_client.models import OptimizersConfigDiff

optimizers_config = OptimizersConfigDiff(
    memmap_threshold=50000  # Utiliser memmap si > 50000 points
)

client.create_collection(
    collection_name="large_collection",
    vectors_config=VectorParams(size=384, distance=Distance.COSINE),
    optimizers_config=optimizers_config
)
```

---

## Performance

### Batch operations

```python
# ✅ Bon : Insertion par batches
points = [PointStruct(...) for _ in range(1000)]
client.upsert(collection_name="products", points=points)

# ❌ Moins bon : Insertion point par point
for point in points:
    client.upsert(collection_name="products", points=[point])
```

### Pré-filtrage vs Post-filtrage

```python
# Pré-filtrage (recommandé si peu de résultats après filtrage)
results = client.search(
    collection_name="products",
    query_vector=vector,
    query_filter=filter,  # Filtre d'abord
    limit=10
)

# Post-filtrage (si beaucoup de résultats après filtrage)
results = client.search(
    collection_name="products",
    query_vector=vector,
    limit=100  # Plus de résultats
)
# Filtrer manuellement ensuite
```

### Optimiser les embeddings

```python
# ✅ Bon : Batch encoding
embeddings = model.encode(texts, batch_size=32)

# ❌ Moins bon : Un par un
embeddings = [model.encode([text])[0] for text in texts]
```

### Utiliser le GPU

```python
import torch

device = "cuda" if torch.cuda.is_available() else "cpu"
model = SentenceTransformer('all-MiniLM-L6-v2', device=device)

# Batch plus grand sur GPU
embeddings = model.encode(texts, batch_size=64)
```

---

## Sécurité

### Authentification

```python
# Connexion avec API key
client = QdrantClient(
    url="https://your-cluster.qdrant.io",
    api_key="your-api-key"
)
```

### Permissions

- Utiliser des API keys différentes par environnement
- Limiter les permissions par collection
- Ne pas exposer les API keys dans le code

### Données sensibles

```python
# ❌ Ne pas stocker de données sensibles dans le payload
payload = {
    "name": "Product",
    "password": "secret123"  # ❌ Ne jamais faire ça
}

# ✅ Stocker seulement les données nécessaires
payload = {
    "name": "Product",
    "category": "electronics"
}
```

---

## Monitoring

### Vérifier la santé

```python
# Informations de la collection
collection_info = client.get_collection("products")

print(f"Points: {collection_info.points_count}")
print(f"Indexed: {collection_info.indexed_vectors_count}")
print(f"Status: {collection_info.status}")
```

### Statistiques

```python
# Statistiques de la collection
stats = client.get_collection("products")

print(f"Vectors count: {stats.vectors_count}")
print(f"Indexed vectors: {stats.indexed_vectors_count}")
print(f"Points count: {stats.points_count}")
```

### Monitoring des performances

```python
import time

# Mesurer le temps de recherche
start = time.time()
results = client.search(
    collection_name="products",
    query_vector=vector,
    limit=10
)
duration = time.time() - start

print(f"Search took {duration:.3f}s")
print(f"Results: {len(results)}")
```

---

## Checklist de bonnes pratiques

### Configuration

- [ ] Dimension appropriée pour le cas d'usage
- [ ] Distance correcte (COSINE pour textes)
- [ ] HNSW configuré selon la taille de la collection
- [ ] Quantisation activée si nécessaire

### Données

- [ ] Payload structuré et cohérent
- [ ] Champs fréquemment filtrés indexés
- [ ] Pas de données sensibles dans le payload
- [ ] Payload pas trop volumineux

### Performance

- [ ] Insertion par batches
- [ ] Pré-filtrage utilisé quand approprié
- [ ] Batch encoding pour les embeddings
- [ ] GPU utilisé si disponible

### Maintenance

- [ ] Monitoring régulier
- [ ] Vérification de la santé des collections
- [ ] Backup régulier
- [ ] Documentation à jour

---

## 🎯 Points clés à retenir

✅ Dimension 384-512 pour textes, 128-256 pour grandes collections  
✅ COSINE pour textes, EUCLID pour coordonnées  
✅ HNSW m=16-32 pour équilibre performance/précision  
✅ Indexer les champs fréquemment filtrés  
✅ Utiliser batch operations pour meilleures performances  
✅ Quantisation pour réduire la mémoire  
✅ Monitoring régulier pour détecter les problèmes  

---

**Prochaine étape :** [Projets Pratiques](./08-projets/README.md)
