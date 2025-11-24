# 6. Obsługa błędów

## 🎯 Cele

- Obsługiwać wyjątki SQL
- Implementować logikę ponawiania
- Logować błędy
- Walidować dane

## Obsługa wyjątków

```java
public void executeQuery(String sql) {
    try (Connection conn = ConnectionManager.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        // Przetwarzaj wyniki
    } catch (SQLException e) {
        System.err.println("Błąd SQL: " + e.getMessage());
        e.printStackTrace();
    }
}
```

## Logika ponawiania

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
    throw new RuntimeException("Nie udało się wykonać");
}
```

---

**Następny krok :** [Najlepsze praktyki](./07-best-practices/README.md)

