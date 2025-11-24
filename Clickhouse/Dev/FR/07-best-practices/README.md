# 7. Bonnes Pratiques

## 🎯 Objectifs

- Structurer le code
- Utiliser des patterns appropriés
- Optimiser les ressources
- Sécuriser l'application

## Architecture

### Service Layer

```java
public class EventService {
    private final ConnectionManager connectionManager;
    
    public EventService(ConnectionManager connectionManager) {
        this.connectionManager = connectionManager;
    }
    
    public List<Event> getEvents(Date date) throws SQLException {
        // Logique métier
    }
    
    public void insertEvent(Event event) throws SQLException {
        // Validation et insertion
    }
}
```

### Repository Pattern

```java
public interface EventRepository {
    List<Event> findByDate(Date date) throws SQLException;
    void save(Event event) throws SQLException;
    void saveAll(List<Event> events) throws SQLException;
}

public class ClickHouseEventRepository implements EventRepository {
    // Implémentation
}
```

## Gestion des ressources

### Try-with-resources

```java
// ✅ Bon
try (Connection conn = getConnection();
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery(sql)) {
    // ...
}

// ❌ Moins bon
Connection conn = getConnection();
Statement stmt = conn.createStatement();
// Risque de fuite de ressources
```

## Configuration

### Externaliser la configuration

```java
public class ClickHouseConfig {
    private String host;
    private int port;
    private String database;
    private String user;
    private String password;
    
    // Charger depuis properties file
    public static ClickHouseConfig load() {
        Properties props = new Properties();
        try (InputStream is = ClickHouseConfig.class
                .getResourceAsStream("/clickhouse.properties")) {
            props.load(is);
        }
        // ...
    }
}
```

## Sécurité

### Préparer les requêtes

```java
// ✅ Bon : PreparedStatement
String sql = "SELECT * FROM events WHERE id = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setLong(1, eventId);

// ❌ Moins bon : Concatenation
String sql = "SELECT * FROM events WHERE id = " + eventId;
```

### Gestion des credentials

```java
// Utiliser des variables d'environnement ou fichiers sécurisés
String password = System.getenv("CLICKHOUSE_PASSWORD");
// Ou utiliser un gestionnaire de secrets
```

## Tests

### Tests unitaires

```java
@Test
public void testInsertEvent() throws SQLException {
    EventService service = new EventService(mockConnectionManager);
    Event event = new Event(1L, new Date(), 100, "click", 1.5);
    
    service.insertEvent(event);
    
    // Vérifications
}
```

---

**Prochaine étape :** [Projets Pratiques](./08-projets/README.md)

