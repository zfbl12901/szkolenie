# 8. Projets Pratiques

## 🎯 Objectifs

- Créer des projets complets
- Appliquer les connaissances acquises
- Intégrer avec l'IA
- Optimiser les performances
- Construire un portfolio

## 📋 Table des matières

1. [Projet 1 : Moteur de recherche sémantique](#projet-1--moteur-de-recherche-sémantique)
2. [Projet 2 : Système de recommandation](#projet-2--système-de-recommandation)
3. [Projet 3 : Chatbot avec RAG](#projet-3--chatbot-avec-rag)
4. [Projet 4 : Déduplication de documents](#projet-4--déduplication-de-documents)
5. [Projet 5 : Classification de textes](#projet-5--classification-de-textes)

---

## Projet 1 : Moteur de recherche sémantique

### Objectif

Créer un moteur de recherche sémantique pour des documents, permettant de trouver des résultats par sens plutôt que par mots-clés.

### Fonctionnalités

- Indexation de documents
- Recherche sémantique
- Filtres avancés
- Interface simple

### Implémentation complète

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct, Filter, FieldCondition, MatchValue
import pandas as pd

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
    
    def index_documents(self, documents, batch_size=100):
        """Indexer des documents"""
        for i in range(0, len(documents), batch_size):
            batch = documents[i:i+batch_size]
            
            texts = [doc["text"] for doc in batch]
            embeddings = self.model.encode(texts, show_progress_bar=True)
            
            points = [
                PointStruct(
                    id=doc["id"],
                    vector=emb.tolist(),
                    payload={
                        "text": doc["text"],
                        "title": doc.get("title", ""),
                        "category": doc.get("category", ""),
                        "author": doc.get("author", ""),
                        "date": doc.get("date", "")
                    }
                )
                for doc, emb in zip(batch, embeddings)
            ]
            
            self.client.upsert(
                collection_name=self.collection_name,
                points=points
            )
            print(f"Indexed {min(i+batch_size, len(documents))}/{len(documents)}")
    
    def search(self, query, limit=10, category=None, min_score=0.5):
        """Rechercher des documents"""
        query_embedding = self.model.encode([query])[0]
        
        # Construire le filtre
        filter_conditions = []
        if category:
            filter_conditions.append(
                FieldCondition(key="category", match=MatchValue(value=category))
            )
        
        query_filter = Filter(must=filter_conditions) if filter_conditions else None
        
        # Recherche
        results = self.client.search(
            collection_name=self.collection_name,
            query_vector=query_embedding.tolist(),
            query_filter=query_filter,
            limit=limit,
            score_threshold=min_score
        )
        
        return [
            {
                "id": r.id,
                "score": r.score,
                "title": r.payload.get("title", ""),
                "text": r.payload.get("text", ""),
                "category": r.payload.get("category", "")
            }
            for r in results
        ]
    
    def index_from_csv(self, csv_path, text_column, id_column=None):
        """Indexer depuis un CSV"""
        df = pd.read_csv(csv_path)
        
        if id_column is None:
            df['_id'] = range(len(df))
            id_column = '_id'
        
        documents = [
            {
                "id": int(row[id_column]),
                "text": str(row[text_column]),
                "title": str(row.get("title", "")),
                "category": str(row.get("category", "")),
                "author": str(row.get("author", ""))
            }
            for _, row in df.iterrows()
        ]
        
        self.index_documents(documents)

# Utilisation
engine = SemanticSearchEngine()

# Indexer depuis CSV
engine.index_from_csv("documents.csv", text_column="content", id_column="doc_id")

# Rechercher
results = engine.search("computer for work", limit=5, category="technology")

for result in results:
    print(f"Score: {result['score']:.4f}")
    print(f"Title: {result['title']}")
    print(f"Text: {result['text'][:100]}...")
    print("---")
```

### Améliorations possibles

- Interface web (Flask/FastAPI)
- Pagination des résultats
- Highlighting des termes pertinents
- Historique de recherche
- Analytics des recherches

---

## Projet 2 : Système de recommandation

### Objectif

Créer un système de recommandation de produits combinant similarité vectorielle et préférences utilisateur.

### Fonctionnalités

- Recommandation par similarité
- Filtres par préférences
- Historique utilisateur
- Scoring hybride

### Implémentation

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import (
    Distance, VectorParams, PointStruct, Filter, 
    FieldCondition, MatchValue, Range, MatchAny
)
import numpy as np

class RecommendationSystem:
    def __init__(self):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.client = QdrantClient(host="localhost", port=6333)
        self._ensure_collection()
    
    def _ensure_collection(self):
        collections = self.client.get_collections()
        if "products" not in [c.name for c in collections.collections]:
            self.client.create_collection(
                collection_name="products",
                vectors_config=VectorParams(size=384, distance=Distance.COSINE)
            )
    
    def index_products(self, products):
        """Indexer des produits"""
        texts = [p["description"] for p in products]
        embeddings = self.model.encode(texts)
        
        points = [
            PointStruct(
                id=p["id"],
                vector=emb.tolist(),
                payload={
                    "name": p["name"],
                    "description": p["description"],
                    "category": p["category"],
                    "price": p["price"],
                    "brand": p.get("brand", "")
                }
            )
            for p, emb in zip(products, embeddings)
        ]
        
        self.client.upsert(collection_name="products", points=points)
    
    def recommend_similar(self, product_id, limit=10):
        """Recommandation par similarité"""
        # Récupérer le produit
        points = self.client.retrieve(
            collection_name="products",
            ids=[product_id],
            with_vectors=True
        )
        
        if not points:
            return []
        
        product_vector = points[0].vector
        
        # Rechercher des produits similaires
        results = self.client.search(
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
    
    def recommend_for_user(self, user_id, user_preferences, limit=10):
        """Recommandation personnalisée"""
        # Récupérer l'historique utilisateur
        user_history = self._get_user_history(user_id)
        
        if user_history:
            # Profil utilisateur (vecteur moyen)
            product_ids = [item["product_id"] for item in user_history]
            points = self.client.retrieve(
                collection_name="products",
                ids=product_ids,
                with_vectors=True
            )
            
            vectors = np.array([p.vector for p in points])
            user_vector = vectors.mean(axis=0).tolist()
        else:
            # Pas d'historique : utiliser catégorie préférée
            category_products = self._get_category_products(user_preferences["category"])
            if category_products:
                vectors = np.array([p.vector for p in category_products])
                user_vector = vectors.mean(axis=0).tolist()
            else:
                return []
        
        # Recherche avec filtres
        filter_conditions = []
        
        if user_preferences.get("min_price"):
            filter_conditions.append(
                FieldCondition(
                    key="price",
                    range=Range(gte=user_preferences["min_price"])
                )
            )
        
        if user_preferences.get("max_price"):
            if filter_conditions:
                # Mettre à jour le range existant
                for cond in filter_conditions:
                    if hasattr(cond, 'key') and cond.key == "price":
                        cond.range.lte = user_preferences["max_price"]
                        break
            else:
                filter_conditions.append(
                    FieldCondition(
                        key="price",
                        range=Range(lte=user_preferences["max_price"])
                    )
                )
        
        if user_preferences.get("category"):
            filter_conditions.append(
                FieldCondition(
                    key="category",
                    match=MatchValue(value=user_preferences["category"])
                )
            )
        
        # Exclure les produits déjà achetés
        if user_history:
            purchased_ids = [item["product_id"] for item in user_history]
            filter_conditions.append(
                FieldCondition(
                    key="id",
                    match=MatchAny(any=purchased_ids)
                )
            )
        
        query_filter = Filter(must=filter_conditions) if filter_conditions else None
        
        results = self.client.search(
            collection_name="products",
            query_vector=user_vector,
            query_filter=query_filter,
            limit=limit,
            score_threshold=0.7
        )
        
        return results
    
    def _get_user_history(self, user_id):
        # Implémentation : récupérer depuis une base de données
        # Exemple simplifié
        return []
    
    def _get_category_products(self, category):
        # Récupérer quelques produits de la catégorie pour créer le profil
        results = self.client.scroll(
            collection_name="products",
            scroll_filter=Filter(
                must=[
                    FieldCondition(key="category", match=MatchValue(value=category))
                ]
            ),
            limit=10,
            with_vectors=True
        )
        return results[0] if results[0] else []

# Utilisation
recommender = RecommendationSystem()

# Recommandation par similarité
similar = recommender.recommend_similar(product_id=123, limit=5)

# Recommandation personnalisée
user_prefs = {
    "category": "electronics",
    "min_price": 100,
    "max_price": 500
}
recommendations = recommender.recommend_for_user(
    user_id=456,
    user_preferences=user_prefs,
    limit=10
)
```

---

## Projet 3 : Chatbot avec RAG

### Objectif

Créer un chatbot qui utilise RAG (Retrieval-Augmented Generation) pour répondre aux questions en utilisant une base de connaissances.

### Implémentation

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
from openai import OpenAI  # Ou autre LLM

class RAGChatbot:
    def __init__(self):
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        self.client = QdrantClient(host="localhost", port=6333)
        self.llm = OpenAI()  # Configuration nécessaire
        self._ensure_collection()
    
    def _ensure_collection(self):
        collections = self.client.get_collections()
        if "knowledge_base" not in [c.name for c in collections.collections]:
            self.client.create_collection(
                collection_name="knowledge_base",
                vectors_config=VectorParams(size=384, distance=Distance.COSINE)
            )
    
    def index_knowledge(self, documents):
        """Indexer la base de connaissances"""
        texts = [doc["text"] for doc in documents]
        embeddings = self.embedding_model.encode(texts)
        
        points = [
            PointStruct(
                id=doc["id"],
                vector=emb.tolist(),
                payload={
                    "text": doc["text"],
                    "title": doc.get("title", ""),
                    "source": doc.get("source", ""),
                    "section": doc.get("section", "")
                }
            )
            for doc, emb in zip(documents, embeddings)
        ]
        
        self.client.upsert(collection_name="knowledge_base", points=points)
    
    def retrieve_context(self, query, limit=5):
        """Récupérer le contexte pertinent"""
        query_embedding = self.embedding_model.encode([query])[0]
        
        results = self.client.search(
            collection_name="knowledge_base",
            query_vector=query_embedding.tolist(),
            limit=limit,
            score_threshold=0.7
        )
        
        context_parts = []
        sources = []
        
        for i, result in enumerate(results):
            context_parts.append(f"[Source {i+1}]: {result.payload['text']}")
            sources.append({
                "id": result.id,
                "title": result.payload.get("title", ""),
                "source": result.payload.get("source", ""),
                "score": result.score
            })
        
        context = "\n\n".join(context_parts)
        return context, sources
    
    def generate_answer(self, query, context):
        """Générer une réponse avec le contexte"""
        prompt = f"""Tu es un assistant utile. Réponds à la question en utilisant uniquement le contexte fourni.

Contexte:
{context}

Question: {query}

Réponds de manière claire et concise. Si la réponse n'est pas dans le contexte, dis-le."""
        
        response = self.llm.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "Tu es un assistant utile et précis."},
                {"role": "user", "content": prompt}
            ],
            temperature=0.7,
            max_tokens=500
        )
        
        return response.choices[0].message.content
    
    def answer(self, query, limit=5):
        """Pipeline RAG complet"""
        # 1. Retrieval
        context, sources = self.retrieve_context(query, limit)
        
        if not context:
            return {
                "answer": "Désolé, je n'ai pas trouvé d'information pertinente dans la base de connaissances.",
                "sources": []
            }
        
        # 2. Generation
        answer = self.generate_answer(query, context)
        
        return {
            "answer": answer,
            "sources": sources
        }

# Utilisation
chatbot = RAGChatbot()

# Indexer la base de connaissances
knowledge = [
    {"id": 1, "text": "Les laptops gaming sont optimisés pour les jeux vidéo...", "title": "Laptops Gaming"},
    {"id": 2, "text": "Les workstations professionnelles sont conçues pour...", "title": "Workstations"}
]
chatbot.index_knowledge(knowledge)

# Poser une question
result = chatbot.answer("Quels sont les avantages des laptops gaming?")
print(f"Réponse: {result['answer']}")
print(f"Sources: {result['sources']}")
```

---

## Projet 4 : Déduplication de documents

### Objectif

Détecter et supprimer les documents dupliqués dans une collection.

### Implémentation

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Filter, FieldCondition, MatchValue

class DeduplicationSystem:
    def __init__(self, collection_name="documents"):
        self.client = QdrantClient(host="localhost", port=6333)
        self.collection_name = collection_name
    
    def find_duplicates(self, threshold=0.95):
        """Trouver les documents dupliqués"""
        # Récupérer tous les points
        all_points = []
        scroll_result = self.client.scroll(
            collection_name=self.collection_name,
            limit=1000,
            with_vectors=True,
            with_payload=True
        )
        points, next_offset = scroll_result
        all_points.extend(points)
        
        while next_offset is not None:
            scroll_result = self.client.scroll(
                collection_name=self.collection_name,
                limit=1000,
                offset=next_offset,
                with_vectors=True,
                with_payload=True
            )
            points, next_offset = scroll_result
            all_points.extend(points)
        
        # Trouver les doublons
        duplicates = []
        processed = set()
        
        for point in all_points:
            if point.id in processed:
                continue
            
            # Chercher des documents similaires
            similar = self.client.search(
                collection_name=self.collection_name,
                query_vector=point.vector,
                limit=10,
                score_threshold=threshold
            )
            
            # Filtrer l'élément lui-même
            similar = [s for s in similar if s.id != point.id]
            
            if similar:
                duplicates.append({
                    "original": point.id,
                    "duplicates": [s.id for s in similar],
                    "scores": [s.score for s in similar]
                })
                processed.add(point.id)
                processed.update([s.id for s in similar])
        
        return duplicates
    
    def remove_duplicates(self, threshold=0.95, keep="first"):
        """Supprimer les doublons"""
        duplicates = self.find_duplicates(threshold)
        
        ids_to_delete = []
        
        for group in duplicates:
            if keep == "first":
                # Garder le premier (original), supprimer les autres
                ids_to_delete.extend(group["duplicates"])
            elif keep == "best":
                # Garder celui avec le meilleur score, supprimer les autres
                # Implémentation simplifiée
                ids_to_delete.extend(group["duplicates"])
        
        # Supprimer les points
        if ids_to_delete:
            self.client.delete(
                collection_name=self.collection_name,
                points_selector=ids_to_delete
            )
        
        return len(ids_to_delete)

# Utilisation
dedup = DeduplicationSystem()
duplicates = dedup.find_duplicates(threshold=0.95)
print(f"Found {len(duplicates)} duplicate groups")

# Supprimer les doublons
removed = dedup.remove_duplicates(threshold=0.95, keep="first")
print(f"Removed {removed} duplicate documents")
```

---

## Projet 5 : Classification de textes

### Objectif

Classer automatiquement des textes dans des catégories prédéfinies.

### Implémentation

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import Filter, FieldCondition, MatchValue

class TextClassifier:
    def __init__(self):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.client = QdrantClient(host="localhost", port=6333)
    
    def train_classifier(self, training_data):
        """Entraîner le classifieur avec des exemples"""
        # training_data = [{"text": "...", "category": "electronics"}, ...]
        
        # Indexer les exemples d'entraînement
        texts = [item["text"] for item in training_data]
        embeddings = self.model.encode(texts)
        
        points = [
            PointStruct(
                id=i,
                vector=emb.tolist(),
                payload={
                    "text": item["text"],
                    "category": item["category"]
                }
            )
            for i, (item, emb) in enumerate(zip(training_data, embeddings))
        ]
        
        self.client.upsert(collection_name="training_data", points=points)
    
    def classify(self, text, categories):
        """Classer un texte"""
        query_embedding = self.model.encode([text])[0]
        
        category_scores = {}
        
        for category in categories:
            results = self.client.search(
                collection_name="training_data",
                query_vector=query_embedding.tolist(),
                query_filter=Filter(
                    must=[
                        FieldCondition(key="category", match=MatchValue(value=category))
                    ]
                ),
                limit=1
            )
            
            if results:
                category_scores[category] = results[0].score
        
        if category_scores:
            best_category = max(category_scores, key=category_scores.get)
            confidence = category_scores[best_category]
            return best_category, confidence, category_scores
        
        return None, 0.0, {}

# Utilisation
classifier = TextClassifier()

# Entraîner
training = [
    {"text": "Laptop computer for work", "category": "electronics"},
    {"text": "Gaming laptop with graphics", "category": "electronics"},
    {"text": "Novel about adventure", "category": "books"},
    {"text": "Science fiction book", "category": "books"}
]
classifier.train_classifier(training)

# Classer
category, confidence, all_scores = classifier.classify(
    "Professional workstation computer",
    categories=["electronics", "books", "clothing"]
)

print(f"Category: {category}")
print(f"Confidence: {confidence:.4f}")
print(f"All scores: {all_scores}")
```

---

## Conseils pour votre portfolio

### Documentation

1. **README.md** : Expliquer le projet, l'installation, l'utilisation
2. **Architecture** : Diagramme de l'architecture
3. **Exemples** : Exemples d'utilisation
4. **Résultats** : Métriques et performances

### GitHub

1. **Code propre** : Commentaires, docstrings
2. **Structure claire** : Organisation des fichiers
3. **Requirements.txt** : Dépendances listées
4. **Exemples** : Scripts d'exemple fonctionnels

### Démo

1. **Interface** : Web app simple (Flask/FastAPI)
2. **Screenshots** : Captures d'écran
3. **Vidéo** : Démo vidéo courte
4. **Live demo** : Démo en ligne si possible

---

## 🎯 Points clés à retenir

✅ Les projets pratiques démontrent vos compétences  
✅ Documenter vos projets pour votre portfolio  
✅ Optimiser les performances pour les cas réels  
✅ Gérer les erreurs et cas limites  
✅ Tester avec des données réelles  

---

**Félicitations ! Vous avez terminé la formation Qdrant pour Data Analyst ! 🎉**

Vous maîtrisez maintenant :
- Les bases de données vectorielles
- Qdrant et ses fonctionnalités
- L'intégration avec Python et l'IA
- Les cas d'usage pratiques
- Les bonnes pratiques

Continuez à pratiquer et à construire des projets pour renforcer vos compétences !
