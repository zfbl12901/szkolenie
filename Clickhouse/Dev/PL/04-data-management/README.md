# 4. Zarządzanie danymi

## 🎯 Cele

- Tworzyć tabele z Java
- Zarządzać schematami
- Importować/eksportować dane
- Zarządzać partycjami

## Utworzenie tabeli

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

## Sprawdzenie istnienia tabeli

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

**Następny krok :** [Wydajność i optymalizacja](./05-performance/README.md)

