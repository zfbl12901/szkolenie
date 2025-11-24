# 7. Best Practices

## 🎯 Objectives

- Structure code
- Manage resources
- Optimize performance

## Service Layer

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

**Next step :** [Practical Projects](./08-projets/README.md)

