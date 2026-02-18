# AI Skills for Claude Code and compatible tools

## [methodical-dev](./methodical-dev/)

 - 👤 Authors: [@k33gorg](https://bsky.app/profile/k33gorg.bsky.social)
 - 📜 Source reference: https://codeberg.org/ai-skills/methodical-dev

The methodical-dev skill enforces a structured, step-by-step development workflow with five phases:

  1. Information Gathering — Asks about feature scope, technical constraints, documentation references, and coding conventions. Also corrects the user's English grammar.
  1. Git Verification — Checks repo status and recommends (but never auto-creates) a dedicated feature branch.
  1. Detailed Planning — Analyzes existing code, creates a 5-8 step plan via TodoWrite, and waits for user approval before proceeding.  
  1. Guided Implementation — Executes one step at a time with mandatory checkpoints after each. The user must validate before the next step begins. No shortcuts, no unauthorized deletions, no architecture changes without consent.
  1. Final Validation — Provides a full summary of changes, runs through a quality checklist, and proposes a conventional commit message (with emoji) — but never commits automatically.

**Core principle:** The user stays in control at all times. The AI must stop after every step, explain decisions transparently, and never skip validation or silently modify existing code.

## [java-dev](./java-dev/)

 - 👤 Authors: [@wildagsx](https://bsky.app/profile/wilda.bsky.social) inspired by [@k33gorg](https://bsky.app/profile/k33gorg.bsky.social)

A **Java-first** development skill that extends the methodical-dev workflow for modern Java 25 projects. It auto-detects (or asks about) the project's technology stack and activates framework-specific guidance conditionally:

  - **Java 25** (always active) — Encourages modern language features: records, sealed classes, pattern matching, virtual threads, structured concurrency, scoped values, module imports, compact source files, and more.
  - **Quarkus** (conditional) — Activated when `io.quarkus` is detected. Enforces CDI-first design, Quarkus REST, Panache for data access, SmallRye MicroProfile, profile-based configuration, and `@QuarkusTest`.
  - **JBang** (conditional) — Activated when `//DEPS` directives or `jbang-catalog.json` are detected. Provides rules for single-file Java scripts with dependency management and Quarkus integration.
  - **LangChain4j** (conditional) — Activated when `dev.langchain4j` or `quarkus-langchain4j` is detected. Covers AI Services, Tools/function calling, RAG, Chat Memory, prompt engineering, and model configuration.

Reference files in `java-dev/references/` provide detailed, technology-specific guidance that is loaded on demand for token efficiency.

**Core principle:** Same as methodical-dev — the user stays in control at all times — with the added guarantee that all code follows modern Java 25 best practices and framework-specific conventions where applicable.
