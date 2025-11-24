# 8. Practical Projects

## 🎯 Objectives

- Create complete projects
- Apply knowledge
- Integrate with AI
- Optimize performance

## Project 1 : Semantic Search Engine

```python
class SemanticSearchEngine:
    def __init__(self):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.client = QdrantClient(host="localhost", port=6333)
    
    def search(self, query, limit=10):
        query_embedding = self.model.encode([query])[0]
        results = self.client.search(
            collection_name="documents",
            query_vector=query_embedding.tolist(),
            limit=limit
        )
        return results
```

## Project 2 : Recommendation System

```python
def recommend_products(user_id, limit=10):
    user_history = get_user_history(user_id)
    user_embedding = model.encode(user_history).mean(axis=0)
    results = client.search(
        collection_name="products",
        query_vector=user_embedding.tolist(),
        limit=limit
    )
    return results
```

---

**Congratulations! You have completed the Qdrant training for Data Analyst! 🎉**

