# 2. Kolekcje i wektory

## 🎯 Cele

- Tworzyć i zarządzać kolekcjami
- Wstawiać wektory
- Zrozumieć odległości
- Zarządzać metadanymi

## Utworzenie kolekcji

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams

client = QdrantClient(host="localhost", port=6333)

client.create_collection(
    collection_name="products",
    vectors_config=VectorParams(
        size=128,
        distance=Distance.COSINE
    )
)
```

## Wstawienie wektorów

```python
from qdrant_client.models import PointStruct

points = [
    PointStruct(
        id=1,
        vector=[0.1, 0.2, 0.3, ...],
        payload={"name": "Product A", "category": "electronics"}
    )
]

client.upsert(collection_name="products", points=points)
```

---

**Następny krok :** [Wyszukiwanie podobieństwa](./03-similarity-search/README.md)

