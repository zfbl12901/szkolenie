# 1. Prise en main Qdrant avec Java

## 🎯 Objectifs

- Comprendre Qdrant et Java
- Configurer l'environnement
- Installer le client Java
- Créer un premier projet

## Configuration Maven

### pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>io.qdrant</groupId>
        <artifactId>qdrant-java-client</artifactId>
        <version>1.7.0</version>
    </dependency>
</dependencies>
```

## Premier exemple

```java
import io.qdrant.client.QdrantClient;
import io.qdrant.client.grpc.Collections;

public class QdrantExample {
    public static void main(String[] args) {
        QdrantClient client = new QdrantClient(
            QdrantClient.newBuilder("localhost", 6334, false).build()
        );
        
        System.out.println("Connected to Qdrant!");
        client.close();
    }
}
```

---

**Prochaine étape :** [Client Java et Connexion](./02-java-client/README.md)

