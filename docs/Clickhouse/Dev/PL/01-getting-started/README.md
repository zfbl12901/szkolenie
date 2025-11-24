# 1. Rozpoczęcie pracy z ClickHouse i Java

## 🎯 Cele

- Zrozumieć ClickHouse i Java
- Skonfigurować środowisko deweloperskie
- Zainstalować sterownik JDBC
- Utworzyć pierwszy projekt

## Konfiguracja Maven

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

## Pierwszy przykład

```java
import com.clickhouse.jdbc.ClickHouseConnection;
import com.clickhouse.jdbc.ClickHouseDataSource;

public class ClickHouseExample {
    public static void main(String[] args) {
        String url = "jdbc:clickhouse://localhost:8123/default";
        
        try (ClickHouseConnection conn = 
             new ClickHouseDataSource(url).getConnection()) {
            System.out.println("Połączono z ClickHouse!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

**Następny krok :** [Sterownik JDBC i połączenie](./02-jdbc-connection/README.md)

