# 3. SQL Queries from Java

## 🎯 Objectives

- Execute SELECT queries
- Use PreparedStatement
- Handle results
- Execute INSERT/UPDATE/DELETE

## SELECT Query

```java
try (Connection conn = ConnectionManager.getConnection();
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery("SELECT * FROM events LIMIT 10")) {
    
    while (rs.next()) {
        long id = rs.getLong("id");
        String eventType = rs.getString("event_type");
        System.out.println(id + " - " + eventType);
    }
}
```

## PreparedStatement

```java
String sql = "SELECT * FROM events WHERE event_date = ?";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setDate(1, Date.valueOf("2024-01-15"));
    try (ResultSet rs = pstmt.executeQuery()) {
        // Process results
    }
}
```

## INSERT

```java
String sql = "INSERT INTO events (id, event_date, user_id, event_type, value) VALUES (?, ?, ?, ?, ?)";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setLong(1, 1L);
    pstmt.setDate(2, Date.valueOf("2024-01-15"));
    pstmt.setInt(3, 100);
    pstmt.setString(4, "click");
    pstmt.setDouble(5, 1.5);
    pstmt.executeUpdate();
}
```

---

**Next step :** [Data Management](./04-data-management/README.md)

