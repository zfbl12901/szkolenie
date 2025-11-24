# 5. Intégration Python

## 🎯 Objectifs

- Intégrer avec des modèles d'embeddings
- Utiliser sentence-transformers
- Intégrer avec OpenAI, Cohere, etc.
- Créer des pipelines de données
- Optimiser les performances
- Gérer les batchs d'embeddings

## 📋 Table des matières

1. [Modèles d'embeddings](#modèles-dembeddings)
2. [sentence-transformers](#sentence-transformers)
3. [OpenAI Embeddings](#openai-embeddings)
4. [Cohere Embeddings](#cohere-embeddings)
5. [Pipelines de données](#pipelines-de-données)
6. [Optimisation batch](#optimisation-batch)
7. [Gestion des erreurs](#gestion-des-erreurs)

---

## Modèles d'embeddings

### Qu'est-ce qu'un embedding ?

Un **embedding** est une représentation vectorielle d'un objet :
- **Texte** → Vecteur de nombres
- **Image** → Vecteur de nombres
- **Audio** → Vecteur de nombres

### Types de modèles

- **Text embeddings** : Pour textes (sentence-transformers, OpenAI)
- **Image embeddings** : Pour images (CLIP, ResNet)
- **Multimodal** : Pour texte + image (CLIP)

---

## sentence-transformers

### Installation

```bash
pip install sentence-transformers
```

### Modèle de base

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct

# Charger un modèle pré-entraîné
model = SentenceTransformer('all-MiniLM-L6-v2')  # Dimension 384

# Générer des embeddings
texts = [
    "Laptop computer for work",
    "Gaming laptop with high performance",
    "Professional workstation"
]

embeddings = model.encode(texts)
print(f"Shape: {embeddings.shape}")  # (3, 384)
```

### Modèles recommandés

```python
# Modèles populaires
models = {
    # Léger et rapide
    "all-MiniLM-L6-v2": 384,  # 22MB, rapide
    
    # Équilibré
    "all-mpnet-base-v2": 768,  # 420MB, meilleure qualité
    
    # Multilingue
    "paraphrase-multilingual-MiniLM-L12-v2": 384,  # Support multilingue
    
    # Spécialisé
    "ms-marco-MiniLM-L-6-v2": 384,  # Optimisé pour recherche
}
```

### Indexer des documents

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

# Initialisation
model = SentenceTransformer('all-MiniLM-L6-v2')
client = QdrantClient(host="localhost", port=6333)

# Créer la collection
client.create_collection(
    collection_name="documents",
    vectors_config=VectorParams(
        size=384,  # Dimension du modèle
        distance=Distance.COSINE
    )
)

# Documents à indexer
documents = [
    {"id": 1, "text": "Laptop computer for professional work"},
    {"id": 2, "text": "Gaming laptop with high-end graphics"},
    {"id": 3, "text": "Business workstation for office use"}
]

# Générer les embeddings
texts = [doc["text"] for doc in documents]
embeddings = model.encode(texts)

# Créer les points
points = [
    PointStruct(
        id=doc["id"],
        vector=embedding.tolist(),
        payload={"text": doc["text"]}
    )
    for doc, embedding in zip(documents, embeddings)
]

# Insérer dans Qdrant
client.upsert(collection_name="documents", points=points)
```

### Recherche sémantique

```python
# Requête utilisateur
query = "computer for work"

# Générer l'embedding de la requête
query_embedding = model.encode([query])[0]

# Rechercher dans Qdrant
results = client.search(
    collection_name="documents",
    query_vector=query_embedding.tolist(),
    limit=5
)

# Afficher les résultats
for result in results:
    print(f"Score: {result.score:.4f}")
    print(f"Text: {result.payload['text']}")
    print("---")
```

### Batch processing

```python
def index_documents_batch(documents, batch_size=100):
    """Indexer des documents par batches"""
    
    model = SentenceTransformer('all-MiniLM-L6-v2')
    client = QdrantClient(host="localhost", port=6333)
    
    for i in range(0, len(documents), batch_size):
        batch = documents[i:i+batch_size]
        
        # Générer embeddings
        texts = [doc["text"] for doc in batch]
        embeddings = model.encode(texts, show_progress_bar=True)
        
        # Créer points
        points = [
            PointStruct(
                id=doc["id"],
                vector=emb.tolist(),
                payload={"text": doc["text"]}
            )
            for doc, emb in zip(batch, embeddings)
        ]
        
        # Insérer
        client.upsert(collection_name="documents", points=points)
        print(f"Indexed batch {i//batch_size + 1}/{(len(documents)-1)//batch_size + 1}")
```

---

## OpenAI Embeddings

### Installation

```bash
pip install openai
```

### Configuration

```python
import os
from openai import OpenAI

# Configuration
os.environ["OPENAI_API_KEY"] = "your-api-key"
client_openai = OpenAI()
```

### Générer des embeddings

```python
from openai import OpenAI
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct

client_openai = OpenAI()
client_qdrant = QdrantClient(host="localhost", port=6333)

# Documents
texts = [
    "Laptop computer for work",
    "Gaming laptop with high performance"
]

# Générer embeddings avec OpenAI
response = client_openai.embeddings.create(
    model="text-embedding-ada-002",  # Dimension 1536
    input=texts
)

# Extraire les embeddings
embeddings = [item.embedding for item in response.data]

# Créer les points
points = [
    PointStruct(
        id=i,
        vector=embedding,
        payload={"text": text}
    )
    for i, (text, embedding) in enumerate(zip(texts, embeddings))
]

# Insérer dans Qdrant
client_qdrant.upsert(
    collection_name="documents_openai",
    points=points
)
```

### Recherche avec OpenAI

```python
# Requête
query = "computer for professional use"

# Embedding de la requête
response = client_openai.embeddings.create(
    model="text-embedding-ada-002",
    input=[query]
)
query_embedding = response.data[0].embedding

# Recherche
results = client_qdrant.search(
    collection_name="documents_openai",
    query_vector=query_embedding,
    limit=5
)
```

---

## Cohere Embeddings

### Installation

```bash
pip install cohere
```

### Utilisation

```python
import cohere
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct

# Initialisation
co = cohere.Client("your-api-key")
client = QdrantClient(host="localhost", port=6333)

# Générer embeddings
texts = ["Laptop computer", "Gaming laptop"]
response = co.embed(
    texts=texts,
    model="embed-english-v2.0"  # Dimension 4096
)

embeddings = response.embeddings

# Insérer dans Qdrant
points = [
    PointStruct(
        id=i,
        vector=embedding,
        payload={"text": text}
    )
    for i, (text, embedding) in enumerate(zip(texts, embeddings))
]

client.upsert(collection_name="documents_cohere", points=points)
```

---

## Pipelines de données

### Pipeline complet

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
import pandas as pd

class DocumentIndexer:
    def __init__(self, collection_name="documents"):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.client = QdrantClient(host="localhost", port=6333)
        self.collection_name = collection_name
        self._ensure_collection()
    
    def _ensure_collection(self):
        """Créer la collection si elle n'existe pas"""
        collections = self.client.get_collections()
        if self.collection_name not in [c.name for c in collections.collections]:
            self.client.create_collection(
                collection_name=self.collection_name,
                vectors_config=VectorParams(
                    size=384,
                    distance=Distance.COSINE
                )
            )
    
    def index_dataframe(self, df, text_column, id_column=None, batch_size=100):
        """Indexer un DataFrame"""
        
        if id_column is None:
            df['_id'] = range(len(df))
            id_column = '_id'
        
        for i in range(0, len(df), batch_size):
            batch = df.iloc[i:i+batch_size]
            
            # Générer embeddings
            texts = batch[text_column].tolist()
            embeddings = self.model.encode(texts)
            
            # Créer points
            points = [
                PointStruct(
                    id=int(row[id_column]),
                    vector=emb.tolist(),
                    payload=row.to_dict()
                )
                for row, emb in zip(batch.itertuples(), embeddings)
            ]
            
            # Insérer
            self.client.upsert(
                collection_name=self.collection_name,
                points=points
            )
            
            print(f"Indexed {min(i+batch_size, len(df))}/{len(df)}")
    
    def search(self, query, limit=10, filter=None):
        """Rechercher dans la collection"""
        
        query_embedding = self.model.encode([query])[0]
        
        results = self.client.search(
            collection_name=self.collection_name,
            query_vector=query_embedding.tolist(),
            query_filter=filter,
            limit=limit
        )
        
        return results

# Utilisation
indexer = DocumentIndexer()

# Charger des données
df = pd.read_csv("products.csv")

# Indexer
indexer.index_dataframe(df, text_column="description", id_column="product_id")

# Rechercher
results = indexer.search("laptop computer", limit=5)
```

---

## Optimisation batch

### Batch encoding

```python
# Plus efficace : encoder par batches
texts = ["text1", "text2", ..., "text1000"]

# ✅ Bon : batch encoding
embeddings = model.encode(texts, batch_size=32, show_progress_bar=True)

# ❌ Moins bon : un par un
embeddings = [model.encode([text])[0] for text in texts]
```

### Batch insertion

```python
def efficient_indexing(documents, batch_size=100):
    """Indexation efficace par batches"""
    
    model = SentenceTransformer('all-MiniLM-L6-v2')
    client = QdrantClient(host="localhost", port=6333)
    
    # Encoder tous les documents
    texts = [doc["text"] for doc in documents]
    embeddings = model.encode(texts, batch_size=32, show_progress_bar=True)
    
    # Insérer par batches dans Qdrant
    for i in range(0, len(documents), batch_size):
        batch_docs = documents[i:i+batch_size]
        batch_embeddings = embeddings[i:i+batch_size]
        
        points = [
            PointStruct(
                id=doc["id"],
                vector=emb.tolist(),
                payload={"text": doc["text"]}
            )
            for doc, emb in zip(batch_docs, batch_embeddings)
        ]
        
        client.upsert(collection_name="documents", points=points)
```

### Utilisation du GPU

```python
import torch

# Vérifier si GPU disponible
device = "cuda" if torch.cuda.is_available() else "cpu"
print(f"Using device: {device}")

# Charger le modèle sur GPU
model = SentenceTransformer('all-MiniLM-L6-v2', device=device)

# Encoder (plus rapide sur GPU)
embeddings = model.encode(texts, batch_size=64)  # Batch plus grand sur GPU
```

---

## Gestion des erreurs

### Gestion robuste

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
import time

def robust_indexing(documents, max_retries=3):
    """Indexation avec gestion d'erreurs"""
    
    model = SentenceTransformer('all-MiniLM-L6-v2')
    client = QdrantClient(host="localhost", port=6333)
    
    for doc in documents:
        retries = 0
        while retries < max_retries:
            try:
                # Générer embedding
                embedding = model.encode([doc["text"]])[0]
                
                # Insérer
                point = PointStruct(
                    id=doc["id"],
                    vector=embedding.tolist(),
                    payload={"text": doc["text"]}
                )
                
                client.upsert(collection_name="documents", points=[point])
                break  # Succès
                
            except Exception as e:
                retries += 1
                print(f"Error indexing doc {doc['id']}: {e}")
                if retries < max_retries:
                    time.sleep(2 ** retries)  # Backoff exponentiel
                else:
                    print(f"Failed to index doc {doc['id']} after {max_retries} retries")
```

---

## Exercices pratiques

### Exercice 1 : Pipeline d'indexation

Créer un pipeline qui :
1. Lit un fichier CSV
2. Génère des embeddings
3. Indexe dans Qdrant
4. Affiche la progression

**Solution :**

```python
import pandas as pd
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct

def index_csv(csv_path, text_column, id_column):
    model = SentenceTransformer('all-MiniLM-L6-v2')
    client = QdrantClient(host="localhost", port=6333)
    
    df = pd.read_csv(csv_path)
    
    # Encoder tous les textes
    texts = df[text_column].tolist()
    embeddings = model.encode(texts, show_progress_bar=True)
    
    # Créer et insérer les points
    points = [
        PointStruct(
            id=int(row[id_column]),
            vector=emb.tolist(),
            payload=row.to_dict()
        )
        for row, emb in zip(df.itertuples(), embeddings)
    ]
    
    client.upsert(collection_name="documents", points=points)
    print(f"Indexed {len(points)} documents")
```

---

## 🎯 Points clés à retenir

✅ sentence-transformers est idéal pour les embeddings de texte  
✅ Utiliser batch encoding pour de meilleures performances  
✅ GPU accélère significativement l'encodage  
✅ Gérer les erreurs avec retry logic  
✅ Créer des pipelines réutilisables pour l'indexation  

---

**Prochaine étape :** [Cas d'usage IA/ML](./06-ai-ml-use-cases/README.md)
