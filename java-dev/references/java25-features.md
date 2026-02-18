# Java 25 Language Features Reference

> Java 25 is an LTS release (September 2025). Prefer these features when writing new code.
> Documentation: https://openjdk.org/projects/jdk/25/

## Finalized Features

### Records
Use records for DTOs, value objects, and data carriers. Records are immutable by default and provide `equals()`, `hashCode()`, and `toString()`.

```java
public record ProductDTO(String name, BigDecimal price, String description) {}
```

**Rules:**
- Always use records instead of POJOs for data transfer
- Records can implement interfaces but cannot extend classes
- Use compact constructors for validation

### Sealed Classes and Interfaces
Use sealed hierarchies to model closed sets of types. Enables exhaustive pattern matching in `switch`.

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}
public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double base, double height) implements Shape {}
```

### Pattern Matching for switch
Use modern switch expressions with patterns instead of if-else chains or visitor patterns.

```java
String describe(Shape shape) {
    return switch (shape) {
        case Circle c    -> "Circle with radius " + c.radius();
        case Rectangle r -> "Rectangle %sx%s".formatted(r.width(), r.height());
        case Triangle t  -> "Triangle with base " + t.base();
    };
}
```

### Text Blocks
Use text blocks (`"""`) for multiline strings: SQL queries, JSON, HTML, XML.

```java
String query = """
    SELECT p.name, p.price
    FROM product p
    WHERE p.active = true
    ORDER BY p.name
    """;
```

### Virtual Threads
Prefer virtual threads for I/O-bound concurrent work. Use platform threads only for CPU-bound work.

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> fetchFromDatabase());
    executor.submit(() -> callExternalApi());
}
```

**Rules:**
- Never pool virtual threads
- Avoid `synchronized` blocks in virtual thread code (use `ReentrantLock` instead)
- Do not use `ThreadLocal` with virtual threads; prefer `ScopedValue`

## JEP Features in Java 25

### Module Import Declarations (JEP 511)
Import all exported packages of a module with a single statement.

```java
import module java.base;        // imports all of java.base
import module java.sql;          // imports all of java.sql
```

> Docs: https://openjdk.org/jeps/511

### Compact Source Files and Instance Main Methods (JEP 512)
Simplified entry points for scripts and small programs. No need for `public class` wrapper or `String[] args`.

```java
void main() {
    println("Hello, Java 25!");
}
```

> Docs: https://openjdk.org/jeps/512

### Flexible Constructor Bodies (JEP 513)
Statements can appear before `super()` or `this()` calls. Useful for argument validation and transformation.

```java
public class PositiveAmount extends Amount {
    public PositiveAmount(BigDecimal value) {
        if (value.signum() <= 0) throw new IllegalArgumentException("Must be positive");
        super(value);  // now valid after the check
    }
}
```

> Docs: https://openjdk.org/jeps/513

### Structured Concurrency (Fifth Preview, JEP 505)
Use `StructuredTaskScope` for managing concurrent subtasks with proper lifecycle and error handling.

```java
try (var scope = StructuredTaskScope.open()) {
    Subtask<User> userTask = scope.fork(() -> findUser(userId));
    Subtask<List<Order>> ordersTask = scope.fork(() -> fetchOrders(userId));
    scope.join();

    return new UserDashboard(userTask.get(), ordersTask.get());
}
```

**Rules:**
- Always use structured concurrency over manual `ExecutorService` for related subtasks
- Handle errors via the joiner policy (e.g., `ShutdownOnFailure`)
- The scope must be closed (use try-with-resources)

> Docs: https://openjdk.org/jeps/505

### Scoped Values (JEP 506)
Prefer `ScopedValue` over `ThreadLocal` for sharing immutable data within a thread or virtual thread.

```java
private static final ScopedValue<UserContext> CURRENT_USER = ScopedValue.newInstance();

ScopedValue.runWhere(CURRENT_USER, userContext, () -> {
    // CURRENT_USER.get() is available here and in all called methods
    processRequest();
});
```

> Docs: https://openjdk.org/jeps/506

### Primitive Types in Patterns (Third Preview, JEP 507)
Use primitives in `instanceof` and `switch` patterns for safer narrowing conversions.

```java
switch (statusCode) {
    case int i when i >= 200 && i < 300 -> "Success";
    case int i when i >= 400 && i < 500 -> "Client Error";
    case int i when i >= 500            -> "Server Error";
    default                             -> "Unknown";
}
```

> Docs: https://openjdk.org/jeps/507

### Stable Values (Preview, JEP 502)
Use `StableValue` for deferred initialization of effectively-final values.

```java
private final StableValue<DatabaseConnection> connection = StableValue.of();

DatabaseConnection getConnection() {
    return connection.orElseSet(this::createConnection);
}
```

> Docs: https://openjdk.org/jeps/502

## General Java Rules (Always Active)

- Use **`var`** for local variables when the type is obvious from context
- Use **`Optional`** for return types that may be absent; never return `null` from public methods
- Use **`Stream` API** for collection transformations; avoid mutating collections in-place
- Use **`java.time`** API exclusively (never `java.util.Date` or `java.util.Calendar`)
- Prefer **`List.of()`, `Map.of()`, `Set.of()`** for immutable collections
- Use **`String.formatted()`** or **`"""` text blocks** over `String.format()` concatenation
- Follow standard **naming conventions**: `PascalCase` for classes, `camelCase` for methods/variables, `SCREAMING_SNAKE_CASE` for constants
- Organize imports: `java.*`, then `jakarta.*`, then third-party, then project — no wildcard imports
- Write **Javadoc** for all public APIs
