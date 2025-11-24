# 2. JDBC Driver and Connection

## 🎯 Objectives

- Configure JDBC connection
- Manage connection pooling
- Use different connection types
- Manage connection parameters

## Connection URL

```java
String url = "jdbc:clickhouse://localhost:8123/default";
String url = "jdbc:clickhouse://localhost:8123/default?user=default&password=password";
```

## Connection Pooling

### With HikariCP

```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.0.1</version>
</dependency>
```

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:clickhouse://localhost:8123/default");
config.setMaximumPoolSize(10);
HikariDataSource dataSource = new HikariDataSource(config);
```

---

**Next step :** [SQL Queries from Java](./03-queries/README.md)

