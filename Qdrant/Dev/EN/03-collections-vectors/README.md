# 3. Collections and Vectors

## 🎯 Objectives

- Create collections
- Insert vectors
- Manage collections

## Create Collection

```java
CreateCollection createCollection = CreateCollection.newBuilder()
    .setCollectionName("products")
    .setVectorsConfig(VectorsConfig.newBuilder()
        .setParams(VectorParams.newBuilder()
            .setSize(128)
            .setDistance(Distance.Cosine)
            .build())
        .build())
    .build();

client.createCollection(createCollection);
```

## Insert Vectors

```java
List<PointStruct> points = new ArrayList<>();
points.add(PointStruct.newBuilder()
    .setId(1)
    .setVectors(Vectors.newBuilder()
        .setVector(Vector.newBuilder()
            .addAllData(Arrays.asList(0.1f, 0.2f, 0.3f, ...))
            .build())
        .build())
    .putPayload("name", Value.newBuilder().setStringValue("Product A").build())
    .build());

client.upsert(UpsertPoints.newBuilder()
    .setCollectionName("products")
    .addAllPoints(points)
    .build());
```

---

**Next step :** [Similarity Search](./04-similarity-search/README.md)

