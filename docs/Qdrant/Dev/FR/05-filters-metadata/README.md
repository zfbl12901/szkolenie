# 5. Filtres et Métadonnées

## 🎯 Objectifs

- Utiliser les filtres avancés
- Gérer les métadonnées
- Combiner plusieurs conditions

## Filtres simples

```java
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
```

## Range filters

```java
Filter filter = Filter.newBuilder()
    .addMust(Condition.newBuilder()
        .setField(FieldCondition.newBuilder()
            .setKey("price")
            .setRange(Range.newBuilder()
                .setGte(100.0)
                .setLte(500.0)
                .build())
            .build())
        .build())
    .build();
```

## Mettre à jour le payload

```java
SetPayloadPoints setPayload = SetPayloadPoints.newBuilder()
    .setCollectionName("products")
    .putPayload("discount", Value.newBuilder().setDoubleValue(0.1).build())
    .addAllPoints(Arrays.asList(1L, 2L, 3L))
    .build();

client.setPayload(setPayload);
```

---

**Prochaine étape :** [Performance et Optimisation](./06-performance/README.md)

