# 5. Python Integration

## 🎯 Objectives

- Integrate with embedding models
- Use with AI frameworks
- Create data pipelines
- Optimize performance

## With sentence-transformers

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient

model = SentenceTransformer('all-MiniLM-L6-v2')
texts = ["Product description 1", "Product description 2"]
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

**Next step :** [AI/ML Use Cases](./06-ai-ml-use-cases/README.md)

