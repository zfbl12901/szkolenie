# 8. Projekty praktyczne

## 🎯 Cele

- Tworzyć kompletną aplikację
- Integrować ClickHouse w rzeczywistym projekcie
- Stosować najlepsze praktyki
- Optymalizować wydajność

## Projekt 1 : Serwis analityczny

### Struktura

```
analytics-service/
├── src/main/java/com/analytics/
│   ├── config/
│   ├── model/
│   ├── repository/
│   └── service/
└── pom.xml
```

### Przykład serwisu

```java
@Service
public class AnalyticsService {
    private final EventRepository eventRepository;
    
    public Map<String, Long> getEventCountsByType(Date date) {
        return eventRepository.countByType(date);
    }
}
```

## Projekt 2 : Aplikacja Spring Boot

### Konfiguracja

```java
@Configuration
public class ClickHouseConfig {
    @Bean
    public DataSource clickHouseDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:clickhouse://localhost:8123/default");
        return new HikariDataSource(config);
    }
}
```

## Projekt 3 : API REST

```java
@RestController
@RequestMapping("/api/events")
public class EventController {
    @GetMapping("/stats")
    public ResponseEntity<EventStats> getStats(@RequestParam Date date) {
        EventStats stats = eventService.getStats(date);
        return ResponseEntity.ok(stats);
    }
}
```

---

**Gratulacje! Ukończyłeś szkolenie ClickHouse dla Dewelopera Java! 🎉**

