# 5. Integracja Python

## 🎯 Cele

- Integrować z modelami embeddings
- Używać z frameworkami AI
- Tworzyć pipeline danych
- Optymalizować wydajność

## Z sentence-transformers

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient

model = SentenceTransformer('all-MiniLM-L6-v2')
texts = ["Opis produktu 1", "Opis produktu 2"]
embeddings = model.encode(texts)

client = QdrantClient(host="localhost", port=6333)
client.upsert(
    collection_name="products",
    points=[
        PointStruct(id=i, vector=emb.tolist(), payload={"text": text})
        for i, (emb, text) in enumerate(zip(embeddings, texts))
    ]
)
```

---

**Następny krok :** [Przypadki użycia AI/ML](./06-ai-ml-use-cases/README.md)

