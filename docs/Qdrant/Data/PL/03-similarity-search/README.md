# 3. Wyszukiwanie podobieństwa

## 🎯 Cele

- Wykonywać wyszukiwania podobieństwa
- Używać różnych algorytmów
- Optymalizować wydajność
- Zrozumieć wyniki

## Proste wyszukiwanie

```python
results = client.search(
    collection_name="products",
    query_vector=[0.1, 0.2, 0.3, ...],
    limit=10
)

for result in results:
    print(f"ID: {result.id}, Wynik: {result.score}")
```

## Wyszukiwanie z filtrami

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

**Następny krok :** [Filtry i metadane](./04-filters-metadata/README.md)

