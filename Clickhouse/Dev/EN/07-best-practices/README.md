# 7. Best Practices

## 🎯 Objectives

- Structure code
- Use appropriate patterns
- Optimize resources
- Secure application

## Service Layer

```java
public class EventService {
    private final ConnectionManager connectionManager;
    
    public EventService(ConnectionManager connectionManager) {
        this.connectionManager = connectionManager;
    }
    
    public List<Event> getEvents(Date date) throws SQLException {
        // Business logic
    }
}
```

## Resource Management

```java
// ✅ Good
try (Connection conn = getConnection();
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery(sql)) {
    // ...
}
```

## Security

```java
// ✅ Good : PreparedStatement
String sql = "SELECT * FROM events WHERE id = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setLong(1, eventId);
```

---

**Next step :** [Practical Projects](./08-projets/README.md)

