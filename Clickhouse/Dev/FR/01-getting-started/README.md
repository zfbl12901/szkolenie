# 1. Prise en main ClickHouse avec Java

## 🎯 Objectifs

- Comprendre ClickHouse et Java
- Configurer l'environnement de développement
- Installer le driver JDBC
- Créer un premier projet

## Introduction

ClickHouse peut être utilisé avec Java via :
- **JDBC Driver** : Connexion standard JDBC
- **HTTP Interface** : Requêtes HTTP REST
- **Native Protocol** : Protocole natif ClickHouse

## Configuration Maven

### pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>com.clickhouse</groupId>
        <artifactId>clickhouse-jdbc</artifactId>
        <version>0.6.0</version>
    </dependency>
</dependencies>
```

## Configuration Gradle

### build.gradle

```gradle
dependencies {
    implementation 'com.clickhouse:clickhouse-jdbc:0.6.0'
}
```

## Premier exemple

### Connexion simple

```java
import com.clickhouse.jdbc.ClickHouseConnection;
import com.clickhouse.jdbc.ClickHouseDataSource;

public class ClickHouseExample {
    public static void main(String[] args) {
        String url = "jdbc:clickhouse://localhost:8123/default";
        
        try (ClickHouseConnection conn = 
             new ClickHouseDataSource(url).getConnection()) {
            System.out.println("Connected to ClickHouse!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Structure de projet

```
clickhouse-java-project/
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── example/
│                   └── ClickHouseApp.java
├── pom.xml
└── README.md
```

---

**Prochaine étape :** [Driver JDBC et Connexion](./02-jdbc-connection/README.md)

