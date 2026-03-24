---
description: "Use when writing Java, Spring Boot, or Gradle projects. Covers scaffold requirements, build configuration, testing with JUnit/JaCoCo, linting with Checkstyle/SpotBugs, and OpenAPI Generator setup."
applyTo: "**/*.java, **/build.gradle.kts, **/build.gradle"
---

## Java / Spring Boot Projects

Always ask about the following before generating any code or scaffold:

| Question | Options |
|----------|---------|
| Build tool | Gradle (preferred) w/ Groovy DSL |
| Database | PostgreSQL / MySQL |
| Auth | JWT / Basic auth / Skip |
| Entities | Basic / Extended |
| Output | Working code / Docs only |

**API Versioning**: Always use **URL path prefix** versioning (e.g. `/api/v1/meetings`). All controllers must be rooted at `/api/v1`. When a v2 is needed, add a new controller package alongside v1 — never modify existing versioned routes.

**Entity-first rule**: Just as interfaces come before implementations in code, entity design comes before the API spec. Never define request/response schemas in `openapi.yaml` until the entities have been reviewed and confirmed.

**Default**: Always start with interfaces only, then add implementations when explicitly asked.

## Scaffold Requirements for Spring Boot Projects

**BEFORE generating any code, verify that scaffold includes ALL of the following:**

### build.gradle must include
- ✅ Spring Boot starter plugins (web, data-jpa, security, validation)
- ✅ **JaCoCo** plugin — code coverage with **80% line coverage** minimum threshold enforced
- ✅ **Checkstyle** plugin (v10.12.1+) with rules configured in `config/checkstyle/checkstyle.xml`
- ✅ **SpotBugs** plugin with exclusion config in `config/spotbugs/exclude.xml`
- ✅ **OpenAPI Generator** plugin — wired into `compileJava` task (spec-first approach)
- ✅ All 4 testing task finalization: `test.finalizedBy jacocoTestReport`, `check.dependsOn jacocoTestCoverageVerification`, `check.dependsOn checkstyleMain checkstyleTest`, `check.dependsOn spotbugsMain`
- ✅ Database driver (PostgreSQL or MySQL as chosen)
- ✅ JWT library (jjwt)
- ✅ Lombok + Maven compiler plugin for annotation processing

### Config files that must exist
- ✅ `src/main/resources/application.properties` — database, JWT, server config
- ✅ `config/checkstyle/checkstyle.xml` — Checkstyle rules (omit `NoWhitespaceAtEndOfLine`, omit `MissingJavadocMethod`, include `LeftCurly`)
- ✅ `config/spotbugs/exclude.xml` — with suppressions for `api.*`, `model.*`, `controller.*` classes
- ✅ `.gitignore` — standard Java/Gradle ignores
- ✅ **Gradle wrapper** — always run `gradle wrapper --gradle-version <latest-stable>` after creating `build.gradle`; commit `gradlew`, `gradlew.bat`, and `gradle/wrapper/gradle-wrapper.properties` so the project builds without a local Gradle install

### Generated code (from OpenAPI Generator)
- ✅ Controller interfaces in `build/generated/src/main/java/com/zoomrival/api/`
- ✅ DTOs in `build/generated/src/main/java/com/zoomrival/model/`
- ✅ Do **NOT** manually create these — always use OpenAPI Generator

---

**Pre-generation checklist**: Before you output any scaffold code, present this checklist to the user and confirm all items will be included.

### Default dependencies
Always include in every Spring Boot project:
- **Lombok** — reduces boilerplate (`@Data`, `@Builder`, `@RequiredArgsConstructor`, etc.)
- **Mockito** — included via `spring-boot-starter-test` but always use it explicitly in unit tests

## Testing & Code Coverage

For all Spring Boot projects:
- Generate unit tests using JUnit 5 + Mockito
- Configure JaCoCo for code coverage reporting
- Enforce a minimum **80% line coverage** — build fails if below threshold
- Add the JaCoCo Gradle plugin and coverage rules to `build.gradle` by default

## Linting & Static Analysis

For all Spring Boot projects, always add:
- **Checkstyle** — code style enforcement; include a `config/checkstyle/checkstyle.xml`
- **SpotBugs** — static analysis for potential bugs; fail build on medium+ severity issues
- Both should be wired into the `check` task so they run with every build

### Checkstyle rules to omit
- Do **not** include `NoWhitespaceAtEndOfLine` — removed in newer Checkstyle versions, causes config failure
- Do **not** include `MissingJavadocMethod` at project start — too noisy before implementations exist; add later
- Use `LeftCurly` rule — requires `{` followed by a line break (no inline method bodies)

### SpotBugs exclusions
Always create `config/spotbugs/exclude.xml` with these suppressions:
- Suppress all bugs for `api.*` classes — OpenAPI-generated code is not hand-written and should not be statically analysed
- Suppress `EI_EXPOSE_REP` and `EI_EXPOSE_REP2` for `model.*` classes — JPA entities require direct mutable references; defensive copies break Hibernate change tracking
- Suppress `EI_EXPOSE_REP2` for `controller.*` classes — Spring constructor-injected beans are managed by Spring and do not need defensive copies

## OpenAPI / API Design

> **RULE: The OpenAPI spec MUST be written and reviewed before any controller, service, model, or scaffold code is generated. No exceptions.**

For all Spring Boot projects:
- **Spec-first** — write `src/main/resources/openapi.yaml` before any controller code
- Use **OpenAPI Generator Gradle plugin** to generate Spring controller interfaces and models from the spec
- Controllers implement the generated interfaces — spec is always the source of truth
- **Generate client SDKs** from the spec (default: Java; add others when asked)
- Spec and generated code live in the same repo; generated sources go to `build/generated` (not committed)
- Add `openApiGenerate` task as a dependency of `compileJava` so generation runs automatically on build

### Spec structure
- Place spec at `src/main/resources/openapi.yaml`
- Use `$ref` to split large specs into components (`schemas`, `paths`, `responses`)
- Always define `components/schemas` for all request/response bodies — no inline schemas

## Integration Tests

When asked to generate integration tests:
- Use **Testcontainers** with a `postgres` container matching the project's database
- **Container reuse is enabled** — add `testcontainers.reuse.enable=true` to `~/.testcontainers.properties`
- Integration tests live in `src/test/.../integration` package, separate from unit tests
- Always create a shared `AbstractIntegrationTest` base class that all integration tests extend:
  - Annotated with `@SpringBootTest`
  - Uses `@DynamicPropertySource` to inject container URL/credentials into the Spring context
  - Declares the container as a `static` field so it is shared across all tests in the class
- Integration tests run as a **separate Gradle task** (`./gradlew integrationTest`), not as part of `./gradlew check`
- Add a `integrationTest` source set and task to `build.gradle` with the Testcontainers dependency scoped to that source set only

## Database — Primary Key

- Use the `uuid-creator` library for UUID v7 generation
- Never use `@GeneratedValue(strategy = GenerationType.UUID)` or `UUID.randomUUID()` as a primary key
- Default to **UUID v7** unless the user explicitly chooses BIGSERIAL
