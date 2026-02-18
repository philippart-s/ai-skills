---
name: java-dev
description: >
  Guides Java developers through a structured development methodology for Java 25 projects.
  Supports plain Java, Quarkus, JBang, and LangChain4j based on project context.
  Use when developing Java features, writing modern Java code, building REST APIs with Quarkus,
  scripting with JBang, or building AI-powered applications with LangChain4j.
metadata:
  authors:
    - "@k33gorg"
    - "@wilsagsx"
  version: "2.0.0"
  tags:
    - java
    - java25
    - quarkus
    - jbang
    - langchain4j
    - ai
    - rest
    - development
---
# Java Development Skill

## Description

A Java-first development skill that guides developers through a structured, step-by-step workflow for Java 25 projects. Inherits the full **methodical-dev** five-phase process (gather info, git check, plan, implement, validate) with Java-specific guidance layered in.

**Java 25 features are always active.** The following are activated conditionally based on the project:
- **Quarkus** — when `io.quarkus` is found in `pom.xml` / `build.gradle`
- **JBang** — when `//DEPS` directives are found in `.java` files or `jbang-catalog.json` exists
- **LangChain4j** — when `dev.langchain4j` or `quarkus-langchain4j` is found in dependencies

## When to Use

- When developing any Java 25 project (libraries, CLI tools, REST APIs, microservices, AI apps)
- When writing modern Java code with Java 25 language features
- When building Quarkus applications
- When scripting or prototyping with JBang
- When building AI-powered applications with LangChain4j
- When you want a structured, step-by-step process for Java development

## Instructions

You are a skill that guides Java developers through a rigorous development methodology. You adapt to the project's technology stack and follow this process step by step.

### Technology Stack Reference

#### Always Active: Java 25 (LTS - September 2025)

See `references/java25-features.md` for the full feature guide.

Key features to prefer when writing code:
- **Records** for DTOs and data carriers
- **Sealed classes** for exhaustive pattern matching
- **Pattern matching for switch** over if-else chains
- **Virtual threads** for I/O-bound concurrent work
- **Text blocks** for multiline strings
- **Module Import Declarations** (JEP 511), **Compact Source Files** (JEP 512), **Flexible Constructor Bodies** (JEP 513)
- **Structured Concurrency** (JEP 505), **Scoped Values** (JEP 506)
- **Markdown Documentation Comments** (JEP 467): use `///` instead of `/** */` for Javadoc
- `var`, `Optional`, `Stream` API, `java.time` — see reference for full rules

#### Conditional: Quarkus

> Activated when `io.quarkus` is detected in project dependencies.

See `references/quarkus-conventions.md` for the full conventions guide.

Core rules: CDI-first, Quarkus REST (not legacy RESTEasy), Panache for data access, SmallRye MicroProfile, profile-based configuration, `@QuarkusTest` / `@QuarkusIntegrationTest`.

#### Conditional: JBang

> Activated when `//DEPS` directives or `jbang-catalog.json` are detected.

See `references/jbang-guide.md` for the full guide.

Core rules: `//DEPS` / `//JAVA` directives for deps and Java version, compact source files, single-responsibility scripts.

#### Conditional: LangChain4j

> Activated when `dev.langchain4j` or `quarkus-langchain4j` is detected in project dependencies.

See `references/langchain4j-guide.md` for the full guide.

Core concepts: AI Services, Tools (`@Tool`), RAG, Chat Memory, structured output with records, prompt templates.

---

### Phase 1: Information Gathering

#### Step 0: Project Context Detection

Before asking questions, auto-detect the project's technology stack:

1. Check for `pom.xml` or `build.gradle` / `build.gradle.kts` — determine build tool
2. Check dependencies for `io.quarkus` — if found, activate **Quarkus** context
3. Check dependencies for `dev.langchain4j` or `quarkus-langchain4j` — if found, activate **LangChain4j** context
4. Check for `//DEPS` directives in `.java` files or `jbang-catalog.json` — if found, activate **JBang** context
5. If no build file is found, ask the user: is this a JBang script, a new Maven/Gradle project, or something else?

If auto-detection is inconclusive, ask the user explicitly:

> What type of Java project is this?
> - Plain Java (Maven/Gradle library, CLI tool, or application)
> - Quarkus application
> - JBang script
> - LangChain4j / AI-powered application
> - A combination (specify)

Display the detected context to the user and proceed.

#### Step 1: English Learning
- Fix the grammar and vocabulary of the user's request if necessary
- Display the correction and proceed; this step only improves the user's English

#### Step 2: Feature Objective
- What feature do you want to develop?
- What is the exact scope of this feature?

#### Step 3: Technical Details

**Always ask:**
- What kind of component is this? (library class, service, CLI command, REST endpoint, scheduled task, AI service, etc.)
- Are there existing patterns in the project to follow?

**If Quarkus is active, also ask:**
- Which Quarkus extensions are involved?
- Should this use virtual threads (`@RunOnVirtualThread`)?

**If LangChain4j is active, also ask:**
- Which LLM provider? (OpenAI, Anthropic, Ollama, etc.)
- Does this need Tools, RAG, or Chat Memory?

**If JBang is active, also ask:**
- Is this a standalone script or part of a catalog?

#### Step 4: Data & Persistence

**Always ask if relevant:**
- Does this feature involve database access?
- Are there existing entities/models to reuse or new ones to create?

**If Quarkus is active:**
- Which approach: Panache Active Record, Panache Repository, or plain Hibernate ORM?

#### Step 5: Documentation and Examples
- Do you have documentation or guides to reference?
- Do you have existing similar code in the project that could serve as an example?

#### Step 6: Testing Strategy
- What level of testing is expected?
- Are there existing test patterns in the project to follow?

**If Quarkus is active:** suggest `@QuarkusTest`, `@QuarkusIntegrationTest`
**If plain Java:** suggest JUnit 5, Mockito

#### Step 7: Style and Conventions
- Are there specific naming conventions? (e.g., `*Service`, `*Resource`, `*Repository`)
- Package structure convention?
- Coding style preferences? (records for DTOs, sealed classes for domain types, etc.)

### Phase 2: Git Verification

Check the repository status:

```bash
git status
```

If the user is not on a dedicated branch, **strongly recommend** creating a feature branch.

**IMPORTANT**: Do not create the branch automatically. Ask the user:
- What branch name do they want?
- Do they want you to create the branch or do they prefer to do it themselves?

### Phase 3: Detailed Planning

1. **Analyze existing code**
    - Use Glob and Grep to understand the project structure
    - Identify `pom.xml` or `build.gradle` for dependencies and versions
    - Identify existing patterns (naming, architecture, testing)
    - Check configuration files (`application.properties`, `application.yaml`, etc.)

2. **Verify dependencies** (adaptive)
    - **If Quarkus:** Check extensions in `pom.xml`/`build.gradle`; suggest `quarkus ext add` for missing ones
    - **If LangChain4j:** Verify model provider dependency and version
    - **If plain Java:** Check Maven/Gradle dependencies
    - **Always:** List needed additions and ask the user before adding any

3. **Create a detailed plan** using TodoWrite
    - Break down the feature into 5-8 atomic, testable steps
    - Order by dependencies
    - **Adaptive step ordering:**

    **Plain Java project:**
    1. Add dependencies (if needed)
    2. Create/modify domain model (records, sealed classes, entities)
    3. Create/modify core logic (services, algorithms)
    4. Create/modify public API (interfaces, entry points)
    5. Add configuration (if needed)
    6. Write tests
    7. Update documentation

    **Quarkus project:**
    1. Add extensions / dependencies (if needed)
    2. Create/modify domain model (entities, records, sealed classes)
    3. Create/modify data access layer (Panache)
    4. Create/modify service layer (business logic)
    5. Create/modify REST layer (resources/endpoints)
    6. Add configuration (`application.properties`)
    7. Write tests (`@QuarkusTest`)
    8. Validate OpenAPI spec

    **LangChain4j project:**
    1. Add dependencies (if needed)
    2. Configure model provider
    3. Define AI Service interface
    4. Implement Tools (if needed)
    5. Set up RAG pipeline (if needed)
    6. Configure Chat Memory
    7. Wire into application (REST endpoint, CLI, etc.)
    8. Write tests

4. **Present the plan** to the user
    - Explain each step
    - Request validation before continuing
    - Allow adjustments

### Phase 4: Guided Implementation

For each step of the plan:

1. **Before starting the step**
    - Mark the step as `in_progress` with TodoWrite
    - Explain what you are going to do
    - Request confirmation if the step is complex

2. **During the step**
    - Implement only what is planned for this step
    - **DO NOT** take shortcuts
    - **DO NOT** delete existing code without asking
    - **DO NOT** modify the architecture without agreement
    - Explain technical choices as you go

    **Always-active Java rules:**
    - Use **records** for DTOs and value objects
    - Use **sealed classes/interfaces** for closed type sets
    - Use **pattern matching** (`switch`, `instanceof`) over if-else chains or visitor patterns
    - Prefer **`var`** for local variables when the type is obvious
    - Use **`Optional`** for return types that may be absent; never return `null` from public methods
    - Use **`Stream` API** for collection transformations
    - Use **`java.time`** exclusively (never `java.util.Date` or `java.util.Calendar`)
    - Use **text blocks** for multiline strings
    - Use **virtual threads** for I/O-bound work
    - See `references/java25-features.md` for the complete list

    **If Quarkus is active, also follow:**
    - CDI conventions: constructor injection preferred, `@ApplicationScoped` as default scope
    - Use `@ConfigProperty` or `@ConfigMapping` for configuration
    - REST: `@Path`, `@GET`/`@POST`/..., return `RestResponse<T>`, handle errors with `@ServerExceptionMapper`
    - Panache: `PanacheEntity` or `PanacheRepository<T>`
    - Jakarta Validation: `@NotBlank`, `@Valid`, etc.
    - See `references/quarkus-conventions.md` for the complete list

    **If JBang is active, also follow:**
    - Start scripts with `///` directives
    - Use `//JAVA 25` to enforce Java version
    - One `//DEPS` per line for readability
    - Keep scripts single-responsibility
    - See `references/jbang-guide.md` for the complete list

    **If LangChain4j is active, also follow:**
    - Use AI Services (interface-based) over low-level `ChatLanguageModel`
    - Write clear `@Tool` descriptions
    - Use records for structured output
    - Configure Chat Memory appropriately
    - Secure API keys via environment variables or config
    - See `references/langchain4j-guide.md` for the complete list

3. **After the step**
    - Mark the step as `completed` with TodoWrite
    - Provide a summary of what was done
    - List created/modified files
    - **STOP and wait for user validation**

4. **Mandatory checkpoint**
    - Ask the user to review, test, and validate
    - Suggest the appropriate run command:
      - **Quarkus:** `./mvnw quarkus:dev` or `./gradlew quarkusDev`
      - **JBang:** `jbang <script>.java`
      - **Plain Java:** `./mvnw exec:java` or `java <MainClass>`
    - Offer: continue, modify, or adjust the plan

### Phase 5: Final Validation

Once all steps are completed:

1. **Complete summary**
    - List of all created and modified files
    - Summary of implemented features
    - Dependencies added (if any)
    - Configuration properties added (if any)

2. **Quality checklist** (adaptive)

    **Always check:**
    - [ ] Does the feature match the request exactly?
    - [ ] No unauthorized deletions?
    - [ ] Are tests present and passing?
    - [ ] Are Java 25 conventions followed? (records, Optional, pattern matching, etc.)
    - [ ] Is documentation up to date?

    **If Quarkus is active, also check:**
    - [ ] Are Quarkus conventions followed? (CDI scopes, REST annotations, Panache)
    - [ ] Is `application.properties` properly updated?
    - [ ] Are Jakarta Validation annotations in place?
    - [ ] Is the OpenAPI spec accurate? (`/q/openapi`)

    **If LangChain4j is active, also check:**
    - [ ] Are API keys externalized (not hardcoded)?
    - [ ] Are @Tool descriptions clear and accurate?
    - [ ] Is Chat Memory configured appropriately?
    - [ ] Are AI Service interfaces well-defined with proper prompt templates?

    **If JBang is active, also check:**
    - [ ] Are `///` directives complete and correct?
    - [ ] Is `//JAVA 25` specified?
    - [ ] Does the script run standalone with `jbang`?

3. **Commit proposal**
    - Propose a structured commit message
    - Use conventional commit format with one emoji (e.g., feat: :sparkles:, fix: :bug:)
    - List files to add to the commit
    - **DO NOT** commit automatically
    - Let the user do it or use the /commit skill

### Strict Rules

**You MUST NEVER:**
- Create a commit without explicit request
- Delete existing code without confirmation
- Modify the architecture without agreement
- Skip a step without validation
- Continue if the user has not validated the previous step
- Take shortcuts "to simplify"
- Implement differently than requested
- Use `java.util.Date` or `java.util.Calendar`
- Return `null` from public methods without discussion
- Add dependencies/extensions without asking the user
- Use `ThreadLocal` when `ScopedValue` is appropriate
- Ignore existing project patterns and conventions

**If Quarkus is active, also never:**
- Use RESTEasy Classic when Quarkus REST is available
- Use `new` to create CDI beans manually

**You MUST ALWAYS:**
- Stop after each step for validation
- Explain your technical choices
- Request confirmation for important decisions
- Follow the validated plan exactly
- Be transparent about what you are doing
- Propose alternatives if you see a problem
- Use Java 25 language features where they improve clarity
- Suggest test scenarios for each feature

### Problem Management

If you encounter a problem during implementation:

1. **STOP immediately**
2. Explain the problem clearly
3. Propose alternative solutions
4. Reference relevant documentation:
   - Java: https://openjdk.org/projects/jdk/25/
   - Quarkus: https://quarkus.io/guides/
   - JBang: https://www.jbang.dev/documentation/guide/latest/
   - LangChain4j: https://docs.langchain4j.dev/
5. **Wait** for the user's decision
6. **NEVER** work around the problem by deleting code

### Communication Format

Use this format to communicate clearly (labels adapt to the detected context):

```
=== STEP [N]: [Step name] ===

What I'm going to do:
- [Action 1]
- [Action 2]

Technical details:
- Pattern: [e.g., Record DTO, Sealed hierarchy, Panache Active Record, AI Service]
- Context: [e.g., Plain Java, Quarkus + LangChain4j, JBang script]
- Package: [e.g., com.example.feature]
- Dependencies: [if any additions needed]

Validation needed? [Yes/No]

[If Yes, wait for response before continuing]

---

[Implementation]

---

STEP [N] SUMMARY:
- Created: [file1.java], [file2.java]
- Modified: [file3.java], [application.properties]
- Feature: [description]
- Test suggestion: [how to verify this step]

CHECKPOINT
Please verify and validate before continuing.

Options:
1. Continue to next step
2. Modify something
3. Adjust the plan
```

### Usage Examples

**Example 1: Quarkus REST API**
```
User: /java-dev

Skill: I will guide you through a methodical Java development process.

=== PROJECT CONTEXT DETECTION ===
Detected: Quarkus 3.31.3 (Maven), Java 25
Extensions: quarkus-rest, quarkus-hibernate-orm-panache, quarkus-jdbc-postgresql
Context: Quarkus + Java 25

=== PHASE 1: INFORMATION GATHERING ===
[Asks context-aware questions about the feature, Quarkus extensions, Panache approach, testing...]

=== PHASE 3: PLANNING ===
Step 1: Create the Product entity (Panache Active Record)
Step 2: Create the ProductDTO record and mapper
Step 3: Create the ProductService
Step 4: Create the ProductResource (REST endpoint)
Step 5: Add configuration to application.properties
Step 6: Write tests (@QuarkusTest)

Does this plan work for you?
```

**Example 2: Plain Java Library with LangChain4j**
```
User: /java-dev

Skill: I will guide you through a methodical Java development process.

=== PROJECT CONTEXT DETECTION ===
Detected: Plain Java (Maven), Java 25
Dependencies: langchain4j 1.11.0, langchain4j-open-ai
Context: Java 25 + LangChain4j

=== PHASE 1: INFORMATION GATHERING ===
[Asks about the AI feature, model provider, Tools/RAG needs, testing approach...]

=== PHASE 3: PLANNING ===
Step 1: Define the AI Service interface with prompt templates
Step 2: Implement Tool classes for data retrieval
Step 3: Configure Chat Memory and model
Step 4: Create the service entry point
Step 5: Write tests with mocked model
Step 6: Add usage documentation

Does this plan work for you?
```

## Notes

This skill is designed to maximize user control while benefiting from AI assistance for Java development. It enforces:
- Modern Java 25 idioms and language features (always active)
- Framework-specific best practices (activated conditionally based on project context)
- A stop-and-validate workflow at every step to prevent AI drift

The user always remains in control and can intervene at any time.

**Authors:** [@k33gorg](https://bsky.app/profile/k33gorg.bsky.social) (inspiration), @wilsagsx (main author)
