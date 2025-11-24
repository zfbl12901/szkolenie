# 5. Performance et Optimisation

## 🎯 Objectifs

- Optimiser les requêtes
- Utiliser le batch processing
- Gérer la mémoire
- Monitorer les performances

## Batch Processing

### INSERT batch optimisé

```java
public void batchInsert(List<Event> events) throws SQLException {
    String sql = "INSERT INTO events (id, event_date, user_id, event_type, value) VALUES";
    
    try (Connection conn = ConnectionManager.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql)) {
        
        conn.setAutoCommit(false);
        
        int batchSize = 1000;
        int count = 0;
        
        for (Event event : events) {
            pstmt.setLong(1, event.getId());
            pstmt.setDate(2, new Date(event.getEventDate().getTime()));
            pstmt.setInt(3, event.getUserId());
            pstmt.setString(4, event.getEventType());
            pstmt.setDouble(5, event.getValue());
            pstmt.addBatch();
            
            if (++count % batchSize == 0) {
                pstmt.executeBatch();
                conn.commit();
            }
        }
        
        pstmt.executeBatch();
        conn.commit();
        conn.setAutoCommit(true);
    }
}
```

## Requêtes optimisées

### Utiliser LIMIT

```java
String sql = "SELECT * FROM events WHERE event_date = ? LIMIT ?";

try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setDate(1, date);
    pstmt.setInt(2, 1000); // Limiter les résultats
    // ...
}
```

### Éviter SELECT *

```java
// ✅ Bon
String sql = "SELECT id, event_type, value FROM events";

// ❌ Moins bon
String sql = "SELECT * FROM events";
```

## Monitoring

### Mesurer le temps d'exécution

```java
public long executeQueryWithTiming(String sql) throws SQLException {
    long startTime = System.currentTimeMillis();
    
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        
        // Traiter les résultats
        while (rs.next()) {
            // ...
        }
    }
    
    long endTime = System.currentTimeMillis();
    return endTime - startTime;
}
```

### Requêtes lentes

```java
public void findSlowQueries() throws SQLException {
    String sql = """
        SELECT query, query_duration_ms, read_rows
        FROM system.query_log
        WHERE type = 'QueryFinish'
        ORDER BY query_duration_ms DESC
        LIMIT 10
        """;
    
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        
        while (rs.next()) {
            System.out.println("Query: " + rs.getString("query"));
            System.out.println("Duration: " + rs.getLong("query_duration_ms") + "ms");
        }
    }
}
```

## Connection Pooling

### Configuration optimale

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:clickhouse://localhost:8123/default");
config.setMaximumPoolSize(20); // Ajuster selon charge
config.setMinimumIdle(5);
config.setConnectionTimeout(30000);
config.setIdleTimeout(600000);
config.setMaxLifetime(1800000);
```

---

**Prochaine étape :** [Gestion des Erreurs](./06-error-handling/README.md)

