# 1. Getting Started with Qdrant

## 🎯 Objectives

- Understand vector databases
- Install Qdrant
- Understand basic concepts
- First operations

## Introduction to Qdrant

**Qdrant** = Vector database

- **Vectors** : Numerical representations (embeddings)
- **Similarity** : Similarity search
- **AI/ML** : Optimized for artificial intelligence
- **Open-source** : Free and open-source

## Installation

### Docker (recommended)

```bash
docker run -p 6333:6333 qdrant/qdrant
```

### Python Client

```bash
pip install qdrant-client
```

## First Example

```python
from qdrant_client import QdrantClient

client = QdrantClient(host="localhost", port=6333)

client.create_collection(
    collection_name="test_collection",
    vectors_config={
        "size": 128,
        "distance": "Cosine"
    }
)
```

---

**Next step :** [Collections and Vectors](./02-collections-vectors/README.md)

