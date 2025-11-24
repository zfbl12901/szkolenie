# 6. Wydajność i optymalizacja

## 🎯 Cele

- Optymalizować wydajność
- Używać przetwarzania wsadowego
- Zarządzać pamięcią

## Wstawianie wsadowe

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

**Następny krok :** [Najlepsze praktyki](./07-best-practices/README.md)

