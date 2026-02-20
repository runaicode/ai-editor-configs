# GitHub Copilot Custom Instructions

## Project Context

This is a Go microservices application using:
- Go 1.23
- gRPC + Protocol Buffers
- PostgreSQL via pgx (not GORM)
- Redis for caching
- Docker + Kubernetes for deployment
- OpenTelemetry for observability

## Code Generation Rules

### Go Patterns
- Always handle errors explicitly. Never use `_` to discard errors.
- Use `context.Context` as the first parameter for all public functions that do I/O.
- Return `error` as the last return value. Use `fmt.Errorf("operation: %w", err)` for wrapping.
- Use struct embedding sparingly. Prefer composition with explicit fields.
- Interfaces should be small (1-3 methods) and defined where they're used, not where they're implemented.
- Use table-driven tests with `t.Run()` for subtests.
- Never use `init()` functions. Initialize explicitly in `main()` or constructors.

### Project Structure
```
cmd/
├── api-server/main.go        # HTTP/gRPC server entry point
├── worker/main.go             # Background worker entry point
internal/
├── domain/                    # Business logic, no external dependencies
│   ├── user/
│   │   ├── service.go         # Business logic
│   │   ├── repository.go      # Interface definition
│   │   └── models.go          # Domain models
├── infra/                     # External adapters
│   ├── postgres/              # PostgreSQL implementations
│   ├── redis/                 # Redis implementations
│   └── grpc/                  # gRPC server/client
├── config/                    # Configuration loading
pkg/                           # Shared utilities (keep minimal)
proto/                         # Protobuf definitions
```

### Naming Conventions
```go
// Packages: short, lowercase, no underscores
package userservice  // not user_service

// Files: snake_case
user_handler.go      // not userHandler.go

// Exported: PascalCase (public)
func NewUserService() {}
type UserRepository interface {}

// Unexported: camelCase (private)
func validateEmail() {}
var defaultTimeout time.Duration

// Acronyms: consistent casing
type HTTPClient struct{}  // not HttpClient
func GetUserID() {}       // not GetUserId
```

### Error Handling
```go
// Define domain errors in the domain package
var (
    ErrUserNotFound    = errors.New("user not found")
    ErrEmailTaken      = errors.New("email already in use")
    ErrInvalidInput    = errors.New("invalid input")
)

// Wrap errors with context
if err := repo.Save(ctx, user); err != nil {
    return fmt.Errorf("saving user %s: %w", user.ID, err)
}

// Check errors with errors.Is/As, not string comparison
if errors.Is(err, ErrUserNotFound) {
    // handle not found
}
```

### Database
- Use `pgx` directly. No ORM.
- Parameterized queries only: `$1, $2` — never string concatenation.
- Use transactions for multi-step operations.
- Close rows in defer: `defer rows.Close()`.
- Connection pooling via `pgxpool`.

### Testing
- Test files: `*_test.go` in the same package.
- Use `testify/assert` and `testify/require`.
- Table-driven tests for multiple cases.
- Use `t.Helper()` in test helper functions.
- Mock interfaces with `mockery` or hand-written mocks.
- Integration tests use `testcontainers-go` for real databases.

### Security
- No secrets in code. Use environment variables loaded in `config/`.
- Validate all gRPC/HTTP input with explicit checks.
- Use `crypto/rand` for random values, never `math/rand`.
- SQL: parameterized queries only.
- Log: never log passwords, tokens, or PII.

### Performance
- Use `sync.Pool` for frequently allocated objects.
- Prefer `strings.Builder` for string concatenation in loops.
- Use `context.WithTimeout` for all external calls.
- Profile with `pprof` before optimizing.

## What Copilot Should NOT Generate

- `panic()` calls outside of `main()` or `init()`.
- Global mutable state (package-level `var` that gets modified).
- `interface{}` or `any` when a specific type is possible.
- Mutex usage without documenting what it protects.
- `time.Sleep()` in production code (use tickers or timers).
- Magic numbers — define named constants.
