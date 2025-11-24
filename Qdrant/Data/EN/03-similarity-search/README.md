# 3. Similarity Search

## 🎯 Objectives

- Perform similarity searches
- Use different algorithms
- Optimize performance
- Understand scores

## Simple Search

```python
results = client.search(
    collection_name="products",
    query_vector=[0.1, 0.2, 0.3, ...],
    limit=10
)

for result in results:
    print(f"ID: {result.id}, Score: {result.score}")
```

## Search with Filters

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

results = client.search(
    collection_name="products",
    query_vector=[0.1, 0.2, 0.3, ...],
    query_filter=Filter(
        must=[
            FieldCondition(
                key="category",
                match=MatchValue(value="electronics")
            )
        ]
    ),
    limit=10
)
```

---

**Next step :** [Filters and Metadata](./04-filters-metadata/README.md)

