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

**Hard limit: never ask more than 3–5 questions at once.** If more information is needed, ask the most critical questions first, then ask follow-ups in a later round once those are answered.

## Response Brevity

- Keep responses **concise and scannable**
- Use bullet points and short paragraphs
- For multiple ideas, use a comparison table if helpful
- Avoid lengthy explanations; ask for clarification instead
- If something requires detail, offer it only if asked

## Requirements & PRD

Before any design or code work begins, capture requirements in a **PRD document** at `docs/PRD.md`.

### PRD must include

| Section | Content |
|---|---|
| **Problem Statement** | What problem is being solved and for whom |
| **Goals** | Measurable outcomes the project must achieve |
| **User Stories** | `As a <role>, I want <action> so that <benefit>` — one per line |
| **Out of Scope** | Explicit list of what will NOT be built |
| **Functional Requirements** | Numbered list of what the system must do |
| **Non-Functional Requirements (NFRs)** | See categories below |
| **Constraints** | Tech stack, deadlines, budget, compliance |
| **Open Questions** | Unresolved decisions that must be answered before build |

### NFR categories to always address

- **Performance** — e.g. p99 response time, throughput targets
- **Scalability** — expected load, growth projections
- **Availability / Reliability** — uptime SLA, failover expectations
- **Security** — auth model, data classification, compliance requirements (e.g. GDPR, SOC 2)
- **Maintainability** — code coverage minimums, linting standards, documentation requirements
- **Observability** — logging, metrics, tracing expectations
- **Portability / Deployment** — target environments (cloud, on-prem, containerised)

> **RULE**: A PRD must be written, reviewed, and confirmed before entity design begins. No entity tables, API specs, or code are produced until the PRD is approved.

> **LIVING DOCUMENT**: `docs/PRD.md` must be kept up to date throughout the project. Whenever requirements change, new user stories emerge, scope is added or cut, or open questions are resolved, update the PRD immediately — do not defer. If a code or design decision contradicts the PRD, flag the conflict and update the PRD before proceeding.

## Technical Design Document

After the PRD is confirmed, capture all significant technical and architectural decisions in `docs/DESIGN.md`.

### DESIGN.md must include

| Section | Content |
|---|---|
| **Architecture Overview** | High-level diagram or description of system components and interactions |
| **Technology Choices** | See table format below — each choice explicitly justified against one or more NFRs |
| **API Design Decisions** | Versioning strategy, auth approach, pagination style, error format |
| **Data Model Summary** | Key entities, relationships, and any notable persistence decisions |
| **Security Design** | Auth/authz flow, secret management, data-at-rest and in-transit decisions |
| **Observability Design** | Logging strategy, metrics, tracing, alerting approach |
| **Deployment Architecture** | Target environment, containerisation, CI/CD pipeline overview |
| **Rejected Alternatives** | What was considered and why it was ruled out |
| **Open Design Questions** | Decisions still to be made before implementation |

### NFR traceability table format

For every **critical NFR** defined in the PRD, record how it is addressed by one or more technology or design choices:

| NFR | Category | Technology / Design Choice | Rationale |
|---|---|---|---|
| e.g. p99 response < 200ms | Performance | Connection pooling (HikariCP), indexed queries | Eliminates connection overhead; index on hot query paths |
| e.g. 99.9% uptime SLA | Reliability | PostgreSQL with replication, health-check endpoints | Failover support; readiness probe enables zero-downtime deploys |
| e.g. GDPR compliance | Security | JWT (jjwt), TLS in transit, encrypted PII columns | Stateless auth; data encrypted at rest and in transit |
| e.g. HIPAA compliance | Security | AES-256 encryption at rest, TLS 1.2+ in transit, audit log table, role-based access control | PHI must be encrypted, access logged, and access-controlled per HIPAA Security Rule (45 CFR §164) |
| e.g. PCI-DSS compliance | Security | Tokenisation (no raw card storage), TLS 1.2+, network segmentation, WAF | Card data must never be stored or transmitted in the clear; scope reduction via tokenisation limits PCI audit surface |
| e.g. SOC 2 Type II compliance | Security / Observability | Centralised structured logging (e.g. ELK / CloudWatch), immutable audit trail, alerting on anomalous access | SOC 2 Trust Criteria require evidence of continuous monitoring, availability, and access control over a defined audit period |
| e.g. CCPA compliance (California) | Security | Consent management, data deletion API endpoint, PII inventory, encrypted PII columns | CCPA grants California residents the right to know, delete, and opt out of sale of personal information; deletion endpoint fulfils right-to-delete obligation |

Every critical NFR from the PRD must have at least one row. Technology choices that do not address any NFR do not need to appear here — requirements drive technology selection, not the other way around.

> **RULE**: `docs/DESIGN.md` must be written and confirmed before entity design or API spec work begins. Technology choices made informally during earlier discussions must be backfilled here before code is generated.

> **LIVING DOCUMENT**: `docs/DESIGN.md` must be kept up to date throughout the project. Whenever a new technology is adopted, an architectural decision is made or revised, a rejected alternative is reconsidered, or an NFR traceability mapping changes, update DESIGN.md immediately — do not defer. If a code change implies an architectural decision not yet recorded, add it to DESIGN.md as part of the same change.

---

## Java / Spring Boot Projects

Always ask about the following before generating any code or scaffold:

| Question | Options |
|----------|---------|
| Build tool | Maven / Gradle |
| Database | PostgreSQL / MySQL |
| Auth | JWT / Basic auth / Skip |
| Entities | Basic / Extended |
| Output | Working code / Docs only |

**API Versioning**: Always use **URL path prefix** versioning (e.g. `/api/v1/meetings`). All controllers must be rooted at `/api/v1`. When a v2 is needed, add a new controller package alongside v1 — never modify existing versioned routes.

**Order of work — always follow this sequence:**
1. **Write and confirm PRD** — capture problem statement, goals, user stories, functional requirements, NFRs, and constraints in `docs/PRD.md`; wait for approval before proceeding
2. **Write and confirm Technical Design** — record technology choices (each mapped to at least one NFR), architecture overview, and API design decisions in `docs/DESIGN.md`; wait for approval before proceeding
3. **Discuss and confirm entities** — propose entity names, key fields, and relationships as a table; wait for approval before writing any spec or code
4. Write and confirm `openapi.yaml` spec (built from the confirmed entities; all paths prefixed with `/api/v1`)
5. Generate scaffold (entities, repositories, controller interfaces) from the spec
6. Write service interfaces
7. Add implementations when explicitly asked
8. Add unit tests
9. Add integration tests when explicitly asked

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


