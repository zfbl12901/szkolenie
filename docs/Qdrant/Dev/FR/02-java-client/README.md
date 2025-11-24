# 2. Client Java et Connexion

## 🎯 Objectifs

- Configurer la connexion
- Gérer le client
- Comprendre les options de connexion

## Connexion

```java
import io.qdrant.client.QdrantClient;

// Connexion simple
QdrantClient client = new QdrantClient(
    QdrantClient.newBuilder("localhost", 6334, false).build()
);

// Avec authentification
QdrantClient client = new QdrantClient(
    QdrantClient.newBuilder("localhost", 6334, false)
        .withApiKey("your-api-key")
        .build()
);
```

## Gestion du client

```java
public class QdrantManager {
    private static QdrantClient client;
    
    public static QdrantClient getClient() {
        if (client == null) {
            client = new QdrantClient(
                QdrantClient.newBuilder("localhost", 6334, false).build()
            );
        }
        return client;
    }
    
    public static void close() {
        if (client != null) {
            client.close();
        }
    }
}
```

---

**Prochaine étape :** [Collections et Vecteurs](./03-collections-vectors/README.md)

