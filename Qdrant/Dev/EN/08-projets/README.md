# 8. Practical Projects

## 🎯 Objectives

- Create complete application
- Integrate Qdrant in real project
- Apply best practices

## Project 1 : Semantic Search Service

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

**Congratulations! You have completed the Qdrant training for Java Developer! 🎉**

