# 3. Kolekcje i wektory

## 🎯 Cele

- Tworzyć kolekcje
- Wstawiać wektory
- Zarządzać kolekcjami

## Utworzenie kolekcji

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

## Wstawienie wektorów

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

**Następny krok :** [Wyszukiwanie podobieństwa](./04-similarity-search/README.md)

