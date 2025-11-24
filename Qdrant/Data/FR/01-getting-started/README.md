# 1. Prise en main Qdrant

## 🎯 Objectifs

- Comprendre les bases de données vectorielles
- Installer Qdrant
- Comprendre les concepts de base
- Premières opérations
- Vérifier l'installation

## 📋 Table des matières

1. [Introduction à Qdrant](#introduction-à-qdrant)
2. [Bases de données vectorielles](#bases-de-données-vectorielles)
3. [Installation](#installation)
4. [Premier exemple](#premier-exemple)
5. [Interface Web](#interface-web)
6. [Vérification](#vérification)

---

## Introduction à Qdrant

### Qu'est-ce que Qdrant ?

**Qdrant** = Base de données vectorielle open-source

- **Vecteurs** : Représentations numériques (embeddings)
- **Similarité** : Recherche par similarité ultra-rapide
- **IA/ML** : Optimisé pour l'intelligence artificielle
- **Open-source** : Gratuit et open-source
- **Scalable** : Gère des millions de vecteurs
- **Production-ready** : Utilisé en production

### Caractéristiques principales

- **Recherche rapide** : Algorithmes HNSW optimisés
- **Filtres avancés** : Filtrage par métadonnées
- **API REST** : Interface HTTP simple
- **Python/Java/Go** : Clients officiels
- **Docker** : Déploiement facile

### Pourquoi Qdrant pour Data Analyst ?

- **Recherche sémantique** : Recherche par sens, pas par mots-clés
- **Recommandations** : Systèmes de recommandation performants
- **IA/ML** : Intégration native avec modèles d'embeddings
- **Performance** : Recherche rapide sur millions de vecteurs
- **Flexibilité** : Filtres complexes sur métadonnées
- **Cas d'usage modernes** : RAG, chatbots, recherche intelligente

### Cas d'usage typiques

- **Recherche sémantique** : Documents, produits, contenu
- **Recommandations** : Produits similaires, contenu personnalisé
- **Déduplication** : Détecter les doublons
- **Clustering** : Grouper des éléments similaires
- **RAG** : Retrieval-Augmented Generation pour LLMs
- **Anomaly detection** : Détecter les anomalies

---

## Bases de données vectorielles

### Qu'est-ce qu'un vecteur ?

Un **vecteur** est une liste de nombres qui représente un objet :

```python
# Exemple de vecteur (embedding) de dimension 128
vector = [0.1, -0.3, 0.7, 0.2, ..., 0.5]  # 128 nombres
```

### Qu'est-ce qu'un embedding ?

Un **embedding** est une représentation vectorielle d'un objet :
- **Texte** → Vecteur de nombres
- **Image** → Vecteur de nombres
- **Audio** → Vecteur de nombres

### Exemple : Embedding de texte

```python
# Texte original
text = "Laptop computer for work"

# Embedding (simplifié, dimension 4 pour l'exemple)
embedding = [0.2, -0.1, 0.8, 0.3]

# Texte similaire
similar_text = "Computer laptop professional"

# Embedding similaire (proche dans l'espace vectoriel)
similar_embedding = [0.25, -0.08, 0.75, 0.28]
```

### Similarité vectorielle

La **similarité** mesure à quel point deux vecteurs sont proches :

- **Distance cosinus** : Angle entre vecteurs (0-1, 1 = identique)
- **Distance euclidienne** : Distance dans l'espace (0-∞, 0 = identique)
- **Produit scalaire** : Projection d'un vecteur sur l'autre

### Pourquoi une base de données vectorielle ?

**Problème avec les bases de données classiques :**
- Recherche par mots-clés exacts
- Ne comprend pas le sens
- Pas optimisée pour la similarité

**Solution avec Qdrant :**
- Recherche par sens (sémantique)
- Comprend les relations
- Optimisée pour la similarité

---

## Installation

### Option 1 : Docker (recommandé)

#### Installation Docker

Si Docker n'est pas installé :
- Windows : [Docker Desktop](https://www.docker.com/products/docker-desktop)
- Linux : `sudo apt-get install docker.io`
- Mac : [Docker Desktop](https://www.docker.com/products/docker-desktop)

#### Lancer Qdrant

```bash
# Lancer Qdrant
docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant

# Ou avec volume persistant
docker run -p 6333:6333 -p 6334:6334 \
  -v $(pwd)/qdrant_storage:/qdrant/storage \
  qdrant/qdrant
```

#### Vérifier que Qdrant tourne

```bash
# Vérifier les conteneurs
docker ps

# Vérifier les logs
docker logs <container_id>
```

### Option 2 : Installation native (Linux)

```bash
# Télécharger Qdrant
wget https://github.com/qdrant/qdrant/releases/download/v1.7.0/qdrant-x86_64-unknown-linux-gnu

# Rendre exécutable
chmod +x qdrant-x86_64-unknown-linux-gnu

# Lancer
./qdrant-x86_64-unknown-linux-gnu
```

### Option 3 : Qdrant Cloud (gratuit)

1. Créer un compte sur [Qdrant Cloud](https://cloud.qdrant.io/)
2. Créer un cluster gratuit
3. Obtenir l'URL et la clé API

---

## Installation du client Python

### Avec pip

```bash
# Installation de base
pip install qdrant-client

# Avec dépendances optionnelles
pip install qdrant-client[fastembed]
```

### Avec conda

```bash
conda install -c conda-forge qdrant-client
```

### Vérifier l'installation

```python
import qdrant_client
print(qdrant_client.__version__)
```

---

## Premier exemple

### Connexion à Qdrant

```python
from qdrant_client import QdrantClient

# Connexion locale
client = QdrantClient(host="localhost", port=6333)

# Ou avec URL complète
client = QdrantClient(url="http://localhost:6333")

# Connexion distante
client = QdrantClient(
    url="https://your-cluster.qdrant.io",
    api_key="your-api-key"
)
```

### Créer une collection

```python
from qdrant_client.models import Distance, VectorParams

# Créer une collection simple
client.create_collection(
    collection_name="test_collection",
    vectors_config=VectorParams(
        size=128,  # Dimension des vecteurs
        distance=Distance.COSINE  # Type de distance
    )
)

print("Collection créée avec succès!")
```

### Vérifier la collection

```python
# Lister toutes les collections
collections = client.get_collections()
print(f"Collections: {collections.collections}")

# Obtenir les informations d'une collection
collection_info = client.get_collection("test_collection")
print(f"Collection info: {collection_info}")
```

### Insérer un premier vecteur

```python
from qdrant_client.models import PointStruct
import random

# Générer un vecteur aléatoire (exemple)
vector = [random.random() for _ in range(128)]

# Créer un point
point = PointStruct(
    id=1,
    vector=vector,
    payload={
        "name": "Premier document",
        "category": "test"
    }
)

# Insérer le point
client.upsert(
    collection_name="test_collection",
    points=[point]
)

print("Point inséré avec succès!")
```

### Première recherche

```python
# Rechercher des vecteurs similaires
results = client.search(
    collection_name="test_collection",
    query_vector=vector,  # Vecteur de requête
    limit=5  # Nombre de résultats
)

# Afficher les résultats
for result in results:
    print(f"ID: {result.id}, Score: {result.score:.4f}")
    print(f"Payload: {result.payload}")
```

---

## Interface Web

### Accéder à l'interface

Une fois Qdrant lancé, accédez à :
- **Interface Web** : http://localhost:6333/dashboard
- **API REST** : http://localhost:6333/docs

### Fonctionnalités de l'interface

- **Collections** : Voir et gérer les collections
- **Points** : Visualiser les points et vecteurs
- **Recherche** : Tester des recherches
- **Métriques** : Statistiques et performances

---

## Vérification

### Script de vérification complet

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
import random

def test_qdrant_installation():
    """Test complet de l'installation Qdrant"""
    
    # 1. Connexion
    try:
        client = QdrantClient(host="localhost", port=6333)
        print("✅ Connexion réussie")
    except Exception as e:
        print(f"❌ Erreur de connexion: {e}")
        return False
    
    # 2. Créer une collection de test
    try:
        client.create_collection(
            collection_name="test_installation",
            vectors_config=VectorParams(size=128, distance=Distance.COSINE)
        )
        print("✅ Collection créée")
    except Exception as e:
        print(f"⚠️ Collection existe peut-être déjà: {e}")
    
    # 3. Insérer des points de test
    try:
        points = [
            PointStruct(
                id=i,
                vector=[random.random() for _ in range(128)],
                payload={"test": f"point_{i}"}
            )
            for i in range(10)
        ]
        client.upsert(collection_name="test_installation", points=points)
        print("✅ Points insérés")
    except Exception as e:
        print(f"❌ Erreur insertion: {e}")
        return False
    
    # 4. Recherche
    try:
        query_vector = [random.random() for _ in range(128)]
        results = client.search(
            collection_name="test_installation",
            query_vector=query_vector,
            limit=5
        )
        print(f"✅ Recherche réussie ({len(results)} résultats)")
    except Exception as e:
        print(f"❌ Erreur recherche: {e}")
        return False
    
    # 5. Nettoyage
    try:
        client.delete_collection("test_installation")
        print("✅ Collection de test supprimée")
    except Exception as e:
        print(f"⚠️ Erreur suppression: {e}")
    
    print("\n🎉 Installation Qdrant vérifiée avec succès!")
    return True

if __name__ == "__main__":
    test_qdrant_installation()
```

### Exécuter le test

```bash
python test_qdrant.py
```

---

## Exercices pratiques

### Exercice 1 : Installation et première collection

1. Installer Qdrant avec Docker
2. Créer une collection nommée "exercice1" avec des vecteurs de dimension 64
3. Vérifier que la collection existe

**Solution :**

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams

client = QdrantClient(host="localhost", port=6333)

client.create_collection(
    collection_name="exercice1",
    vectors_config=VectorParams(size=64, distance=Distance.COSINE)
)

# Vérifier
collections = client.get_collections()
print("exercice1" in [c.name for c in collections.collections])
```

### Exercice 2 : Insérer et rechercher

1. Insérer 5 points avec des vecteurs aléatoires
2. Effectuer une recherche avec un nouveau vecteur
3. Afficher les 3 meilleurs résultats

**Solution :**

```python
from qdrant_client.models import PointStruct
import random

# Insérer 5 points
points = [
    PointStruct(
        id=i,
        vector=[random.random() for _ in range(64)],
        payload={"name": f"Item {i}"}
    )
    for i in range(5)
]
client.upsert(collection_name="exercice1", points=points)

# Rechercher
query_vector = [random.random() for _ in range(64)]
results = client.search(
    collection_name="exercice1",
    query_vector=query_vector,
    limit=3
)

for result in results:
    print(f"ID: {result.id}, Score: {result.score:.4f}, Name: {result.payload['name']}")
```

---

## 🎯 Points clés à retenir

✅ Qdrant est une base de données vectorielle optimisée pour la recherche par similarité  
✅ Les vecteurs (embeddings) représentent des objets sous forme numérique  
✅ La similarité mesure la proximité entre vecteurs  
✅ Qdrant peut être installé avec Docker, nativement ou via le cloud  
✅ L'interface web permet de visualiser et tester les collections  

---

## 📚 Ressources complémentaires

- [Documentation officielle Qdrant](https://qdrant.tech/documentation/)
- [Guide de démarrage rapide](https://qdrant.tech/documentation/quick-start/)
- [API REST](https://qdrant.github.io/qdrant/redoc/index.html)
- [Exemples Python](https://github.com/qdrant/qdrant-client/tree/master/examples)

---

**Prochaine étape :** [Collections et Vecteurs](./02-collections-vectors/README.md)

