# 5. Performance and Optimization

## 🎯 Objectives

- Optimize queries
- Use batch processing
- Manage memory
- Monitor performance

## Batch Processing

```java
public void batchInsert(List<Event> events) throws SQLException {
    String sql = "INSERT INTO events (id, event_date, user_id, event_type, value) VALUES";
    
    try (Connection conn = ConnectionManager.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql)) {
        
        conn.setAutoCommit(false);
        int batchSize = 1000;
        
        for (Event event : events) {
            pstmt.setLong(1, event.getId());
            pstmt.setDate(2, new Date(event.getEventDate().getTime()));
            pstmt.setInt(3, event.getUserId());
            pstmt.setString(4, event.getEventType());
            pstmt.setDouble(5, event.getValue());
            pstmt.addBatch();
            
            if (count % batchSize == 0) {
                pstmt.executeBatch();
                conn.commit();
            }
        }
        
        pstmt.executeBatch();
        conn.commit();
    }
}
```

---

**Next step :** [Error Handling](./06-error-handling/README.md)

