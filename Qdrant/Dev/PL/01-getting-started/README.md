# 1. Rozpoczęcie pracy z Qdrant i Java

## 🎯 Cele

- Zrozumieć Qdrant i Java
- Skonfigurować środowisko
- Zainstalować klienta Java
- Utworzyć pierwszy projekt

## Konfiguracja Maven

```xml
<dependencies>
    <dependency>
        <groupId>io.qdrant</groupId>
        <artifactId>qdrant-java-client</artifactId>
        <version>1.7.0</version>
    </dependency>
</dependencies>
```

## Pierwszy przykład

```java
import io.qdrant.client.QdrantClient;

QdrantClient client = new QdrantClient(
    QdrantClient.newBuilder("localhost", 6334, false).build()
);

System.out.println("Połączono z Qdrant!");
client.close();
```

---

**Następny krok :** [Klient Java i połączenie](./02-java-client/README.md)

