# 7. Bonnes Pratiques

## 🎯 Objectifs

- Structurer le code
- Gérer les ressources
- Optimiser les performances

## Service Layer

```java
public class QdrantService {
    private final QdrantClient client;
    
    public QdrantService(QdrantClient client) {
        this.client = client;
    }
    
    public List<ScoredPoint> search(String collection, List<Float> vector, int limit) {
        SearchPoints searchPoints = SearchPoints.newBuilder()
            .setCollectionName(collection)
            .addAllVector(vector)
            .setLimit(limit)
            .build();
        return client.search(searchPoints).getResultList();
    }
}
```

## Gestion des ressources

```java
// ✅ Bon
try (QdrantClient client = new QdrantClient(...)) {
    // Utiliser le client
}

// Ou fermer explicitement
QdrantClient client = new QdrantClient(...);
try {
    // Utiliser
} finally {
    client.close();
}
```

---

**Prochaine étape :** [Projets Pratiques](./08-projets/README.md)

