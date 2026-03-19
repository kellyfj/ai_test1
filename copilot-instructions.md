---
name: Project Instructions
description: "Workspace guidelines for all agent interactions"
---

# Coding Approach

## Present Ideas Before Code

When proposing solutions:
1. **Present 3–5 ideas first** with brief descriptions (what each does, tradeoffs)
2. **Wait for feedback** before implementing
3. Only write code after you've gotten input or guidance

This approach ensures we discuss options, constraints, and design upfront rather than jumping straight to implementation.

## Clarifying Questions

When asking for clarification, **always explain upfront what I'll do with the answers**. Don't just ask — say "I'll use this to generate X" or "This will determine Y." Be specific about the outcome.

## Response Brevity

- Keep responses **concise and scannable**
- Use bullet points and short paragraphs
- For multiple ideas, use a comparison table if helpful
- Avoid lengthy explanations; ask for clarification instead
- If something requires detail, offer it only if asked

## Java / Spring Boot Projects

Always ask about the following before generating any code or scaffold:

| Question | Options |
|----------|---------|
| Build tool | Maven / Gradle |
| Database | PostgreSQL / MySQL |
| Auth | JWT / Basic auth / Skip |
| Entities | Basic / Extended |
| Output | Working code / Docs only |

**Order of work — always follow this sequence:**
1. Write and confirm `openapi.yaml` spec first
2. Generate scaffold (entities, repositories, controller interfaces) from the spec
3. Write service interfaces
4. Add implementations when explicitly asked
5. Add unit tests
6. Add integration tests when explicitly asked

**Default**: Always start with interfaces only, then add implementations when explicitly asked.

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


