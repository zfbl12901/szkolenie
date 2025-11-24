# 6. AI/ML Use Cases

## 🎯 Objectives

- Understand typical use cases
- Implement semantic search
- Create recommendation systems
- Use with RAG

## Semantic Search

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

## Recommendation System

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

**Next step :** [Best Practices](./07-best-practices/README.md)

