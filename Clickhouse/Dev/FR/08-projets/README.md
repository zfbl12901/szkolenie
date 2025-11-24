# 8. Projets Pratiques

## 🎯 Objectifs

- Créer une application complète
- Intégrer ClickHouse dans un projet réel
- Appliquer les bonnes pratiques
- Optimiser les performances

## Projet 1 : Service d'Analytics

### Objectif

Créer un service Java pour analyser les événements web.

### Structure

```
analytics-service/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/
│       │       └── analytics/
│       │           ├── config/
│       │           ├── model/
│       │           ├── repository/
│       │           └── service/
│       └── resources/
│           └── application.properties
└── pom.xml
```

### Exemple de service

```java
@Service
public class AnalyticsService {
    private final EventRepository eventRepository;
    
    public AnalyticsService(EventRepository eventRepository) {
        this.eventRepository = eventRepository;
    }
    
    public Map<String, Long> getEventCountsByType(Date date) {
        return eventRepository.countByType(date);
    }
    
    public List<TopUser> getTopUsers(int limit) {
        return eventRepository.findTopUsers(limit);
    }
}
```

## Projet 2 : Application Spring Boot

### Dépendances

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.clickhouse</groupId>
    <artifactId>clickhouse-jdbc</artifactId>
</dependency>
```

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

### Repository

```java
@Repository
public class EventRepository {
    private final JdbcTemplate jdbcTemplate;
    
    public List<Event> findByDate(Date date) {
        String sql = "SELECT * FROM events WHERE event_date = ?";
        return jdbcTemplate.query(sql, new EventRowMapper(), date);
    }
}
```

## Projet 3 : API REST

### Controller

```java
@RestController
@RequestMapping("/api/events")
public class EventController {
    private final EventService eventService;
    
    @GetMapping("/stats")
    public ResponseEntity<EventStats> getStats(@RequestParam Date date) {
        EventStats stats = eventService.getStats(date);
        return ResponseEntity.ok(stats);
    }
    
    @PostMapping
    public ResponseEntity<Void> createEvent(@RequestBody Event event) {
        eventService.insertEvent(event);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

## Conseils

1. **Structure claire** : Séparer les couches (Service, Repository, Model)
2. **Configuration externe** : Utiliser properties files
3. **Tests** : Écrire des tests unitaires et d'intégration
4. **Documentation** : Documenter l'API et le code
5. **Monitoring** : Ajouter des logs et métriques

---

**Félicitations ! Vous avez terminé la formation ClickHouse pour Développeur Java ! 🎉**

