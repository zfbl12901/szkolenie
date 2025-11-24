# 8. Practical Projects

## 🎯 Objectives

- Create complete application
- Integrate ClickHouse in real project
- Apply best practices
- Optimize performance

## Project 1 : Analytics Service

### Structure

```
analytics-service/
├── src/main/java/com/analytics/
│   ├── config/
│   ├── model/
│   ├── repository/
│   └── service/
└── pom.xml
```

### Service Example

```java
@Service
public class AnalyticsService {
    private final EventRepository eventRepository;
    
    public Map<String, Long> getEventCountsByType(Date date) {
        return eventRepository.countByType(date);
    }
}
```

## Project 2 : Spring Boot Application

### Configuration

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

## Project 3 : REST API

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

**Congratulations! You have completed the ClickHouse training for Java Developer! 🎉**

