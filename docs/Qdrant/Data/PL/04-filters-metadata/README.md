# 4. Filtry i metadane

## 🎯 Cele

- Używać zaawansowanych filtrów
- Zarządzać metadanymi (payload)
- Łączyć wiele warunków
- Optymalizować zapytania z filtrami

## Proste filtry

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

## Filtry zakresu

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

**Następny krok :** [Integracja Python](./05-python-integration/README.md)

