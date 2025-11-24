# 7. Najlepsze praktyki

## 🎯 Cele

- Strukturyzować kod
- Zarządzać zasobami
- Optymalizować wydajność

## Warstwa serwisowa

```java
public class QdrantService {
    private final QdrantClient client;
    
    public List<ScoredPoint> search(String collection, List<Float> vector, int limit) {
        SearchPoints searchPoints = SearchPoints.newBuilder()
            .setCollectionName(collection)
            .addAllVector(vector)
            .setLimit(limit)
            .build();
        return client.search(searchPoints).getResultList();
    }
}
```

---

**Następny krok :** [Projekty praktyczne](./08-projets/README.md)

