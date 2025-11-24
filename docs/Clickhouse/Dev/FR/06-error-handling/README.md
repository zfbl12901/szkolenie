# 6. Gestion des Erreurs

## 🎯 Objectifs

- Gérer les exceptions SQL
- Implémenter la retry logic
- Logger les erreurs
- Valider les données

## Gestion des exceptions

### Try-catch basique

```java
public void executeQuery(String sql) {
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        
        // Traiter les résultats
    } catch (SQLException e) {
        System.err.println("SQL Error: " + e.getMessage());
        System.err.println("SQL State: " + e.getSQLState());
        System.err.println("Error Code: " + e.getErrorCode());
        e.printStackTrace();
    }
}
```

## Retry Logic

### Implémentation simple

```java
public <T> T executeWithRetry(Supplier<T> operation, int maxRetries) {
    int attempts = 0;
    
    while (attempts < maxRetries) {
        try {
            return operation.get();
        } catch (SQLException e) {
            attempts++;
            if (attempts >= maxRetries) {
                throw new RuntimeException("Failed after " + maxRetries + " attempts", e);
            }
            
            try {
                Thread.sleep(1000 * attempts); // Backoff exponentiel
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt();
                throw new RuntimeException(ie);
            }
        }
    }
    
    throw new RuntimeException("Failed to execute operation");
}
```

### Utilisation

```java
List<Event> events = executeWithRetry(() -> {
    return getEventsFromDatabase();
}, 3);
```

## Logging

### Avec SLF4J

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ClickHouseService {
    private static final Logger logger = LoggerFactory.getLogger(ClickHouseService.class);
    
    public void insertEvent(Event event) {
        try {
            // Insertion
            logger.info("Event inserted: {}", event.getId());
        } catch (SQLException e) {
            logger.error("Failed to insert event: {}", event.getId(), e);
            throw new RuntimeException(e);
        }
    }
}
```

## Validation des données

### Valider avant insertion

```java
public void insertEvent(Event event) throws ValidationException {
    if (event.getId() <= 0) {
        throw new ValidationException("Invalid event ID");
    }
    
    if (event.getEventDate() == null) {
        throw new ValidationException("Event date is required");
    }
    
    if (event.getEventType() == null || event.getEventType().isEmpty()) {
        throw new ValidationException("Event type is required");
    }
    
    // Insérer après validation
    // ...
}
```

## Gestion des transactions

### Rollback en cas d'erreur

```java
public void insertMultipleEvents(List<Event> events) throws SQLException {
    try (Connection conn = ConnectionManager.getConnection()) {
        conn.setAutoCommit(false);
        
        try (PreparedStatement pstmt = conn.prepareStatement(insertSQL)) {
            for (Event event : events) {
                // Préparer l'insertion
                pstmt.addBatch();
            }
            pstmt.executeBatch();
            conn.commit();
        } catch (SQLException e) {
            conn.rollback();
            throw e;
        } finally {
            conn.setAutoCommit(true);
        }
    }
}
```

---

**Prochaine étape :** [Bonnes Pratiques](./07-best-practices/README.md)

