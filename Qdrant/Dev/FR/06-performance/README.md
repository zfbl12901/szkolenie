# 6. Performance et Optimisation

## 🎯 Objectifs

- Optimiser les performances
- Utiliser le batch processing
- Gérer la mémoire

## Batch insert

```java
List<PointStruct> points = new ArrayList<>();
// Ajouter plusieurs points
for (int i = 0; i < 1000; i++) {
    points.add(createPoint(i, generateVector()));
}

client.upsert(UpsertPoints.newBuilder()
    .setCollectionName("products")
    .addAllPoints(points)
    .build());
```

## Configuration HNSW

```java
HnswConfigDiff hnswConfig = HnswConfigDiff.newBuilder()
    .setM(16)
    .setEfConstruct(100)
    .build();

CreateCollection createCollection = CreateCollection.newBuilder()
    .setCollectionName("products")
    .setVectorsConfig(VectorsConfig.newBuilder()
        .setParams(VectorParams.newBuilder()
            .setSize(128)
            .setDistance(Distance.Cosine)
            .build())
        .build())
    .setHnswConfig(hnswConfig)
    .build();
```

---

**Prochaine étape :** [Bonnes Pratiques](./07-best-practices/README.md)

