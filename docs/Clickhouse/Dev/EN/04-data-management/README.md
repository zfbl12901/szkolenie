# 4. Data Management

## 🎯 Objectives

- Create tables from Java
- Manage schemas
- Import/export data
- Manage partitions

## Create Table

```java
String createTableSQL = """
    CREATE TABLE IF NOT EXISTS events (
        id UInt64,
        event_date Date,
        user_id UInt32,
        event_type String,
        value Float64
    )
    ENGINE = MergeTree()
    PARTITION BY toYYYYMM(event_date)
    ORDER BY (event_date, user_id)
    """;

try (Connection conn = ConnectionManager.getConnection();
     Statement stmt = conn.createStatement()) {
    stmt.execute(createTableSQL);
}
```

## Check Table Exists

```java
public boolean tableExists(String tableName) throws SQLException {
    String sql = "EXISTS TABLE " + tableName;
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        return rs.next() && rs.getInt(1) == 1;
    }
}
```

---

**Next step :** [Performance and Optimization](./05-performance/README.md)

