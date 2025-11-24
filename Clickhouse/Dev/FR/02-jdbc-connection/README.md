# 2. Driver JDBC et Connexion

## 🎯 Objectifs

- Configurer la connexion JDBC
- Gérer le pooling de connexions
- Utiliser différents types de connexion
- Gérer les paramètres de connexion

## URL de connexion

### Format de base

```java
String url = "jdbc:clickhouse://localhost:8123/default";
```

### Avec authentification

```java
String url = "jdbc:clickhouse://localhost:8123/default?user=default&password=password";
```

### Avec paramètres

```java
String url = "jdbc:clickhouse://localhost:8123/default?" +
             "socket_timeout=300000&" +
             "connect_timeout=10000";
```

## Connexion simple

```java
import com.clickhouse.jdbc.ClickHouseConnection;
import com.clickhouse.jdbc.ClickHouseDataSource;
import java.sql.Statement;

public class ConnectionExample {
    public static void main(String[] args) {
        String url = "jdbc:clickhouse://localhost:8123/default";
        
        try (ClickHouseConnection conn = 
             new ClickHouseDataSource(url).getConnection();
             Statement stmt = conn.createStatement()) {
            
            // Test de connexion
            stmt.execute("SELECT 1");
            System.out.println("Connection successful!");
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Pooling de connexions

### Avec HikariCP

```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.0.1</version>
</dependency>
```

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:clickhouse://localhost:8123/default");
config.setMaximumPoolSize(10);
config.setMinimumIdle(2);

HikariDataSource dataSource = new HikariDataSource(config);
```

## Gestion des connexions

```java
public class ConnectionManager {
    private static HikariDataSource dataSource;
    
    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:clickhouse://localhost:8123/default");
        config.setMaximumPoolSize(10);
        dataSource = new HikariDataSource(config);
    }
    
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
    
    public static void close() {
        if (dataSource != null) {
            dataSource.close();
        }
    }
}
```

---

**Prochaine étape :** [Requêtes SQL depuis Java](./03-queries/README.md)

