# 5. Filtry i metadane

## 🎯 Cele

- Używać zaawansowanych filtrów
- Zarządzać metadanymi
- Łączyć wiele warunków

## Proste filtry

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

---

**Następny krok :** [Wydajność i optymalizacja](./06-performance/README.md)

