# 6. Cas d'usage IA/ML

## 🎯 Objectifs

- Comprendre les cas d'usage typiques
- Implémenter la recherche sémantique
- Créer des systèmes de recommandation
- Utiliser avec RAG (Retrieval-Augmented Generation)
- Détecter les doublons
- Clustering et classification

## 📋 Table des matières

1. [Recherche sémantique](#recherche-sémantique)
2. [Systèmes de recommandation](#systèmes-de-recommandation)
3. [RAG (Retrieval-Augmented Generation)](#rag-retrieval-augmented-generation)
4. [Déduplication](#déduplication)
5. [Clustering](#clustering)
6. [Classification](#classification)
7. [Anomaly Detection](#anomaly-detection)

---

## Recherche sémantique

### Qu'est-ce que la recherche sémantique ?

La **recherche sémantique** trouve des résultats basés sur le **sens** plutôt que sur des mots-clés exacts.

**Recherche classique (mots-clés) :**
- Requête : "laptop"
- Résultats : Documents contenant le mot "laptop"

**Recherche sémantique :**
- Requête : "computer for work"
- Résultats : Documents sur "laptop", "workstation", "desktop", etc.

### Implémentation complète

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

class SemanticSearchEngine:
    def __init__(self, collection_name="documents"):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.client = QdrantClient(host="localhost", port=6333)
        self.collection_name = collection_name
        self._ensure_collection()
    
    def _ensure_collection(self):
        """Créer la collection si nécessaire"""
        collections = self.client.get_collections()
        if self.collection_name not in [c.name for c in collections.collections]:
            self.client.create_collection(
                collection_name=self.collection_name,
                vectors_config=VectorParams(size=384, distance=Distance.COSINE)
            )
    
    def index_documents(self, documents):
        """Indexer des documents"""
        texts = [doc["text"] for doc in documents]
        embeddings = self.model.encode(texts)
        
        points = [
            PointStruct(
                id=doc["id"],
                vector=emb.tolist(),
                payload={"text": doc["text"], **doc.get("metadata", {})}
            )
            for doc, emb in zip(documents, embeddings)
        ]
        
        self.client.upsert(collection_name=self.collection_name, points=points)
    
    def search(self, query, limit=10, filter=None):
        """Rechercher des documents similaires"""
        query_embedding = self.model.encode([query])[0]
        
        results = self.client.search(
            collection_name=self.collection_name,
            query_vector=query_embedding.tolist(),
            query_filter=filter,
            limit=limit
        )
        
        return [
            {
                "id": r.id,
                "score": r.score,
                "text": r.payload["text"],
                "metadata": {k: v for k, v in r.payload.items() if k != "text"}
            }
            for r in results
        ]

# Utilisation
engine = SemanticSearchEngine()

# Indexer
documents = [
    {"id": 1, "text": "Laptop computer for professional work"},
    {"id": 2, "text": "Gaming laptop with high-end graphics"},
    {"id": 3, "text": "Business workstation for office use"}
]
engine.index_documents(documents)

# Rechercher
results = engine.search("computer for work", limit=5)
for result in results:
    print(f"Score: {result['score']:.4f} - {result['text']}")
```

---

## Systèmes de recommandation

### Recommandation basique (similarité)

```python
def recommend_similar_products(product_id, limit=10):
    """Recommander des produits similaires"""
    
    client = QdrantClient(host="localhost", port=6333)
    
    # 1. Récupérer le vecteur du produit
    points = client.retrieve(
        collection_name="products",
        ids=[product_id],
        with_vectors=True
    )
    
    if not points:
        return []
    
    product_vector = points[0].vector
    
    # 2. Rechercher des produits similaires (exclure l'original)
    results = client.search(
        collection_name="products",
        query_vector=product_vector,
        query_filter=Filter(
            must_not=[
                FieldCondition(key="id", match=MatchValue(value=product_id))
            ]
        ),
        limit=limit
    )
    
    return results
```

### Recommandation hybride (similarité + préférences)

```python
def hybrid_recommendation(product_id, user_preferences, limit=10):
    """Recommandation combinant similarité et préférences"""
    
    client = QdrantClient(host="localhost", port=6333)
    
    # Vecteur du produit
    points = client.retrieve(
        collection_name="products",
        ids=[product_id],
        with_vectors=True
    )
    product_vector = points[0].vector
    
    # Recherche avec filtres de préférences
    results = client.search(
        collection_name="products",
        query_vector=product_vector,
        query_filter=Filter(
            must=[
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
                FieldCondition(key="id", match=MatchValue(value=product_id))
            ]
        ),
        limit=limit,
        score_threshold=0.7
    )
    
    return results
```

### Recommandation basée sur l'historique utilisateur

```python
def user_based_recommendation(user_id, limit=10):
    """Recommandation basée sur l'historique utilisateur"""
    
    client = QdrantClient(host="localhost", port=6333)
    model = SentenceTransformer('all-MiniLM-L6-v2')
    
    # 1. Récupérer l'historique utilisateur
    user_history = get_user_purchase_history(user_id)  # Fonction externe
    
    # 2. Générer embeddings des produits achetés
    product_texts = [item["description"] for item in user_history]
    embeddings = model.encode(product_texts)
    
    # 3. Calculer le vecteur moyen (profil utilisateur)
    user_profile_vector = embeddings.mean(axis=0).tolist()
    
    # 4. Recommander des produits similaires
    results = client.search(
        collection_name="products",
        query_vector=user_profile_vector,
        query_filter=Filter(
            must_not=[
                FieldCondition(
                    key="id",
                    match=MatchAny(any=[item["product_id"] for item in user_history])
                )
            ]
        ),
        limit=limit
    )
    
    return results
```

---

## RAG (Retrieval-Augmented Generation)

### Qu'est-ce que RAG ?

**RAG** combine :
- **Retrieval** : Recherche de documents pertinents (Qdrant)
- **Augmented Generation** : Génération de texte avec LLM (GPT, etc.)

### Pipeline RAG simple

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from openai import OpenAI

class RAGSystem:
    def __init__(self):
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        self.client = QdrantClient(host="localhost", port=6333)
        self.llm = OpenAI()  # Ou autre LLM
    
    def retrieve_context(self, query, limit=5):
        """Récupérer le contexte pertinent"""
        query_embedding = self.embedding_model.encode([query])[0]
        
        results = self.client.search(
            collection_name="knowledge_base",
            query_vector=query_embedding.tolist(),
            limit=limit
        )
        
        # Extraire le texte
        context = "\n\n".join([
            result.payload["text"] for result in results
        ])
        
        return context
    
    def generate_answer(self, query, context):
        """Générer une réponse avec le contexte"""
        
        prompt = f"""Contexte:
{context}

Question: {query}

Réponds à la question en utilisant le contexte fourni."""

        response = self.llm.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "Tu es un assistant utile."},
                {"role": "user", "content": prompt}
            ]
        )
        
        return response.choices[0].message.content
    
    def answer(self, query, limit=5):
        """Pipeline RAG complet"""
        # 1. Retrieval
        context = self.retrieve_context(query, limit)
        
        # 2. Generation
        answer = self.generate_answer(query, context)
        
        return {
            "answer": answer,
            "context": context
        }

# Utilisation
rag = RAGSystem()
result = rag.answer("Quels sont les avantages des laptops gaming?")
print(result["answer"])
```

### RAG avec sources

```python
def rag_with_sources(query, limit=5):
    """RAG qui retourne aussi les sources"""
    
    query_embedding = model.encode([query])[0]
    
    results = client.search(
        collection_name="knowledge_base",
        query_vector=query_embedding.tolist(),
        limit=limit
    )
    
    # Construire le contexte avec sources
    context_parts = []
    sources = []
    
    for i, result in enumerate(results):
        text = result.payload["text"]
        source = result.payload.get("source", f"Document {result.id}")
        
        context_parts.append(f"[Source {i+1}]: {text}")
        sources.append({
            "id": result.id,
            "source": source,
            "score": result.score
        })
    
    context = "\n\n".join(context_parts)
    
    # Générer réponse
    answer = generate_answer(query, context)
    
    return {
        "answer": answer,
        "sources": sources
    }
```

---

## Déduplication

### Détecter les doublons

```python
def find_duplicates(threshold=0.95):
    """Trouver les documents dupliqués"""
    
    client = QdrantClient(host="localhost", port=6333)
    
    # Parcourir tous les points
    all_points = []
    scroll_result = client.scroll(
        collection_name="documents",
        limit=100,
        with_vectors=True,
        with_payload=True
    )
    
    points, next_offset = scroll_result
    all_points.extend(points)
    
    # Continuer le scroll
    while next_offset is not None:
        scroll_result = client.scroll(
            collection_name="documents",
            limit=100,
            offset=next_offset,
            with_vectors=True,
            with_payload=True
        )
        points, next_offset = scroll_result
        all_points.extend(points)
    
    # Trouver les doublons
    duplicates = []
    processed = set()
    
    for i, point1 in enumerate(all_points):
        if point1.id in processed:
            continue
        
        similar = client.search(
            collection_name="documents",
            query_vector=point1.vector,
            limit=10,
            score_threshold=threshold
        )
        
        # Filtrer l'élément lui-même
        similar = [s for s in similar if s.id != point1.id]
        
        if similar:
            duplicates.append({
                "original": point1.id,
                "duplicates": [s.id for s in similar]
            })
            processed.add(point1.id)
            processed.update([s.id for s in similar])
    
    return duplicates
```

---

## Clustering

### Clustering simple

```python
from sklearn.cluster import KMeans
import numpy as np

def cluster_documents(n_clusters=5):
    """Grouper des documents similaires"""
    
    client = QdrantClient(host="localhost", port=6333)
    
    # Récupérer tous les vecteurs
    all_points = []
    scroll_result = client.scroll(
        collection_name="documents",
        limit=1000,
        with_vectors=True,
        with_payload=True
    )
    points, _ = scroll_result
    all_points.extend(points)
    
    # Extraire les vecteurs
    vectors = np.array([point.vector for point in all_points])
    
    # Clustering K-means
    kmeans = KMeans(n_clusters=n_clusters, random_state=42)
    clusters = kmeans.fit_predict(vectors)
    
    # Mettre à jour le payload avec les clusters
    for point, cluster_id in zip(all_points, clusters):
        client.set_payload(
            collection_name="documents",
            payload={"cluster": int(cluster_id)},
            points=[point.id]
        )
    
    return clusters
```

---

## Classification

### Classification par similarité

```python
def classify_document(query_vector, categories):
    """Classer un document dans une catégorie"""
    
    client = QdrantClient(host="localhost", port=6333)
    
    # Pour chaque catégorie, trouver le document le plus similaire
    category_scores = {}
    
    for category in categories:
        results = client.search(
            collection_name="documents",
            query_vector=query_vector,
            query_filter=Filter(
                must=[
                    FieldCondition(key="category", match=MatchValue(value=category))
                ]
            ),
            limit=1
        )
        
        if results:
            category_scores[category] = results[0].score
    
    # Retourner la catégorie avec le meilleur score
    if category_scores:
        best_category = max(category_scores, key=category_scores.get)
        confidence = category_scores[best_category]
        return best_category, confidence
    
    return None, 0.0
```

---

## Anomaly Detection

### Détecter les anomalies

```python
def detect_anomalies(threshold=0.3):
    """Détecter des documents anormaux (peu similaires aux autres)"""
    
    client = QdrantClient(host="localhost", port=6333)
    
    # Parcourir tous les points
    all_points = []
    scroll_result = client.scroll(
        collection_name="documents",
        limit=1000,
        with_vectors=True
    )
    points, _ = scroll_result
    all_points.extend(points)
    
    anomalies = []
    
    for point in all_points:
        # Chercher les documents similaires
        results = client.search(
            collection_name="documents",
            query_vector=point.vector,
            limit=10,
            score_threshold=threshold
        )
        
        # Si peu de documents similaires, c'est une anomalie
        if len(results) < 3:  # Moins de 3 documents similaires
            anomalies.append(point.id)
    
    return anomalies
```

---

## Exercices pratiques

### Exercice 1 : Système de recommandation complet

Créer un système qui recommande des produits basé sur :
- Similarité vectorielle
- Catégorie préférée de l'utilisateur
- Budget de l'utilisateur

**Solution :**

```python
def recommend_products(user_id, limit=10):
    # Récupérer préférences utilisateur
    user_prefs = get_user_preferences(user_id)
    
    # Historique utilisateur
    history = get_user_history(user_id)
    
    # Profil utilisateur (vecteur moyen)
    if history:
        vectors = [get_product_vector(pid) for pid in history]
        user_vector = np.mean(vectors, axis=0).tolist()
    else:
        # Pas d'historique : utiliser catégorie préférée
        category_products = get_category_products(user_prefs["category"])
        vectors = [p.vector for p in category_products]
        user_vector = np.mean(vectors, axis=0).tolist()
    
    # Recherche avec filtres
    results = client.search(
        collection_name="products",
        query_vector=user_vector,
        query_filter=Filter(
            must=[
                FieldCondition(
                    key="price",
                    range=Range(
                        gte=user_prefs["min_price"],
                        lte=user_prefs["max_price"]
                    )
                )
            ],
            must_not=[
                FieldCondition(
                    key="id",
                    match=MatchAny(any=history)
                )
            ]
        ),
        limit=limit
    )
    
    return results
```

---

## 🎯 Points clés à retenir

✅ La recherche sémantique trouve par sens, pas par mots-clés  
✅ Les recommandations combinent similarité et préférences  
✅ RAG combine retrieval (Qdrant) et generation (LLM)  
✅ La déduplication utilise des seuils de similarité élevés  
✅ Le clustering groupe des documents similaires  

---

**Prochaine étape :** [Bonnes Pratiques](./07-best-practices/README.md)
