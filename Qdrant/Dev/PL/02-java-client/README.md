# 2. Klient Java i połączenie

## 🎯 Cele

- Skonfigurować połączenie
- Zarządzać klientem
- Zrozumieć opcje połączenia

## Połączenie

```java
QdrantClient client = new QdrantClient(
    QdrantClient.newBuilder("localhost", 6334, false).build()
);

// Z uwierzytelnianiem
QdrantClient client = new QdrantClient(
    QdrantClient.newBuilder("localhost", 6334, false)
        .withApiKey("your-api-key")
        .build()
);
```

---

**Następny krok :** [Kolekcje i wektory](./03-collections-vectors/README.md)

