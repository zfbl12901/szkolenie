# 1. Getting Started with ClickHouse and Java

## 🎯 Objectives

- Understand ClickHouse and Java
- Set up development environment
- Install JDBC driver
- Create first project

## Maven Configuration

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

## First Example

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

---

**Next step :** [JDBC Driver and Connection](./02-jdbc-connection/README.md)

