# 6. Przypadki użycia AI/ML

## 🎯 Cele

- Zrozumieć typowe przypadki użycia
- Implementować wyszukiwanie semantyczne
- Tworzyć systemy rekomendacji
- Używać z RAG

## Wyszukiwanie semantyczne

```python
def semantic_search(query_text, limit=10):
    query_embedding = model.encode([query_text])[0]
    results = client.search(
        collection_name="documents",
        query_vector=query_embedding.tolist(),
        limit=limit
    )
    return results
```

## System rekomendacji

```python
def recommend_similar_products(product_id, limit=5):
    product = client.retrieve(
        collection_name="products",
        ids=[product_id]
    )[0]
    
    results = client.search(
        collection_name="products",
        query_vector=product.vector,
        limit=limit
    )
    return results
```

---

**Następny krok :** [Najlepsze praktyki](./07-best-practices/README.md)

