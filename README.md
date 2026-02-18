# ✨ AI Skills for Claude code and compatible tools ✨

## 🧑‍💻 [methodical-dev](./methodical-dev/) 🧑‍💻

 - 👤: [@k33gorg](https://bsky.app/profile/k33gorg.bsky.social)
 - 🐙: https://codeberg.org/ai-skills/methodical-dev

The methodical-dev skill enforces a structured, step-by-step development workflow with five phases:

  1. Information Gathering — Asks about feature scope, technical constraints, documentation references, and coding conventions. Also corrects the user's English grammar.
  1. Git Verification — Checks repo status and recommends (but never auto-creates) a dedicated feature branch.
  1. Detailed Planning — Analyzes existing code, creates a 5–8 step plan via TodoWrite, and waits for user approval before proceeding.  
  1. Guided Implementation — Executes one step at a time with mandatory checkpoints after each. The user must validate before the next step begins. No shortcuts, no unauthorized deletions, no architecture changes without consent.
  1. Final Validation — Provides a full summary of changes, runs through a quality checklist, and proposes a conventional commit message (with emoji) — but never commits automatically.

**Core principle:** The user stays in control at all times. The AI must stop after every step, explain decisions transparently, and never skip validation or silently modify existing code.

## :coffee: [java-dev](./java-dev/) :coffee:

The java-dev skill extends the methodical-dev workflow for **Java 25, Quarkus, and JBang** development. It inherits the same five-phase structure and adds:

  1. **Java 25 guidance** — Encourages modern language features: records, sealed classes, pattern matching, virtual threads, module imports, compact source files, structured concurrency, and scoped values.
  1. **Quarkus conventions** — Enforces CDI-first design, Quarkus REST (not legacy RESTEasy), Panache for data access, SmallRye MicroProfile implementations, profile-based configuration, and proper testing with `@QuarkusTest`.
  1. **JBang support** — Provides rules for single-file Java scripts with `///` directives, dependency management, and Quarkus integration via JBang.
  1. **Java-specific strict rules** — No `java.util.Date`, no `null` returns from public methods, records for DTOs, Jakarta Validation annotations, proper exception handling with `@ServerExceptionMapper`.
  1. **Enhanced communication format** — Each step includes Java/Quarkus-specific details (pattern used, extensions involved, package structure) and test suggestions.

**Core principle:** Same as methodical-dev — the user stays in control at all times — with the added guarantee that all code follows modern Java 25 and Quarkus best practices.