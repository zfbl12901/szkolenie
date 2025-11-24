# 6. Performance and Optimization

## 🎯 Objectives

- Optimize performance
- Use batch processing
- Manage memory

## Batch Insert

```java
List<PointStruct> points = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    points.add(createPoint(i, generateVector()));
}

client.upsert(UpsertPoints.newBuilder()
    .setCollectionName("products")
    .addAllPoints(points)
    .build());
```

---

**Next step :** [Best Practices](./07-best-practices/README.md)

