# 7. Najlepsze praktyki

## 🎯 Cele

- Strukturyzować kod
- Używać odpowiednich wzorców
- Optymalizować zasoby
- Zabezpieczać aplikację

## Warstwa serwisowa

```java
public class EventService {
    private final ConnectionManager connectionManager;
    
    public EventService(ConnectionManager connectionManager) {
        this.connectionManager = connectionManager;
    }
    
    public List<Event> getEvents(Date date) throws SQLException {
        // Logika biznesowa
    }
}
```

## Zarządzanie zasobami

```java
// ✅ Dobrze
try (Connection conn = getConnection();
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery(sql)) {
    // ...
}
```

## Bezpieczeństwo

```java
// ✅ Dobrze : PreparedStatement
String sql = "SELECT * FROM events WHERE id = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setLong(1, eventId);
```

---

**Następny krok :** [Projekty praktyczne](./08-projets/README.md)

