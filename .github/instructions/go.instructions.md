---
description: "Use when writing Go projects. Covers scaffold requirements, project layout, testing with testify, linting with golangci-lint, oapi-codegen setup, and Testcontainers integration tests."
applyTo: "**/*.go, **/go.mod"
---

## Go Projects

Always ask about the following before generating any code or scaffold:

| Question | Options |
|----------|---------|
| HTTP framework | `net/http` (stdlib) / Chi / Gin / Echo |
| Database | PostgreSQL / MySQL |
| ORM / query layer | `sqlx` (preferred) / GORM / raw `database/sql` |
| Auth | JWT / Basic auth / Skip |
| Entities | Basic / Extended |
| Output | Working code / Docs only |

**API Versioning**: Always use **URL path prefix** versioning (e.g. `/api/v1/meetings`). All routes must be rooted at `/api/v1`. When a v2 is needed, register a new route group alongside v1 — never modify existing versioned routes.

**Entity-first rule**: Entity design comes before the API spec. Never define request/response schemas in `openapi.yaml` until the entities have been reviewed and confirmed.

**Default**: Always start with handler interfaces / route stubs, then add implementations when explicitly asked.

## Scaffold Requirements for Go Projects

**BEFORE generating any code, verify that the scaffold includes ALL of the following:**

### `go.mod` / project root must include
- ✅ A `go.mod` at the repo root with the correct module path and Go version (default: latest stable)
- ✅ HTTP router dependency (Chi preferred: `github.com/go-chi/chi/v5`)
- ✅ `sqlx` (`github.com/jmoiron/sqlx`) + PostgreSQL driver (`github.com/lib/pq` or `github.com/jackc/pgx/v5`)
- ✅ JWT library (`github.com/golang-jwt/jwt/v5`)
- ✅ `golang-migrate` for DB migrations (`github.com/golang-migrate/migrate/v4`)
- ✅ `oapi-codegen` for OpenAPI-driven code generation
- ✅ `testify` for assertions and mocking (`github.com/stretchr/testify`)
- ✅ `golangci-lint` wired into CI / `Makefile` (not a `go.mod` dep — installed separately)

### Config files / tooling that must exist
- ✅ `Makefile` — with targets: `build`, `test`, `lint`, `migrate-up`, `migrate-down`, `generate`
- ✅ `.golangci.yml` — linter config (see rules below)
- ✅ `internal/config/` — app config loaded from environment variables (never hardcoded)
- ✅ `db/migrations/` — all SQL migration files (`V1__create_tables.sql` etc.)
- ✅ `.gitignore` — standard Go ignores (`/bin`, `/vendor`, `*.env`)
- ✅ `cmd/<app-name>/main.go` — entry point; wire dependencies here, keep it thin

### Project layout convention
Follow the standard Go project layout:
```
cmd/<app-name>/       # main package — entry point only
internal/
  api/                # HTTP handlers, middleware, router setup
    generated/        # oapi-codegen output (not committed)
  domain/             # entity types (pure structs, no framework deps)
  repository/         # DB access layer (interfaces + implementations)
  service/            # business logic (interfaces + implementations)
  config/             # env-based config structs
  mocks/              # mockery-generated mocks (not committed)
db/
  migrations/         # SQL migration files
build/                # compiled binaries (gitignored)
```

### Generated code (from `oapi-codegen`)
- ✅ Server interfaces and request/response types generated into `internal/api/generated/`
- ✅ Do **NOT** manually create handler types — always use `oapi-codegen`
- ✅ Wire generation into `go generate` (add `//go:generate` directive) and the `make generate` target

---

**Pre-generation checklist**: Before outputting any scaffold code, present this checklist to the user and confirm all items will be included.

### Default dependencies
Always include in every Go project:
- **`testify`** — assertions (`assert`, `require`) and mock generation via `mockery`
- **`slog`** (stdlib, Go 1.21+) — structured logging; no third-party logger unless user requests one
- **`envconfig`** or `os.Getenv` — read configuration from environment; never from config files committed to the repo

## Testing & Code Coverage

For all Go projects:
- Generate unit tests using the stdlib `testing` package + `testify/assert`
- Run with `go test ./... -race -coverprofile=coverage.out`
- Enforce a minimum **80% statement coverage** — `make test` fails if below threshold using `go tool cover`
- Table-driven tests are the default pattern — avoid one-assertion-per-function style
- Mock interfaces using `mockery` (`github.com/vektra/mockery/v2`); generated mocks live in `internal/mocks/`

## Linting & Static Analysis

For all Go projects, always configure:
- **`golangci-lint`** — enable at minimum: `errcheck`, `govet`, `staticcheck`, `gosimple`, `unused`, `gofmt`, `goimports`, `revive`
- **`go vet`** — always runs as part of `make lint`
- Lint runs as part of CI and `make lint`; build fails on any lint error

### `.golangci.yml` rules
- Enable `errcheck` — all returned errors must be handled; no silent discards
- Enable `gocritic` — catches common code quality issues
- Disable `exhaustruct` at project start — too noisy before all fields are in use
- Exclude generated files (`internal/api/generated/`) from all linters

## OpenAPI / API Design

> **RULE: The OpenAPI spec MUST be written and reviewed before any handler, service, model, or scaffold code is generated. No exceptions.**

For all Go projects:
- **Spec-first** — write `docs/openapi.yaml` before any handler code
- Use **`oapi-codegen`** to generate Chi-compatible server interfaces and type-safe request/response models
- Handlers implement the generated `ServerInterface` — spec is always the source of truth
- Generated sources go to `internal/api/generated/` (not committed)
- Add a `//go:generate` directive and a `make generate` target so generation runs before build

### Spec structure
- Place spec at `docs/openapi.yaml`
- Use `$ref` to split large specs into components (`schemas`, `paths`, `responses`)
- Always define `components/schemas` for all request/response bodies — no inline schemas

## Integration Tests

When asked to generate integration tests:
- Use **Testcontainers for Go** (`github.com/testcontainers/testcontainers-go`) with a `postgres` container
- Integration tests live in `internal/integration/` package, separate from unit tests
- Always create a shared `TestMain` or helper that starts/stops the container once per test run
- Integration tests run via a separate `make integration-test` target gated by the `//go:build integration` build tag

## Database — Primary Key

- Use `github.com/google/uuid` v1.6+ or `github.com/gofrs/uuid/v5` for UUID v7 generation
- Never use `uuid.New()` as a primary key
- Default to **UUID v7** unless the user explicitly chooses BIGSERIAL
