# 4. Recherche par Similarité

## 🎯 Objectifs

- Effectuer des recherches
- Utiliser différents algorithmes
- Gérer les résultats

## Recherche simple

```java
import io.qdrant.client.grpc.Points.*;

List<Float> queryVector = Arrays.asList(0.1f, 0.2f, 0.3f, ...);

SearchPoints searchPoints = SearchPoints.newBuilder()
    .setCollectionName("products")
    .addAllVector(queryVector)
    .setLimit(10)
    .build();

List<ScoredPoint> results = client.search(searchPoints).getResultList();

for (ScoredPoint result : results) {
    System.out.println("ID: " + result.getId() + ", Score: " + result.getScore());
}
```

## Recherche avec filtres

```java
import io.qdrant.client.grpc.Points.*;

Filter filter = Filter.newBuilder()
    .addMust(Condition.newBuilder()
        .setField(FieldCondition.newBuilder()
            .setKey("category")
            .setMatch(Match.newBuilder()
                .setValue(Value.newBuilder()
                    .setStringValue("electronics")
                    .build())
                .build())
            .build())
        .build())
    .build();

SearchPoints searchPoints = SearchPoints.newBuilder()
    .setCollectionName("products")
    .addAllVector(queryVector)
    .setFilter(filter)
    .setLimit(10)
    .build();
```

---

**Prochaine étape :** [Filtres et Métadonnées](./05-filters-metadata/README.md)

