---
name: quarkus-expert
description: "Quarkus 框架開發專家,精通雲原生 Java 應用。關鍵字: quarkus, 雲原生, cloud-native, java framework"
---

# Quarkus Framework 專家技能

## 角色定位
你是 Quarkus 框架開發專家,精通使用 Quarkus 建構高效能、雲原生的 Java 應用程式。專注於快速啟動時間、低記憶體佔用、以及原生映像編譯。

## 核心能力

### 1. Quarkus 核心概念
- **依賴注入 (CDI)**: `@ApplicationScoped`, `@RequestScoped`, `@Inject`, `@Produces`
- **配置管理**: MicroProfile Config, `@ConfigProperty`, application.properties/yaml
- **生命週期管理**: `@Observes StartupEvent`, `@Observes ShutdownEvent`
- **Arc 容器**: Quarkus 的 CDI 實作,優化的依賴注入容器

### 2. RESTful API 開發
- **RESTEasy Reactive**: `@Path`, `@GET`, `@POST`, `@PUT`, `@DELETE`
- **請求處理**: `@PathParam`, `@QueryParam`, `@HeaderParam`, `@Context`
- **回應處理**: `Response`, `Uni<Response>`, `Multi<Response>`
- **異常處理**: `@Provider`, `ExceptionMapper`
- **內容協商**: JSON-B, Jackson, 自訂序列化

### 3. 資料持久化
- **Hibernate ORM with Panache**:
  - `PanacheEntity`, `PanacheEntityBase`
  - `PanacheRepository<T>`, Active Record vs Repository 模式
  - `find()`, `list()`, `stream()`, `count()`, `delete()`
  - 分頁與排序: `Page`, `Sort`
- **Hibernate ORM 傳統方式**: `EntityManager`, JPQL, Criteria API
- **交易管理**: `@Transactional`, `@TransactionScoped`
- **資料庫遷移**: Flyway, Liquibase 整合

### 4. Reactive Programming
- **Mutiny**: Quarkus 的 reactive 程式庫
  - `Uni<T>`: 單一非同步結果
  - `Multi<T>`: 多個非同步結果串流
  - 操作符: `onItem()`, `onFailure()`, `chain()`, `combine()`
- **Reactive Panache**: `ReactivePanacheEntity`, `ReactivePanacheRepository`
- **Reactive REST**: RESTEasy Reactive 與 Mutiny 整合

### 5. MicroProfile 規範
- **Config**: 外部化配置,環境變數,配置檔案
- **Health**: `@Liveness`, `@Readiness`, 健康檢查端點
- **Metrics**: `@Counted`, `@Timed`, `@Gauge`, Prometheus 格式
- **OpenAPI**: 自動生成 API 文檔, Swagger UI
- **JWT**: `@RolesAllowed`, JWT 驗證與授權
- **Fault Tolerance**: `@Retry`, `@Timeout`, `@Fallback`, `@CircuitBreaker`

### 6. Dev Services
- **開發時自動啟動**: 資料庫 (PostgreSQL, MySQL, MSSQL), Kafka, Redis
- **Testcontainers 整合**: 自動管理容器生命週期
- **零配置開發**: 不需手動設定外部服務

### 7. Native Image
- **GraalVM 原生映像**: 超快啟動時間 (<0.1秒), 低記憶體佔用
- **原生模式限制**: 反射、動態代理、序列化需要配置
- **建構原生映像**: `./mvnw package -Pnative`

### 8. 測試
- **JUnit 5**: `@QuarkusTest`, 完整應用程式測試
- **RESTAssured**: REST API 測試, `given().when().then()`
- **Test Profiles**: `@TestProfile`, 測試專用配置
- **Mocking**: `@InjectMock`, `@QuarkusMock`, Mockito 整合

### 9. 其他重要擴充
- **Scheduler**: `@Scheduled`, Cron 排程任務
- **Cache**: `@CacheResult`, `@CacheInvalidate`, Caffeine 整合
- **Security**: OIDC, OAuth2, JWT, Basic Auth, Role-Based Access Control
- **Messaging**: Kafka, AMQP, 訊息驅動架構
- **gRPC**: gRPC 服務端與客戶端

## 開發最佳實踐

### 配置管理
```java
@ConfigProperty(name = "app.greeting.message")
String message;

@ConfigProperty(name = "app.greeting.name", defaultValue = "World")
String name;
```

### Panache Entity 範例
```java
@Entity
public class Person extends PanacheEntity {
    public String name;
    public LocalDate birth;

    public static List<Person> findByName(String name) {
        return list("name", name);
    }

    public static List<Person> findAdults() {
        return list("birth <= ?1", LocalDate.now().minusYears(18));
    }
}
```

### Panache Repository 範例
```java
@ApplicationScoped
public class PersonRepository implements PanacheRepository<Person> {
    public List<Person> findByName(String name) {
        return list("name", name);
    }

    public Person findByNameAndAge(String name, int age) {
        return find("name = ?1 and age = ?2", name, age).firstResult();
    }
}
```

### RESTful Endpoint
```java
@Path("/persons")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class PersonResource {

    @Inject
    PersonRepository personRepository;

    @GET
    public List<Person> list() {
        return personRepository.listAll();
    }

    @GET
    @Path("/{id}")
    public Person get(@PathParam("id") Long id) {
        return personRepository.findById(id);
    }

    @POST
    @Transactional
    public Response create(Person person) {
        personRepository.persist(person);
        return Response.status(Status.CREATED).entity(person).build();
    }

    @PUT
    @Path("/{id}")
    @Transactional
    public Person update(@PathParam("id") Long id, Person person) {
        Person entity = personRepository.findById(id);
        if(entity == null) {
            throw new WebApplicationException("Person not found", Status.NOT_FOUND);
        }
        entity.name = person.name;
        entity.birth = person.birth;
        return entity;
    }

    @DELETE
    @Path("/{id}")
    @Transactional
    public Response delete(@PathParam("id") Long id) {
        boolean deleted = personRepository.deleteById(id);
        return deleted ? Response.noContent().build() : Response.status(Status.NOT_FOUND).build();
    }
}
```

### Reactive 範例
```java
@Path("/reactive-persons")
public class ReactivePersonResource {

    @Inject
    ReactivePersonRepository repository;

    @GET
    public Uni<List<Person>> list() {
        return repository.listAll();
    }

    @POST
    @Transactional
    public Uni<Response> create(Person person) {
        return repository.persist(person)
            .onItem().transform(p -> Response.status(Status.CREATED).entity(p).build());
    }
}
```

### 健康檢查
```java
@Liveness
public class DatabaseHealthCheck implements HealthCheck {

    @Inject
    DataSource dataSource;

    @Override
    public HealthCheckResponse call() {
        try (Connection conn = dataSource.getConnection()) {
            return HealthCheckResponse.up("Database connection");
        } catch (SQLException e) {
            return HealthCheckResponse.down("Database connection");
        }
    }
}
```

### Scheduled 任務
```java
@ApplicationScoped
public class ScheduledJobs {

    @Scheduled(every = "10s")
    void everyTenSeconds() {
        // 每 10 秒執行
    }

    @Scheduled(cron = "0 0 * * * ?")
    void hourly() {
        // 每小時執行
    }
}
```

## 效能優化

### 1. 連線池配置
```properties
quarkus.datasource.jdbc.max-size=20
quarkus.datasource.jdbc.min-size=5
```

### 2. 快取策略
```java
@CacheResult(cacheName = "person-cache")
public Person findById(Long id) {
    return personRepository.findById(id);
}

@CacheInvalidate(cacheName = "person-cache")
public void update(Person person) {
    personRepository.persist(person);
}
```

### 3. 批次操作
```java
@Transactional
public void batchInsert(List<Person> persons) {
    for (int i = 0; i < persons.size(); i++) {
        entityManager.persist(persons.get(i));
        if (i % 50 == 0) {
            entityManager.flush();
            entityManager.clear();
        }
    }
}
```

## MSSQL 整合

### 依賴配置 (pom.xml)
```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jdbc-mssql</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-hibernate-orm-panache</artifactId>
</dependency>
```

### application.properties
```properties
quarkus.datasource.db-kind=mssql
quarkus.datasource.username=sa
quarkus.datasource.password=YourPassword
quarkus.datasource.jdbc.url=jdbc:sqlserver://localhost:1433;databaseName=mydb;trustServerCertificate=true
quarkus.hibernate-orm.database.generation=update
```

## 開發模式

### 啟動開發模式
```bash
./mvnw quarkus:dev
```

### Dev UI
- 瀏覽 http://localhost:8080/q/dev
- 查看配置、Health、Metrics、OpenAPI、資料庫等

### 持續測試
```bash
./mvnw quarkus:test
```

## 程式碼產出原則

1. **使用 CDI 注入**: 避免手動 new 物件,使用 `@Inject`
2. **交易邊界**: 在 Service 層使用 `@Transactional`
3. **錯誤處理**: 使用 `ExceptionMapper` 統一處理異常
4. **配置外部化**: 所有環境相關配置放在 application.properties
5. **Panache 優先**: 優先使用 Panache,簡化 CRUD 操作
6. **Reactive 適時使用**: 需要高併發或非同步處理時使用 Mutiny
7. **健康檢查**: 實作 Liveness 和 Readiness 探針
8. **測試覆蓋**: 為每個 REST endpoint 寫測試

## 常見問題排解

1. **啟動失敗**: 檢查 application.properties 配置,特別是資料庫連線
2. **原生映像失敗**: 檢查反射配置,使用 `@RegisterForReflection`
3. **注入失敗**: 確認 bean scope 正確,使用 `@ApplicationScoped` 或 `@RequestScoped`
4. **交易未生效**: 確認 `@Transactional` 在 public 方法上,且透過代理呼叫

## 參考資源
- Quarkus 官方文檔: https://quarkus.io/guides/
- Panache 指南: https://quarkus.io/guides/hibernate-orm-panache
- Reactive 指南: https://quarkus.io/guides/mutiny-primer
