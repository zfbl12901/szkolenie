# 6. Error Handling

## 🎯 Objectives

- Handle SQL exceptions
- Implement retry logic
- Log errors
- Validate data

## Exception Handling

```java
public void executeQuery(String sql) {
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        // Process results
    } catch (SQLException e) {
        System.err.println("SQL Error: " + e.getMessage());
        e.printStackTrace();
    }
}
```

## Retry Logic

```java
public <T> T executeWithRetry(Supplier<T> operation, int maxRetries) {
    int attempts = 0;
    while (attempts < maxRetries) {
        try {
            return operation.get();
        } catch (SQLException e) {
            attempts++;
            if (attempts >= maxRetries) throw new RuntimeException(e);
            try { Thread.sleep(1000 * attempts); } 
            catch (InterruptedException ie) { throw new RuntimeException(ie); }
        }
    }
    throw new RuntimeException("Failed to execute");
}
```

---

**Next step :** [Best Practices](./07-best-practices/README.md)

