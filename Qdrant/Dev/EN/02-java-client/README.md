# 2. Java Client and Connection

## 🎯 Objectives

- Configure connection
- Manage client
- Understand connection options

## Connection

```java
QdrantClient client = new QdrantClient(
    QdrantClient.newBuilder("localhost", 6334, false).build()
);

// With authentication
QdrantClient client = new QdrantClient(
    QdrantClient.newBuilder("localhost", 6334, false)
        .withApiKey("your-api-key")
        .build()
);
```

---

**Next step :** [Collections and Vectors](./03-collections-vectors/README.md)

