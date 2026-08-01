Развернуто про Testcontainers, аннотации и жизненный цикл:

---

## 1. `@Testcontainers` (на классе)

- Подключает JUnit 5 extension: `TestcontainersExtension`
- Сканирует поля с `@Container` в тестовом классе
- Без этой аннотации `@Container` игнорируются

```java
@Testcontainers
class MyTest {
    // ...
}
```

---

## 2. `@Container` (на поле)

### static поле (Shared Container)
```java
@Container
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
```
- Контейнер **стартует один раз** перед всеми тестами
- Останавливается **после всех тестов** (в `@AfterAll`)
- **Плюс**: быстрый для больших наборов тестов
- **Минус**: состояние БД сохраняется между тестами

### instance поле (Per-Test Container)
```java
@Container
private PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
```
- Контейнер **стартует для каждого теста**
- Останавливается **после каждого теста** (в `@AfterEach`)
- **Плюс**: чистое состояние для каждого теста
- **Минус**: медленнее (старт/стоп контейнера)

---

## 3. Жизненный цикл (подробно)

### Для static полей:
```
@BeforeAll (JUnit)
    ↓
Контейнер.start() → поднимается контейнер
    ↓
@BeforeEach (первый тест) → тест → @AfterEach
    ↓
@BeforeEach (второй тест) → тест → @AfterEach
    ↓
...
    ↓
@AfterAll
    ↓
Контейнер.stop() → контейнер останавливается
```

### Для instance полей:
```
@BeforeEach
    ↓
Контейнер.start() → поднимается контейнер
    ↓
Тест
    ↓
@AfterEach
    ↓
Контейнер.stop() → контейнер останавливается
```

---

## 4. `@DynamicPropertySource` (метод)

Используется **только** в Spring Boot тестах.

```java
@DynamicPropertySource
static void setProperties(DynamicPropertyRegistry registry) {
    // Добавляем динамические свойства
    registry.add("spring.datasource.url", postgres::getJdbcUrl);
    registry.add("spring.datasource.username", postgres::getUsername);
}
```

### Важные нюансы:

1. **Метод должен быть static** (для JUnit 5)
2. **Не работает с `@DataJpaTest`** без доп. настроек (см. ниже)
3. Добавляет свойства **перед** созданием Spring Context
4. Можно добавлять **условно**:
```java
@DynamicPropertySource
static void setProperties(DynamicPropertyRegistry registry) {
    if (System.getProperty("env") == null) {
        registry.add("app.port", () -> container.getMappedPort(8080));
    }
}
```

---

## 5. Специфичные аннотации для Testcontainers

### `@Testcontainers(disabledWithoutDocker = true)`
- Пропускает тесты, если Docker не установлен
```java
@Testcontainers(disabledWithoutDocker = true)
class MyTest { ... }
```

### `@Testcontainers(parallel = true)` (экспериментальное)
- Разрешает параллельный запуск контейнеров
- Требует настройки в `testcontainers.properties`

---

## 6. Важные детали с Spring Boot

### `@DataJpaTest` + Testcontainers
```java
@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // ← ключевая аннотация
class UserRepositoryTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
    
    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        // ...
    }
}
```
**Без `@AutoConfigureTestDatabase`** Spring подменит БД на H2 (Embedded), игнорируя Testcontainers.

### `@SpringBootTest` + Testcontainers
```java
@SpringBootTest
@Testcontainers
class MyIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
    
    // Можно инжектить контейнер напрямую через @Autowired
    @Autowired
    private DataSource dataSource;
    
    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
    }
}
```

---

## 7. JDBC URL shortcut (без @Container)

Можно **вообще не объявлять** `@Container`, если использовать специальный URL:

```java
// application-test.properties
spring.datasource.url=jdbc:tc:postgresql:16:///testdb
spring.datasource.driver-class-name=org.testcontainers.jdbc.ContainerDatabaseDriver
```

Testcontainers автоматически:
- Поднимет контейнер
- Подключится к нему
- Уничтожит после тестов

**Минус**: нет доступа к контейнеру (нельзя добавить init-скрипты, проверить логи).

---

## 8. `@TestInstance` и его влияние

```java
@Testcontainers
@TestInstance(TestInstance.Lifecycle.PER_CLASS) // ← контейнер будет один на класс
class MyTest {
    
    @Container
    private PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
    // Т.к. PER_CLASS, контейнер стартует один раз для всего класса, 
    // даже если поле не static
}
```

---

## 9. Проверка готовности контейнера

### `waitingFor()` — дожидаемся готовности сервиса
```java
@Container
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
    .waitingFor(Wait.forListeningPort())  // ждем, пока порт слушается
    .waitingFor(Wait.forLogMessage(".*ready to accept connections.*", 1)); // ждем лог

// Для HTTP контейнеров
.waitingFor(Wait.forHttp("/health").forStatusCode(200))
```

### `withStartupTimeout()`
```java
.withStartupTimeout(Duration.ofSeconds(60)) // таймаут старта
```

---

## 10. `@BeforeAll`/`@BeforeEach` + контейнеры

```java
@Testcontainers
class MyTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
    
    @BeforeAll
    static void init() {
        // Контейнер УЖЕ запущен (стартует до @BeforeAll)
        System.out.println("JDBC URL: " + postgres.getJdbcUrl());
    }
    
    @BeforeEach
    void clean() {
        // Можно чистить БД после каждого теста
        jdbcTemplate.execute("TRUNCATE TABLE users");
    }
}
```

---

## 11. Работа с несколькими контейнерами

```java
@Testcontainers
class MultiContainerTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:latest"));
    
    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", () -> redis.getMappedPort(6379));
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }
}
```

---

## 12. Кастомные контейнеры (Compose)

```java
@Container
static DockerComposeContainer<?> compose = new DockerComposeContainer<>(
    new File("docker-compose.yml")
)
    .withExposedService("postgres_1", 5432); // Получить порт сервиса
```

---

## 13. Reusable containers (для CI/CD)

В `~/.testcontainers.properties`:
```properties
testcontainers.reuse.enable=true
```

В коде:
```java
@Container
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
    .withReuse(true); // Контейнер НЕ удаляется после тестов, переиспользуется
```

---

## 14. Ошибки и их решение

### "No Docker environment found"
```java
@Testcontainers(disabledWithoutDocker = true) // ← пропускаем тесты
```

### "Port already in use"
```java
.withExposedPorts(5432) // не указывайте фиксированный порт, используйте random
// Вместо .withExposedPorts(5432) и фиксации порта через .withPortBindings()
```

### "Could not start container" (не хватает памяти)
```java
.withCreateContainerCmdModifier(cmd -> cmd.withHostConfig(
    new HostConfig().withMemory(512 * 1024 * 1024L) // 512 MB
));
```

### "Connection refused" — контейнер не готов
```java
.waitingFor(Wait.forLogMessage(".*started.*", 1)) // Ждем сообщение в логе
```2