# 3. Requêtes SQL depuis Java

## 🎯 Objectifs

- Exécuter des requêtes SELECT
- Utiliser PreparedStatement
- Gérer les résultats
- Exécuter des INSERT/UPDATE/DELETE

## Requêtes SELECT

### Requête simple

```java
try (Connection conn = ConnectionManager.getConnection();
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery("SELECT * FROM events LIMIT 10")) {
    
    while (rs.next()) {
        long id = rs.getLong("id");
        String eventType = rs.getString("event_type");
        double value = rs.getDouble("value");
        System.out.println(id + " - " + eventType + " - " + value);
    }
}
```

## PreparedStatement

### Requête paramétrée

```java
String sql = "SELECT * FROM events WHERE event_date = ? AND event_type = ?";

try (Connection conn = ConnectionManager.getConnection();
     PreparedStatement pstmt = conn.prepareStatement(sql)) {
    
    pstmt.setDate(1, Date.valueOf("2024-01-15"));
    pstmt.setString(2, "click");
    
    try (ResultSet rs = pstmt.executeQuery()) {
        while (rs.next()) {
            // Traiter les résultats
        }
    }
}
```

## INSERT de données

### INSERT simple

```java
String sql = "INSERT INTO events (id, event_date, event_time, user_id, event_type, value) VALUES (?, ?, ?, ?, ?, ?)";

try (Connection conn = ConnectionManager.getConnection();
     PreparedStatement pstmt = conn.prepareStatement(sql)) {
    
    pstmt.setLong(1, 1L);
    pstmt.setDate(2, Date.valueOf("2024-01-15"));
    pstmt.setTimestamp(3, Timestamp.valueOf("2024-01-15 10:00:00"));
    pstmt.setInt(4, 100);
    pstmt.setString(5, "click");
    pstmt.setDouble(6, 1.5);
    
    pstmt.executeUpdate();
}
```

### INSERT batch

```java
String sql = "INSERT INTO events (id, event_date, user_id, event_type, value) VALUES (?, ?, ?, ?, ?)";

try (Connection conn = ConnectionManager.getConnection();
     PreparedStatement pstmt = conn.prepareStatement(sql)) {
    
    for (int i = 0; i < 1000; i++) {
        pstmt.setLong(1, i);
        pstmt.setDate(2, Date.valueOf("2024-01-15"));
        pstmt.setInt(3, 100 + i);
        pstmt.setString(4, "event_" + i);
        pstmt.setDouble(5, Math.random() * 100);
        pstmt.addBatch();
    }
    
    pstmt.executeBatch();
}
```

## Gestion des résultats

### Mapper vers objets

```java
public class Event {
    private long id;
    private Date eventDate;
    private int userId;
    private String eventType;
    private double value;
    
    // Constructeurs, getters, setters...
}

public List<Event> getEvents() throws SQLException {
    List<Event> events = new ArrayList<>();
    String sql = "SELECT * FROM events LIMIT 100";
    
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        
        while (rs.next()) {
            Event event = new Event();
            event.setId(rs.getLong("id"));
            event.setEventDate(rs.getDate("event_date"));
            event.setUserId(rs.getInt("user_id"));
            event.setEventType(rs.getString("event_type"));
            event.setValue(rs.getDouble("value"));
            events.add(event);
        }
    }
    
    return events;
}
```

---

**Prochaine étape :** [Gestion des Données](./04-data-management/README.md)

