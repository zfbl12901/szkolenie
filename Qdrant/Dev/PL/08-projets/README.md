# 8. Projekty praktyczne

## 🎯 Cele

- Tworzyć kompletną aplikację
- Integrować Qdrant w rzeczywistym projekcie
- Stosować najlepsze praktyki

## Projekt 1 : Serwis wyszukiwania semantycznego

```java
@Service
public class SemanticSearchService {
    private final QdrantClient client;
    
    public List<SearchResult> search(String query, int limit) {
        List<Float> queryVector = embeddingModel.encode(query);
        SearchPoints searchPoints = SearchPoints.newBuilder()
            .setCollectionName("documents")
            .addAllVector(queryVector)
            .setLimit(limit)
            .build();
        return mapToSearchResults(client.search(searchPoints).getResultList());
    }
}
```

---

**Gratulacje! Ukończyłeś szkolenie Qdrant dla Dewelopera Java! 🎉**

