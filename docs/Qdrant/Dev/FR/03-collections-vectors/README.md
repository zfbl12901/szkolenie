# 3. Collections et Vecteurs

## 🎯 Objectifs

- Créer des collections
- Insérer des vecteurs
- Gérer les collections

## Créer une collection

```java
import io.qdrant.client.QdrantClient;
import io.qdrant.client.grpc.Collections.*;

QdrantClient client = QdrantManager.getClient();

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

## Insérer des vecteurs

```java
import io.qdrant.client.grpc.Points.*;

List<PointStruct> points = new ArrayList<>();
points.add(PointStruct.newBuilder()
    .setId(1)
    .setVectors(Vectors.newBuilder()
        .setVector(Vector.newBuilder()
            .addAllData(Arrays.asList(0.1f, 0.2f, 0.3f, ...))
            .build())
        .build())
    .putPayload("name", Value.newBuilder().setStringValue("Product A").build())
    .putPayload("category", Value.newBuilder().setStringValue("electronics").build())
    .build());

client.upsert(UpsertPoints.newBuilder()
    .setCollectionName("products")
    .addAllPoints(points)
    .build());
```

---

**Prochaine étape :** [Recherche par Similarité](./04-similarity-search/README.md)

