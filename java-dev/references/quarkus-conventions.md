# Quarkus Conventions Reference

> Activate this reference when the project uses Quarkus (detected via `io.quarkus` in `pom.xml` or `build.gradle`).
> Documentation: https://quarkus.io/guides/

## Core Principles

- **CDI-first**: Use `@ApplicationScoped`, `@RequestScoped`, `@Inject` for dependency injection
- **Build-time optimization**: Quarkus does most work at build time; follow its conventions for optimal startup
- **Dev Services**: Leverage automatic dev services for databases, Kafka, etc. in dev/test mode
- **Continuous Testing**: Use `quarkus:dev` with continuous testing enabled

## REST Layer

Use **Quarkus REST** (formerly RESTEasy Reactive), not the legacy RESTEasy Classic.

> Guide: https://quarkus.io/guides/rest

```java
@Path("/products")
@ApplicationScoped
public class ProductResource {

    @Inject
    ProductService productService;

    @GET
    public List<ProductDTO> list() {
        return productService.listAll();
    }

    @GET
    @Path("/{id}")
    public RestResponse<ProductDTO> getById(@PathParam("id") Long id) {
        return productService.findById(id)
            .map(RestResponse::ok)
            .orElse(RestResponse.notFound());
    }

    @POST
    public RestResponse<ProductDTO> create(@Valid CreateProductRequest request) {
        var product = productService.create(request);
        return RestResponse.created(URI.create("/products/" + product.id()));
    }
}
```

**Rules:**
- Use `RestResponse<T>` for proper HTTP status codes
- Use `@Valid` with Jakarta Validation annotations on request bodies
- Handle exceptions with `@ServerExceptionMapper`
- Group related endpoints in a single `*Resource` class

> Exception handling: https://quarkus.io/guides/rest#exception-mapping

```java
@ServerExceptionMapper
public RestResponse<ErrorResponse> mapNotFoundException(NotFoundException e) {
    return RestResponse.status(Response.Status.NOT_FOUND,
        new ErrorResponse(e.getMessage()));
}
```

## Data Access with Panache

> Guide (Active Record): https://quarkus.io/guides/hibernate-orm-panache
> Guide (Repository): https://quarkus.io/guides/hibernate-orm-panache#solution-2-using-the-repository-pattern

### Active Record Pattern

```java
@Entity
public class Product extends PanacheEntity {
    public String name;

    @Column(precision = 10, scale = 2)
    public BigDecimal price;

    public boolean active;

    // Custom query methods
    public static List<Product> findActive() {
        return find("active", true).list();
    }

    public static Optional<Product> findByName(String name) {
        return find("name", name).firstResultOptional();
    }
}
```

### Repository Pattern

```java
@ApplicationScoped
public class ProductRepository implements PanacheRepository<Product> {

    public List<Product> findActive() {
        return find("active", true).list();
    }

    public Optional<Product> findByName(String name) {
        return find("name", name).firstResultOptional();
    }
}
```

**Rules:**
- Choose one pattern (Active Record or Repository) and be consistent within the project
- Use Panache query methods over raw JPQL/HQL when possible
- Always return `Optional` for single-result queries

## CDI and Dependency Injection

> Guide: https://quarkus.io/guides/cdi

```java
@ApplicationScoped
public class ProductService {

    private final ProductRepository repository;

    @Inject  // Constructor injection preferred
    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }
}
```

**Scopes:**
- `@ApplicationScoped` — one instance per application (most services)
- `@RequestScoped` — one instance per HTTP request
- `@Dependent` — new instance per injection point (default, rarely used explicitly)
- `@Singleton` — like `@ApplicationScoped` but not proxied (use sparingly)

**Rules:**
- Prefer constructor injection; field injection with `@Inject` is acceptable for simplicity
- Use `@ApplicationScoped` as the default scope for services and repositories
- Never use `new` to create CDI beans manually

## Configuration

> Guide: https://quarkus.io/guides/config-reference

### Using @ConfigProperty

```java
@ApplicationScoped
public class EmailService {

    @ConfigProperty(name = "email.from", defaultValue = "noreply@example.com")
    String fromAddress;

    @ConfigProperty(name = "email.max-retries")
    int maxRetries;
}
```

### Using @ConfigMapping (preferred for groups)

```java
@ConfigMapping(prefix = "email")
public interface EmailConfig {
    String from();

    @WithDefault("3")
    int maxRetries();

    Optional<String> replyTo();
}
```

### Profile-Based Configuration
Use `%dev.`, `%test.`, `%prod.` prefixes in `application.properties`:

```properties
# Common
quarkus.datasource.db-kind=postgresql

# Dev (uses Dev Services by default)
%dev.quarkus.hibernate-orm.database.generation=drop-and-create

# Test
%test.quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/testdb

# Prod
%prod.quarkus.datasource.jdbc.url=${DATABASE_URL}
%prod.quarkus.hibernate-orm.database.generation=none
```

## Testing

> Guide: https://quarkus.io/guides/getting-started-testing

### Unit-like Tests with @QuarkusTest

```java
@QuarkusTest
class ProductResourceTest {

    @Test
    void shouldListProducts() {
        given()
            .when().get("/products")
            .then()
            .statusCode(200)
            .body("$.size()", greaterThan(0));
    }

    @Test
    void shouldReturn404ForUnknownProduct() {
        given()
            .when().get("/products/999999")
            .then()
            .statusCode(404);
    }
}
```

### Integration Tests with @QuarkusIntegrationTest

```java
@QuarkusIntegrationTest
class ProductResourceIT extends ProductResourceTest {
    // Runs the same tests against the packaged application
}
```

**Rules:**
- Use `@QuarkusTest` for tests that need CDI and the full Quarkus stack
- Use `@QuarkusIntegrationTest` for testing the packaged artifact (jar/native)
- Use `@InjectMock` for mocking CDI beans in tests
- Use REST Assured (auto-configured) for HTTP endpoint testing
- Use `@TestProfile` for test-specific configuration

## Virtual Threads in Quarkus

> Guide: https://quarkus.io/guides/virtual-threads

```java
@Path("/reports")
@ApplicationScoped
public class ReportResource {

    @GET
    @RunOnVirtualThread
    public Report generateReport() {
        // Blocking I/O is fine here — runs on a virtual thread
        var data = fetchFromDatabase();
        return buildReport(data);
    }
}
```

## Extensions Management

```bash
# List available extensions
quarkus ext list

# Add an extension
quarkus ext add rest-jackson
quarkus ext add hibernate-orm-panache

# Remove an extension
quarkus ext remove rest-jackson
```

**Rules:**
- Never add extensions without asking the user first
- Check existing extensions in `pom.xml`/`build.gradle` before suggesting additions
- Use `quarkus ext add` (the Quarkus CLI) instead of manually editing the POM

## Build and Run Commands

```bash
# Dev mode with live reload
./mvnw quarkus:dev          # Maven
./gradlew quarkusDev        # Gradle

# Run tests
./mvnw test                 # Maven
./gradlew test              # Gradle

# Build
./mvnw package              # Maven
./gradlew build             # Gradle

# Native build (requires GraalVM)
./mvnw package -Dnative     # Maven
./gradlew build -Dquarkus.native.enabled=true  # Gradle
```

## Common Extensions Reference

| Extension | Artifact | Use Case |
|-----------|----------|----------|
| Quarkus REST | `quarkus-rest` | REST endpoints |
| REST Jackson | `quarkus-rest-jackson` | JSON serialization |
| Hibernate ORM Panache | `quarkus-hibernate-orm-panache` | Data access |
| PostgreSQL | `quarkus-jdbc-postgresql` | PostgreSQL driver |
| SmallRye OpenAPI | `quarkus-smallrye-openapi` | OpenAPI/Swagger |
| SmallRye Health | `quarkus-smallrye-health` | Health checks |
| SmallRye Fault Tolerance | `quarkus-smallrye-fault-tolerance` | Retries, circuit breakers |
| Scheduler | `quarkus-scheduler` | Scheduled tasks |
| Mailer | `quarkus-mailer` | Email sending |

> Full extensions list: https://quarkus.io/extensions/
