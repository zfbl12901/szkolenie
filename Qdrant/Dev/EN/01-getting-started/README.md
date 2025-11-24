# 1. Getting Started with Qdrant and Java

## 🎯 Objectives

- Understand Qdrant and Java
- Set up environment
- Install Java client
- Create first project

## Maven Configuration

```xml
<dependencies>
    <dependency>
        <groupId>io.qdrant</groupId>
        <artifactId>qdrant-java-client</artifactId>
        <version>1.7.0</version>
    </dependency>
</dependencies>
```

## First Example

```java
import io.qdrant.client.QdrantClient;

QdrantClient client = new QdrantClient(
    QdrantClient.newBuilder("localhost", 6334, false).build()
);

System.out.println("Connected to Qdrant!");
client.close();
```

---

**Next step :** [Java Client and Connection](./02-java-client/README.md)

