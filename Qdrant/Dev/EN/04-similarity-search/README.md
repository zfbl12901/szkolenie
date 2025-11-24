# 4. Similarity Search

## 🎯 Objectives

- Perform searches
- Use different algorithms
- Handle results

## Simple Search

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

**Next step :** [Filters and Metadata](./05-filters-metadata/README.md)

