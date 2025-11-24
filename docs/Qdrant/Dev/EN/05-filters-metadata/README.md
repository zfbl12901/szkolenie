# 5. Filters and Metadata

## 🎯 Objectives

- Use advanced filters
- Manage metadata
- Combine multiple conditions

## Simple Filters

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

**Next step :** [Performance and Optimization](./06-performance/README.md)

