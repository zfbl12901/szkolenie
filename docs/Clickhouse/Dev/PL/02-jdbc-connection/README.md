# 2. Sterownik JDBC i połączenie

## 🎯 Cele

- Skonfigurować połączenie JDBC
- Zarządzać pulą połączeń
- Używać różnych typów połączeń

## URL połączenia

```java
String url = "jdbc:clickhouse://localhost:8123/default";
String url = "jdbc:clickhouse://localhost:8123/default?user=default&password=password";
```

## Pula połączeń

### Z HikariCP

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

**Następny krok :** [Zapytania SQL z Java](./03-queries/README.md)

