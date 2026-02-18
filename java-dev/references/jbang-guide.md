# JBang Guide Reference

> Activate this reference when the project uses JBang (detected via `//DEPS` directives in `.java` files, `jbang-catalog.json`, or user request).
> Documentation: https://www.jbang.dev/documentation/guide/latest/

## What is JBang?

JBang lets you run Java code as scripts without a build tool. It handles dependency resolution, compilation, and execution in a single step. Perfect for:
- Single-file Java scripts and utilities
- Prototyping and experimentation
- CLI tools
- Quick Quarkus applications

## Core Directives

JBang uses `///` comment directives at the top of Java files:

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 25
//DEPS com.google.code.gson:gson:2.11.0
//DEPS info.picocli:picocli:4.7.6

import com.google.gson.Gson;
import picocli.CommandLine;

void main(String... args) {
    // script logic
}
```

### Directive Reference

| Directive | Purpose | Example |
|-----------|---------|---------|
| `//JAVA <version>` | Enforce Java version | `//JAVA 25` |
| `//DEPS <gav>` | Add a dependency (Maven GAV) | `//DEPS io.quarkus:quarkus-rest:3.31.3` |
| `//DEPS <gav>@pom` | Import a BOM | `//DEPS io.quarkus.platform:quarkus-bom:3.31.3@pom` |
| `//SOURCES <file>` | Include additional source files | `//SOURCES helper.java` |
| `//FILES <file>` | Include resource files | `//FILES application.properties` |
| `//REPOS <url>` | Add Maven repository | `//REPOS mavencentral,myrepo=https://repo.example.com/maven` |
| `//JAVAC_OPTIONS` | Compiler options | `//JAVAC_OPTIONS --enable-preview` |
| `//JAVA_OPTIONS` | Runtime JVM options | `//JAVA_OPTIONS -Xmx512m` |
| `//COMPILE_OPTIONS` | Compilation flags | `//COMPILE_OPTIONS --enable-preview` |
| `//RUNTIME_OPTIONS` | Runtime flags | `//RUNTIME_OPTIONS --enable-preview` |
| `//GAV <group:artifact:version>` | Set script GAV coordinates | `//GAV com.example:myscript:1.0` |
| `//DESCRIPTION <text>` | Script description | `//DESCRIPTION A utility for data conversion` |

> Directives reference: https://www.jbang.dev/documentation/guide/latest/usage.html

## Dependency Management

One `//DEPS` per line for readability:

```java
//DEPS io.quarkus.platform:quarkus-bom:3.31.3@pom
//DEPS io.quarkus:quarkus-rest
//DEPS io.quarkus:quarkus-rest-jackson
//DEPS io.quarkus:quarkus-hibernate-orm-panache
//DEPS io.quarkus:quarkus-jdbc-h2
```

**Rules:**
- Use `@pom` suffix for BOM imports (controls versions of other deps)
- When using a BOM, you can omit versions for managed dependencies
- One dependency per line for clarity
- Group related dependencies together

## Running JBang Scripts

```bash
# Run directly
jbang script.java

# Run with arguments
jbang script.java --input data.csv --output result.json

# Run from URL
jbang https://gist.github.com/user/abc123/raw/script.java

# Edit in IDE (opens with proper classpath)
jbang edit script.java

# Export to Maven project
jbang export mavenrepo script.java

# Install as command
jbang app install --name mytool script.java
```

> Running guide: https://www.jbang.dev/documentation/guide/latest/running.html

## JBang with Quarkus

JBang can run full Quarkus applications from a single file:

```java
//JAVA 25
//DEPS io.quarkus.platform:quarkus-bom:3.31.3@pom
//DEPS io.quarkus:quarkus-rest
//DEPS io.quarkus:quarkus-rest-jackson

import jakarta.ws.rs.*;
import jakarta.enterprise.context.ApplicationScoped;

@Path("/hello")
@ApplicationScoped
public class HelloApp {

    record Greeting(String message, String timestamp) {}

    @GET
    public Greeting hello(@QueryParam("name") @DefaultValue("World") String name) {
        return new Greeting("Hello, " + name + "!", java.time.Instant.now().toString());
    }
}
```

```bash
jbang HelloApp.java
# Quarkus starts automatically, endpoint at http://localhost:8080/hello
```

> Quarkus + JBang guide: https://quarkus.io/guides/scripting

## JBang Catalogs

Organize reusable scripts with `jbang-catalog.json`:

```json
{
  "catalogs": {},
  "aliases": {
    "db-migrate": {
      "script-ref": "scripts/db-migrate.java",
      "description": "Run database migrations"
    },
    "generate-report": {
      "script-ref": "scripts/generate-report.java",
      "description": "Generate monthly report",
      "arguments": ["--format", "pdf"]
    }
  }
}
```

```bash
# Run a catalog alias
jbang db-migrate
jbang generate-report
```

> Catalogs guide: https://www.jbang.dev/documentation/guide/latest/alias_catalogs.html

## Multi-File JBang Projects

For scripts that need multiple source files:

```java
// main.java
//SOURCES utils/Helper.java
//SOURCES models/DataModel.java
//FILES application.properties

void main() {
    var helper = new Helper();
    helper.process();
}
```

**Rules:**
- Use `//SOURCES` for additional Java source files
- Use `//FILES` for resource files (properties, JSON, etc.)
- Keep JBang scripts focused on a single responsibility
- For complex projects with many files, consider migrating to a Maven/Gradle project

## CLI Tools with Picocli

JBang pairs well with Picocli for building CLI tools:

```java
//JAVA 25
//DEPS info.picocli:picocli:4.7.6

import picocli.CommandLine;
import picocli.CommandLine.*;
import java.nio.file.*;

@Command(name = "converter", mixinStandardHelpOptions = true, version = "1.0",
         description = "Converts data files between formats")
public class Converter implements Runnable {

    @Parameters(index = "0", description = "Input file")
    Path inputFile;

    @Option(names = {"-f", "--format"}, description = "Output format", defaultValue = "json")
    String format;

    @Option(names = {"-o", "--output"}, description = "Output file")
    Path outputFile;

    @Override
    public void run() {
        // conversion logic
    }

    void main(String... args) {
        new CommandLine(new Converter()).execute(args);
    }
}
```

## Best Practices

- Always use `//JAVA 25` to pin the Java version
- Prefer compact source files (JEP 512) with `void main()` for scripts
- Keep scripts self-contained — all dependencies declared via `//DEPS`
- Use JBang for prototyping; migrate to Maven/Gradle when the project grows
- Use `jbang edit` to get IDE support with proper classpath resolution
- For Quarkus scripts, use the BOM to manage versions consistently
