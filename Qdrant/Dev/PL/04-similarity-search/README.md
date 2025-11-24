# 4. Wyszukiwanie podobieństwa

## 🎯 Cele

- Wykonywać wyszukiwania
- Używać różnych algorytmów
- Obsługiwać wyniki

## Proste wyszukiwanie

```java
List<Float> queryVector = Arrays.asList(0.1f, 0.2f, 0.3f, ...);

SearchPoints searchPoints = SearchPoints.newBuilder()
    .setCollectionName("products")
    .addAllVector(queryVector)
    .setLimit(10)
    .build();

List<ScoredPoint> results = client.search(searchPoints).getResultList();
```

---

**Następny krok :** [Filtry i metadane](./05-filters-metadata/README.md)

