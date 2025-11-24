# 4. Filters and Metadata

## 🎯 Objectives

- Use advanced filters
- Manage metadata (payload)
- Combine multiple conditions
- Optimize filtered queries

## Simple Filters

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

filter = Filter(
    must=[
        FieldCondition(
            key="category",
            match=MatchValue(value="electronics")
        )
    ]
)
```

## Range Filters

```python
from qdrant_client.models import Range

filter = Filter(
    must=[
        FieldCondition(
            key="price",
            range=Range(gte=100, lte=500)
        )
    ]
)
```

---

**Next step :** [Python Integration](./05-python-integration/README.md)

