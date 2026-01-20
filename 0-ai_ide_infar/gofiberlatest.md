# Architectural Mastery in Go Fiber: The "100" Series Guide

## Table of Contents: The 100 Parts

### Section 1: Core Fundamentals
- [Part 1: Getting Started & Architecture](#part-1-getting-started--architecture)
- [Part 2: Routing Strategy](#part-2-routing-strategy)
- [Part 3: v3 Ecosystem](#part-3-v3-ecosystem)
- [Part 4: Expert Patterns](#part-4-expert-patterns)
- [Part 5: Operational Excellence](#part-5-operational-excellence)
- [Part 6: Zero Allocation Config](#part-6-zero-allocation-config)
- [Part 7: Middleware Strategy](#part-7-middleware-strategy)
- [Part 8: Error Handling (Global)](#part-8-error-handling-global)
- [Part 9: Validation (Struct Validator)](#part-9-validation-struct-validator)
- [Part 10: Testing Strategies](#part-10-testing-strategies-unitintegration)

### Section 2: Developer Experience & Data
- [Part 11: Dependency Injection (Uber fx)](#part-11-dependency-injection-uber-fx)
- [Part 12: Configuration Management (Viper)](#part-12-configuration-management-viper)
- [Part 13: Database Migrations](#part-13-database-migrations-golang-migrate)
- [Part 14: ORM Integration (GORM)](#part-14-orm-integration-gorm)
- [Part 15: SQL Builder (Sqlc)](#part-15-sql-builder-sqlc)
- [Part 16: NoSQL (MongoDB)](#part-16-nosql-mongodb)
- [Part 17: Caching Strategies (Redis)](#part-17-caching-strategies-redis)
- [Part 18: Task Queues (Asynq)](#part-18-task-queues-asynq)
- [Part 19: File Uploads (S3/MinIO)](#part-19-file-uploads-s3minio)
- [Part 20: Email Sending (SMTP/SendGrid)](#part-20-email-sending-smtpsendgrid)
- [Part 21: Authentication (JWT/Oauth2)](#part-21-authentication-jwtoauth2)
- [Part 22: Authorization (Casbin RBAC)](#part-22-authorization-casbin-rbac)
- [Part 23: API Documentation (Swagger)](#part-23-api-documentation-swaggeropenapi)
- [Part 24: Real-time (WebSockets)](#part-24-real-time-websockets)
- [Part 25: GraphQL (Gqlgen)](#part-25-graphql-gqlgen)
- [Part 26: gRPC Integration](#part-26-grpc-integration)
- [Part 27: Logging (Zerolog)](#part-27-logging-zerologzap)
- [Part 28: Metrics (Prometheus)](#part-28-metrics-prometheusgrafana)

### Section 3: Cloud Native & Security
- [Part 29: Cloud Auth (Cognito/Firebase)](#part-29-cloud-auth-cognitofirebase)
- [Part 30: Feature Flags](#part-30-feature-flags)
- [Part 31: Rate Limiting (Distributed)](#part-31-rate-limiting-distributed)
- [Part 32: Bot Protection](#part-32-bot-protection)
- [Part 33: WebAuthn (Passkeys)](#part-33-webauthn-passkeys)
- [Part 34: Mutual TLS (mTLS)](#part-34-mutual-tls-mtls)
- [Part 35: Secret Management (Vault)](#part-35-secret-management-vault)
- [Part 36: Audit Logging](#part-36-audit-logging)
- [Part 37: Resilience (Circuit Breaker)](#part-37-resilience-circuit-breaker)
- [Part 38: Retries & Backoff](#part-38-retries--backoff)
- [Part 39: Timeout Management](#part-39-timeout-management)
- [Part 40: Distributed Tracing (Otel)](#part-40-distributed-tracing-opentelemetry)
- [Part 41: Kubernetes Probes](#part-41-kubernetes-probes)
- [Part 42: Graceful Shutdown](#part-42-graceful-shutdown)
- [Part 43: Docker Optimization](#part-43-docker-optimization)
- [Part 44: CI/CD Pipelines](#part-44-cicd-pipelines)
- [Part 45: Data Processing (ETL)](#part-45-data-processing-etl)
- [Part 46: Batch Jobs](#part-46-batch-jobs)
- [Part 47: Stream Processing (Kafka)](#part-47-stream-processing-kafka)
- [Part 48: Event Sourcing](#part-48-event-sourcing)
- [Part 49: Serverless (Lambda)](#part-49-serverless-lambda)
- [Part 50: Edge Computing (Wasm)](#part-50-edge-computing-wasm)
- [Part 51: Multi-Tenancy (SaaS)](#part-51-multi-tenancy-saas)
- [Part 52: Internationalization (i18n)](#part-52-internationalization-i18n)

### Section 4: Enterprise & Architecture
- [Part 53: Clean Architecture](#part-53-clean-architecture)
- [Part 54: Domain-Driven Design (DDD)](#part-54-domain-driven-design-ddd)
- [Part 55: Hexagonal Architecture](#part-55-hexagonal-architecture)
- [Part 56: Microservices Patterns](#part-56-microservices-patterns)
- [Part 57: Architectural Mastery](#part-57-architectural-mastery)
- [Part 58: Hexagonal Architecture (Ports)](#part-58-hexagonal-architecture-ports--adapters)
- [Part 59: Property-Based Testing](#part-59-property-based-testing)
- [Part 60: Contract Testing (Pact)](#part-60-contract-testing-pact)
- [Part 61: AI Integration (LLMs)](#part-61-ai-integration-llms)
- [Part 62: Network Performance Tuning](#part-62-network-performance-tuning)
- [Part 63: GPU/Compute Offloading](#part-63-gpucompute-offloading)
- [Part 64: Distributed Rate Limiting](#part-64-distributed-rate-limiting-lua)
- [Part 65: Zero-Trust Security (mTLS)](#part-65-zero-trust-security-mtls)
- [Part 66: Real-Time Updates (SSE)](#part-66-real-time-updates-sse)
- [Part 67: GraphQL Subscriptions](#part-67-graphql-subscriptions)
- [Part 68: Decentralized Storage (IPFS)](#part-68-decentralized-storage-ipfs)
- [Part 69: Security Hardening (Helmet)](#part-69-security-hardening-helmet)
- [Part 70: Interactive CLI Tools](#part-70-interactive-cli-tools)
- [Part 71: Audit Logging](#part-71-audit-logging)
- [Part 72: Zero-Downtime Shutdown](#part-72-zero-downtime-shutdown)
- [Part 73: Distributed Transactions (Saga)](#part-73-distributed-transactions-saga)
- [Part 74: Profiling & Optimization](#part-74-profiling--optimization-pprof)
- [Part 75: Benchmarking](#part-75-high-performance-benchmarking)
- [Part 76: Middleware Composition](#part-76-advanced-middleware-composition)

### Section 5: The Frontier
- [Part 77: gRPC Gateway](#part-77-grpc-gateway)
- [Part 78: Secure Webhooks (HMAC)](#part-78-secure-webhooks-hmac)
- [Part 79: Localization (i18n)](#part-79-localization-i18n)
- [Part 80: Cloud Storage Patterns](#part-80-cloud-storage-patterns-s3-presigned)
- [Part 81: Production Deployment (Systemd)](#part-81-production-deployment-systemd)
- [Part 82: Docker Optimization](#part-82-docker-optimization)
- [Part 83: Horizontal Scaling (Nginx)](#part-83-horizontal-scaling-nginx)
- [Part 84: IaC (Ansible)](#part-84-infrastructure-as-code-ansible)
- [Part 85: Kubernetes (K8s)](#part-85-kubernetes-k8s)
- [Part 86: Serverless (AWS Lambda)](#part-86-serverless-aws-lambda)
- [Part 87: Cross-Platform Compilation](#part-87-cross-platform-compilation)
- [Part 88: Desktop Apps (FiberWebGUI)](#part-88-desktop-apps-fiberwebgui)
- [Part 89: Real-Time Analytics (ClickHouse)](#part-89-real-time-analytics-clickhouse)
- [Part 90: Full-Text Search (Elasticsearch)](#part-90-full-text-search-elasticsearch)
- [Part 91: Graph Databases (Neo4j)](#part-91-graph-databases-neo4j)
- [Part 92: Event Streaming (NATS)](#part-92-event-streaming-nats-jetstream)
- [Part 93: Distributed Configuration (Etcd)](#part-93-distributed-configuration-etcd)
- [Part 94: Service Discovery (Consul)](#part-94-service-discovery-consul)
- [Part 95: Secrets Management (Vault)](#part-95-secrets-management-vault)
- [Part 96: Distributed Tracing (Otel)](#part-96-distributed-tracing-opentelemetry)
- [Part 97: WebAssembly (Wasm)](#part-97-webassemblywithfiber-client-side)
- [Part 98: Advanced Generics](#part-98-advanced-generics-go-118)
- [Part 99: The Zen of Fiber](#part-99-the-zen-of-fiber-best-practices)
- [Part 100: The Future (HTTP/3)](#part-100-the-future-http3--quic)

### Section 6: Go Concurrency Patterns (Fundamentals)
- [Part 101: Goroutines in Handlers](#part-101-goroutines-in-handlers)
- [Part 102: Channels & Worker Pools](#part-102-channels--worker-pools)
- [Part 103: Mutex & Sync](#part-103-mutex--sync)
- [Part 104: The Scheduler (Ticker)](#part-104-the-scheduler-tickercron)
- [Part 105: WaitGroups](#part-105-waitgroups)

### Section 7: Real-World Blueprints
- [Part 106: Blueprint: E-Commerce API](#part-106-blueprint-e-commerce-api)
- [Part 107: Blueprint: Real-Time Chat](#part-107-blueprint-real-time-chat)
- [Part 108: Blueprint: URL Shortener](#part-108-blueprint-url-shortener)

### Section 8: Troubleshooting & Migration
- [Part 109: Common Errors & Fixes](#part-109-common-errors--fixes)
- [Part 110: Debugging Performance (pprof)](#part-110-debugging-performance-pprof)
- [Part 111: Migration Guide (v2 -> v3)](#part-111-migration-guide-v2---v3)
- [Part 112: The FAQ](#part-112-the-faq)

### Section 9: LLM & AI Context
- [Part 113: Role-Playing Prompts (for LLMs)](#part-113-role-playing-prompts-for-llms)
- [Part 114: RAG Keywords & Tags](#part-114-rag-keywords--tags)

---

> **Version:** 3.x (Beta/Candidate)
> **Status:** Definitive Guide for Senior Engineers
> **Updated:** Jan 2026

---

## Part 1: Getting Started & Architecture

### 1. Installation & Setup
v3 requires Go 1.25+.

```bash
go get github.com/gofiber/fiber/v3
```

**Hello World (v3 Style):**
```go
package main

import "github.com/gofiber/fiber/v3"

func main() {
    app := fiber.New()

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })

    app.Listen(":3000")
}
```

### 1. Introduction: The High-Performance Landscape
The evolution of web development within the Go ecosystem has historically been bifurcated into two distinct philosophies: the standard library-compliant approach and the performance-maximalist approach. While the standard `net/http` library offers robust compatibility, it was not originally designed for the extreme throughput and low-latency requirements of high-frequency trading platforms or real-time telecommunications.

**Fiber** emerged as the preeminent framework built atop `fasthttp`, designed to bridge the gap between raw performance and developer ergonomics. Inspired by Express.js, it provides a familiar developer experience while leveraging Go’s compilation benefits.

As of 2025/2026, Fiber v3 represents a structural realignment with Go idioms, introducing interface-based contexts and unified binding while maintaining its zero-allocation philosophy.

### 2. The Philosophy of Zero Allocation
The central tenet of Fiber’s architecture is "zero memory allocation" in the hot path.
- **Mechanism:** Fiber utilizes `sync.Pool` to recycle request context objects. When a request arrives, a context is retrieved; when the response is sent, it is reset and returned.
- **The Trade-off:** Mutability. `c.Params("id")` returns a string that points to a shared byte buffer. It **will be overwritten** by the next request.

**Antigravity Best Practice:**
If you need to pass data to a goroutine or store it, **you must copy it**.

```go
// ❌ UNSAFE: Data Race
go func() {
    log.Println(c.Params("user")) // Might print garbage from the next request
}()

// ✅ SAFE: Immutable Copy
user := utils.CopyString(c.Params("user"))
go func() {
    log.Println(user)
}()
```

### 3. Clean Architecture Implementation
To build maintainable systems, avoid placing business logic in handlers.

#### 3.1 The Dependency Rule
- **Entities (Domain):** Pure Go structs (e.g., `User`). No framework dependencies.
- **Use Cases (Service):** Application business rules. Dependencies are interfaces.
- **Interface Adapters (Handlers):** exclusively translate HTTP -> DTO -> Service.
- **Infrastructure:** Database drivers, Fiber app initialization.

#### 3.2 Dependency Injection
For high-performance systems, favor Manual Injection ("Pure DI") or Google Wire over reflection-based frameworks to maintain type safety and startup speed.

```go
func main() {
    // 1. Infrastructure
    db := postgres.Connect()
    
    // 2. Repository
    userRepo := repository.NewUserRepo(db)
    
    // 3. Service
    userService := service.NewUserService(userRepo)
    
    // 4. Handler
    userHandler := handlers.NewUserHandler(userService)
    
    // 5. Fiber
    app := fiber.New()
    app.Post("/users", userHandler.Register)
}
```

---

## Part 2: Go Fiber v3 Technical Reference

### 5. What's New in v3?

#### 5.1 `fiber.Ctx` is now an Interface
The move from `*fiber.Ctx` (struct pointer) to `fiber.Ctx` (interface) facilitates easier mocking and standard library integration.
- **Migration:** Update all handler signatures from `func(c *fiber.Ctx) error` to `func(c fiber.Ctx) error`.

#### 5.2 Unified Binding
Data binding has been consolidated into a fluent API.
- **Old (v2):** `c.BodyParser(&out)`
- **New (v3):** `c.Bind().Body(&out)`

The `Bind()` method supports:
- `c.Bind().Query(&out)`
- `c.Bind().Header(&out)`
- `c.Bind().URI(&out)`

#### 5.3 Route Chaining
Inspired by Express, you can now chain handlers for a specific route path.

```go
app.RouteChain("/api/user")
    .Get(func(c fiber.Ctx) error { return c.SendString("Get User") })
    .Post(func(c fiber.Ctx) error { return c.SendString("Create User") })
    .Delete(func(c fiber.Ctx) error { return c.SendString("Delete User") })
```

#### 5.4 Global State
v3 introduces a thread-safe global state container.

```go
// Store
c.App().SetState("config", myConfig)

// Retrieve
config := c.App().GetState("config").(*MyConfig)
```

### 6. Performance Engineering

#### 6.1 JSON Serialization
Replace standard `encoding/json` with **Sonic** or **Goccy/Go-JSON** for 2-3x throughput improvements.

```go
import "github.com/bytedance/sonic"

app := fiber.New(fiber.Config{
    JSONEncoder: sonic.Marshal,
    JSONDecoder: sonic.Unmarshal,
})
```

#### 6.2 Pre-forking
Enable `Prefork: true` to utilize `SO_REUSEPORT`, allowing multiple processes to bind to port 3000.
**Docker Note:** If using prefork in Docker, you **must** use `dumb-init` or `tini` as the entrypoint to handle PID 1 signal propagation and zombie reaping.

### 7. Security Engineering

#### 7.1 Middleware Stack
- **Helmet:** `app.Use(helmet.New())` (XSS, HSTS)
- **CORS:** explicit whitelist. Never use `*` with credentials.
- **CSRF:** Mandatory for session/cookie auth.

#### 7.2 Secure Cookies
v3 automatically enforces `Secure: true` if `SameSite` is set to `None`, complying with modern browser standards.

### 8. Migration Checklist (v2 -> v3)

- [ ] **Context:** Refactor handlers to accept `fiber.Ctx` interface.
- [ ] **Routing:** Use `app.Use()` instead of `app.Mount()`.
- [ ] **TLS:** Use `app.Listen(":443", fiber.ListenConfig{CertFile: "...", KeyFile: "..."})` instead of dedicated `ListenTLS` methods.
- [ ] **Middleware:** Check `gofiber/fiber/v3/middleware` for updated packages. Many have moved to the core repo or changed import paths.
- [ ] **Locals:** Verify usage of `c.Locals`. Prioritize strong typing or context-based getters.

---

## Part 3: Deep Dive into the v3 Ecosystem

### 9. Application Lifecycle Hooks
Fiber v3 introduces a robust hooking system that allows you to execute code at specific points in the application lifecycle. This is critical for initialization logic, graceful shutdowns, and custom logging.

**Available Hooks:**
- `OnRoute`, `OnName`, `OnGroup`, `OnGroupName`, `OnListen`, `OnFork`
- **Shutdown Hooks (New):**
  - `OnPreShutdown`: Executed *before* the shutdown process begins.
  - `OnPostShutdown`: Executed *after* the server has stopped (receives error).
- **Startup Hooks (New):**
  - `OnPreStartupMessage`: Customize the startup banner.
  - `OnPostStartupMessage`: Run logic immediately after the banner prints.

**Crucial Pattern for Graceful Shutdown:**
When using hooks, `app.Listen` must be non-blocking (run in a goroutine) so that `app.Shutdown()` can actually be called.

```go
func main() {
    app := fiber.New()
    
    // Register Hooks
    app.Hooks().OnPreShutdown(func() error {
        log.Println("cleanup: closing db connections...")
        return nil
    })

    // Run in background
    go func() {
        if err := app.Listen(":3000"); err != nil {
            log.Panic(err)
        }
    }()

    // Wait for signal (implementation detail)
    waitForSignal() 
    app.Shutdown() // Triggers OnPreShutdown -> Stop Listener -> OnPostShutdown
}
```

### 10. The New Client Package
The `gofiber/fiber/v3/client` has been rebuilt from the ground up. It centralizes configuration while allowing per-request overrides.

**Key Features:**
- **Centralized Config:** set base URL, default headers, and timeouts once.
- **Fasthttp Integration:** `NewWithHostClient` allows reusing existing `fasthttp` clients (e.g., for specialized connection pooling).
- **Load Balancing:** `NewWithLBClient` for client-side load balancing.

**Example Usage:**

```go
import "github.com/gofiber/fiber/v3/client"

func main() {
    // Initialize Client
    cc := client.New().
        SetBaseURL("https://api.internal").
        SetTimeout(5 * time.Second).
        AddHeader("X-Service-ID", "payment-service")

    // Per-request override
    resp, err := cc.Get("/users/:id", client.Config{
        PathParam: map[string]string{
            "id": "42",
        },
        Header: map[string]string{
            "X-Request-ID": "123",
        },
    })
}
```

### 11. Storage Interface & `WithContext`
The `gofiber/storage` interface is used by middlewares like Session, Cache, and Idempotency. v3 adds context-aware methods to support timeouts and cancellation, which is essential for distributed stores (Redis, Memcached) in microservices.

**New Interface Signature:**
```go
type Storage interface {
    Get(key string) ([]byte, error)
    Set(key string, val []byte, exp time.Duration) error
    // ... other v2 methods

    // v3 Context Methods
    GetWithContext(ctx context.Context, key string) ([]byte, error)
    SetWithContext(ctx context.Context, key string, val []byte, exp time.Duration) error
    DeleteWithContext(ctx context.Context, key string) error
    ResetWithContext(ctx context.Context) error
}
```

**Why this matters:**
If your Redis call hangs, `GetWithContext` allows the operation to be cancelled when the HTTP request context times out, preventing a cascading failure of your API.

---

## Part 4: Expert Patterns & Ecosystem Deep Dive

### 12. Middleware Best Practices
In v3, the "Magic String" pattern for accessing middleware data (e.g., `c.Locals("user_id")`) is discouraged. The ecosystem has shifted to **type-safe context extractors**.

**The Old Way (v2):**
```go
userID := c.Locals("user_id").(string) // Runtime panic risk
```

**The New Way (v3):**
Middleware packages now export `FromContext` getters that retrieve data from the opaque `context.Context`.

```go
import "github.com/gofiber/fiber/v3/middleware/requestid"

func handler(c fiber.Ctx) error {
    // Type-safe retrieval
    rid := requestid.FromContext(c)
    return c.SendString("Request ID: " + rid)
}
```

**Custom Middleware Pattern:**
When writing your own middleware, define unexported context keys and exported getters.

```go
package myauth

type ctxKey struct{}

func New() fiber.Handler {
    return func(c fiber.Ctx) error {
        c.Locals(ctxKey{}, "secret-value") // Store using unique key type
        return c.Next()
    }
}

func FromContext(c fiber.Ctx) string {
    val, _ := c.Locals(ctxKey{}).(string)
    return val
}
```

### 13. Advanced Testing Strategies
The shift to `fiber.Ctx` as an interface allows for true unit testing without spinning up the router.

**Mocking the Context:**
You can generate mocks for `fiber.Ctx` using `mockery` or implement a stub for testing complex logic in isolation.

**Integration Testing with `app.Test`:**
The `Test` method signature has changed to support configuration objects, allowing precise control over timeouts.

```go
import "github.com/gofiber/fiber/v3"

func Test_Route(t *testing.T) {
    app := fiber.New()
    app.Get("/", func(c fiber.Ctx) error { return c.SendStatus(200) })

    req := httptest.NewRequest("GET", "/", nil)
    
    // v3 Test Config
    resp, err := app.Test(req, fiber.TestConfig{
        Timeout:       2 * time.Second,
        FailOnTimeout: true,
    })

    assert.Equal(t, 200, resp.StatusCode)
}
```

### 14. Production-Grade WebSockets
WebSockets in Fiber v3 are handled via the `contrib` package.

**Installation:**
```bash
go get github.com/gofiber/contrib/websocket
```

**Critical Pitfall: Middleware Conflicts**
If you use `middleware/cache` or `middleware/compress`, they can corrupt the WebSocket handshake. You **must** explicitly skip the WebSocket endpoint.

```go
app.Use(cache.New(cache.Config{
    Next: func(c fiber.Ctx) bool {
        // SKIP cache for WebSocket routes
        return strings.Contains(c.Path(), "/ws")
    },
}))

app.Get("/ws", websocket.New(func(c *websocket.Conn) {
    for {
        _, msg, err := c.ReadMessage()
        if err != nil { break }
        c.WriteMessage(websocket.TextMessage, msg)
    }
}))
```

**Accessing Context in WebSockets:**
The `*websocket.Conn` object is bridging the HTTP world.
- `c.Locals("key")` inside the websocket handler retrieves values set by HTTP middleware *before* the upgrade.
- `c.Params()`, `c.Query()` work as expected inside the callback.

---

## Part 5: Operational Excellence

### 15. Global Error Handling
In professional applications, never expose raw errors or stack traces to the client. v3 allows a centralized `ErrorHandler` that can distinguish between your domain errors and framework exceptions.

**The "Defense in Depth" Handler:**

```go
app := fiber.New(fiber.Config{
    ErrorHandler: func(c fiber.Ctx, err error) error {
        // Default to 500 Internal Server Error
        code := fiber.StatusInternalServerError
        
        // Check if it's a specific Fiber error (e.g. 404 Not Found, 405 Method Not Allowed)
        var e *fiber.Error
        if errors.As(err, &e) {
            code = e.Code
        }

        // Log the error securely (don't log sensitive data)
        log.Error("Request Failed", "path", c.Path(), "error", err)

        // Return a safe, sanitized JSON response
        return c.Status(code).JSON(fiber.Map{
            "status": "error",
            "message": "Something went wrong. Please try again later.",
            "request_id": requestid.FromContext(c), // Correlate with logs
        })
    },
})
```

### 16. Advanced Validation Pattern
v3's `Bind()` is powerful, but it doesn't validate business rules (like "email format" or "age > 18") by default. You should integrate `go-playground/validator`.

**1. Create a Custom Validator:**
```go
import "github.com/go-playground/validator/v10"

type StructValidator struct {
    validate *validator.Validate
}

func (v *StructValidator) Validate(out any) error {
    return v.validate.Struct(out)
}
```

**2. Register it in Config:**
```go
app := fiber.New(fiber.Config{
    StructValidator: &StructValidator{validate: validator.New()},
})
```

**3. Use it in Handlers:**
The `Bind().Body()` call will now automatically run the validator and return an error if the struct tags fail.

```go
type CreateUserRequest struct {
    Email string `json:"email" validate:"required,email"`
    Age   int    `json:"age"   validate:"gte=18"`
}

app.Post("/users", func(c fiber.Ctx) error {
    req := new(CreateUserRequest)
    
    // Parses JSON AND runs validator
    if err := c.Bind().Body(req); err != nil {
        return c.Status(400).SendString(err.Error())
    }
    
    return c.SendStatus(201)
})
```

### 17. The Adaptor Pattern (Net/HTTP Compatibility)
Migrating legacy code? v3 has first-class support for standard `net/http` handlers.

**Automatic Adaptation (New in v3):**
You can pass a standard `http.HandlerFunc` directly to Fiber's router. The underlying adapter handles the translation transparently.

```go
func legacyHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "I am a standard Go handler!")
}

// v3 automatically adapts this:
app.Get("/legacy", http.HandlerFunc(legacyHandler))
```

**Using Net/HTTP Middleware:**
To use a standard middleware (like a security scanner or an auth proxy written for net/http):

```go
import "github.com/gofiber/fiber/v3/middleware/adaptor"

func main() {
    app := fiber.New()
    
    // Wrap standard middleware
    app.Use(adaptor.HTTPMiddleware(legacyLoggerMiddleware))
}
```

---

## Part 6: Infrastructure, Observability & Deployment

### 18. Structured Logging & Tracing (OpenTelemetry)
For production-grade observability, simple text logging is insufficient. Use `otelfiber` to automatically push traces to Jaeger/Tempos.

**Integration:**
```go
import "github.com/gofiber/contrib/otelfiber/v3"

// Initialize basic tracer provider first (omitted for brevity)
tp := initTracer() 

app := fiber.New()

// Middleware must be first
app.Use(otelfiber.New(otelfiber.Config{
    TracerProvider: tp,
    SpanNameFormatter: func(c fiber.Ctx) string {
        return fmt.Sprintf("%s %s", c.Method(), c.Route().Path)
    },
}))
```

### 19. Robust Database Layer (pgxpool)
Do not use the standard `database/sql` if you want maximum performance with PostgreSQL. Use `jackc/pgx`.

**Connection Multiplexing Pattern:**
Always create a single pool and inject it.

```go
func InitDB() *pgxpool.Pool {
    config, _ := pgxpool.ParseConfig(os.Getenv("DATABASE_URL"))
    config.MaxConns = 100
    config.MinConns = 10
    
    pool, _ := pgxpool.NewWithConfig(context.Background(), config)
    return pool
}

// Usage in Handler with v3 Context
func GetUser(c fiber.Ctx) error {
    // Pass Fiber's UserContext() which tracks request cancellation
    row := db.QueryRow(c.UserContext(), "SELECT id FROM users WHERE id=$1", id)
}
```

### 20. Production Dockerfile (Distroless)
For the smallest, most secure footprint, use Google's `distroless` images. They contain no shell, reducing attack surface.

```dockerfile
# Stage 1: Build
FROM golang:1.24-alpine AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
COPY . .
# CGO_ENABLED=0 is critical for distroless
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o server ./cmd/api

# Stage 2: Runtime
FROM gcr.io/distroless/static-debian12
COPY --from=builder /build/server /server
# Run as non-root user (distroless uses 65532:65532)
USER nonroot:nonroot
CMD ["/server"]
```

---

## Part 7: The The Middleware Catalog (Comprehensive)

Fiber v3 has a rich ecosystem of middleware. Here is the definitive configuration catalog for the most critical ones.

### 21. Standard & Contrib Middlewares

#### Monitor (`middleware/monitor`)
Exposes a metrics dashboard. **Do not expose to public!**

```go
app.Get("/metrics", monitor.New(monitor.Config{
    Title:   "System Status",
    Refresh: 5 * time.Second,
    APIOnly: false, // Set true if you only want JSON
}))
```

#### Limiter (`middleware/limiter`)
Rate limiting to prevent abuse. Supports distributed storage (Redis).

```go
app.Use(limiter.New(limiter.Config{
    Max:        100,
    Expiration: 1 * time.Minute,
    KeyGenerator: func(c fiber.Ctx) string {
        return c.IP() // or c.Get("X-Forwarded-For")
    },
    LimitReached: func(c fiber.Ctx) error {
        return c.Status(429).SendString("Slow down!")
    },
}))
```

#### Cache (`middleware/cache`)
Cache responses mapped to specific keys.

```go
app.Use(cache.New(cache.Config{
    Expiration: 30 * time.Minute,
    KeyGenerator: func(c fiber.Ctx) string {
        return c.Path() + "?" + string(c.Request().URI().QueryString())
    },
    // Dynamically set expiration via header
    ExpirationGenerator: func(c fiber.Ctx, cfg *cache.Config) time.Duration {
        if ttl := c.GetRespHeader("Cache-Time"); ttl != "" {
            d, _ := time.ParseDuration(ttl + "s")
            return d
        }
        return cfg.Expiration
    },
}))
```

#### Timeout (`middleware/timeout`)
Wraps a handler with a deadline.

```go
// Usage: Wraps the handler, not applied globally usually
app.Get("/long-op", timeout.New(handlerFunc, timeout.Config{
    Timeout: 5 * time.Second,
    TimeoutHandler: func(c fiber.Ctx) error {
        return c.Status(504).SendString("Operation timed out")
    },
}))
```

#### RequestID (`middleware/requestid`)
Injects a unique ID into every request.

```go
app.Use(requestid.New(requestid.Config{
    Header: "X-Trace-ID",
    Generator: func() string {
        return uuid.New().String()
    },
}))
```

---

## Part 8: The Complete API Reference

### 22. App Config (`fiber.Config`)
When initializing `fiber.New()`, you have granular control.

| Option | Description | Recommended (Prod) |
| :--- | :--- | :--- |
| `Prefork` | Spawns multiple processes on same port | `true` (on Linux) |
| `ServerHeader` | Value of `Server` header | "Fiber" or Hidden |
| `BodyLimit` | Max request body size | `4 * 1024 * 1024` (4MB) |
| `ReadTimeout` | Max time to read request | `5 * time.Second` |
| `WriteTimeout` | Max time to write response | `10 * time.Second` |
| `IdleTimeout` | Max Keep-Alive idle time | `60 * time.Second` |
| `Concurrency` | Max concurrent connections | `256 * 1024` |
| `DisableStartupMessage` | Hide banner | `true` |
| `StrictRouting` | `/foo` != `/foo/` | `true` |
| `CaseSensitive` | `/Foo` != `/foo` | `true` |
| `JSONEncoder` | Custom JSON marshaller | `sonic.Marshal` |
| `JSONDecoder` | Custom JSON unmarshaller | `sonic.Unmarshal` |

### 23. The `fiber.Ctx` Interface (v3)

The `ctx` object is your primary interaction point. It implements `context.Context` (sort of) and provides helper methods.

**Core Methods:**
- `c.Body() []byte`: Raw body.
- `c.Params(key string) string`: Route parameter (immutable copy).
- `c.Query(key string) string`: Query string parameter.
- `c.Get(key string) string`: Request header.
- `c.Set(key, val string)`: Response header.
- `c.Status(code int) fiber.Ctx`: Set status code.
- `c.SendString(s string) error`: Write string response.
- `c.JSON(data any) error`: Write JSON response.
- `c.Next() error`: Pass to next middleware.

**Advanced Methods:**
- `c.Locals(key, val any) any`: Request-scoped storage.
- `c.UserContext() context.Context`: Context tied to request lifecycle (for DB calls).
- `c.SetUserContext(ctx context.Context)`: Override context.
- `c.App() *fiber.App`: Access parent app instance.
- `c.OriginalURL() string`: Full raw URL.
- `c.IP() string`: Client IP.

---

## Part 9: Advanced Routing & Recipes

### 24. Route Constraints (Regex & Types)
Fiber support robust constraints in the path.

```go
// Constraint: Int only
app.Get("/users/:id<int>", handler)

// Constraint: Min Length
app.Get("/search/:query<minLen(3)>", handler)

// Constraint: Regex (Date YYYY-MM-DD)
app.Get("/calendar/:date<regex(\\d{4}-\\d{2}-\\d{2})>", handler)

// Custom Constraint (e.g., ULID)
app.RegisterCustomConstraint(&UlidConstraint{})
app.Get("/item/:id<ulid>", handler)
```

### 25. Grouping & Versioning
Organize large APIs using Groups.

```go
api := app.Group("/api")
v1 := api.Group("/v1")

// Apply middleware to group
v1.Use(authMiddleware)

v1.Get("/users", userHandler) // /api/v1/users
```

### 26. Recipe: JWT Authentication
Using `gofiber/contrib/jwt`.

```go
import jwtware "github.com/gofiber/contrib/jwt"

app.Use(jwtware.New(jwtware.Config{
    SigningKey: jwtware.SigningKey{Key: []byte("secret")},
    ErrorHandler: func(c fiber.Ctx, err error) error {
        return c.Status(401).SendString("Invalid or expired token")
    },
}))
```

### 27. Recipe: Server-Sent Events (SSE)
Real-time updates without WebSockets.

```go
app.Get("/sse", func(c fiber.Ctx) error {
    c.Set("Content-Type", "text/event-stream")
    c.set("Cache-Control", "no-cache")
    c.Set("Connection", "keep-alive")

    c.Context().SetBodyStreamWriter(func(w *bufio.Writer) {
        for {
            fmt.Fprintf(w, "data: %s\n\n", time.Now())
            w.Flush()
            time.Sleep(1 * time.Second)
        }
    })
    return nil
})
```

---

## Part 10: Cloud Mastery & Security Hardening

### 28. Kubernetes Health Probes
Fiber v3's `healthcheck` middleware essentially powers your K8s probes.

**Configuration for Deployments:**
```go
import "github.com/gofiber/fiber/v3/middleware/healthcheck"

// /livez -> Liveness Probe (Is the pod running?)
app.Get(healthcheck.LivenessEndpoint, healthcheck.New())

// /readyz -> Readiness Probe (Can I serve traffic?)
app.Get(healthcheck.ReadinessEndpoint, healthcheck.New(healthcheck.Config{
    Probe: func(c fiber.Ctx) bool {
        // Only return true if DB and Redis are connected
        return db.Ping() == nil && redis.Ping() == nil
    },
}))
```

**K8s Manifest:**
```yaml
livenessProbe:
  httpGet:
    path: /livez
    port: 3000
readinessProbe:
  httpGet:
    path: /readyz
    port: 3000
```

### 29. OAuth2 with Google (OIDC)
Modern auth requires OIDC. Use the `shareed2k/go_auth` pattern or manual integration.

**Manual Google Auth Flow:**
```go
func LoginGoogle(c fiber.Ctx) error {
    url := oauthConfig.AuthCodeURL("state")
    return c.Redirect(url)
}

func CallbackGoogle(c fiber.Ctx) error {
    code := c.Query("code")
    token, _ := oauthConfig.Exchange(context.Background(), code)
    // Use token to fetch user profile...
    return c.JSON(token)
}
```

### 30. Performance Profiling (pprof)
**Warning:** NEVER enable this on the public internet. Use internal IPs only or behind a VPN.

```go
import "github.com/gofiber/fiber/v3/middleware/pprof"

// Exposes /debug/pprof/*
app.Use(pprof.New())

// USAGE: 
// go tool pprof http://localhost:3000/debug/pprof/profile
```

---

## Part 11: View Engines & SSR

Fiber supports Server-Side Rendering (SSR) via the `gofiber/template` interface. It decouples the rendering engine from the core.

### 31. Template Engine Setup
Common engines include HTML (standard), Handlebars, Pug, and Django.

**Installation:**
```bash
go get github.com/gofiber/template/html/v2
go get github.com/gofiber/template/handlebars/v2
```

**Initialization:**
```go
import (
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/template/html/v2"
)

func main() {
    // 1. Initialize Engine
    engine := html.New("./views", ".html")
    
    // Optional: Hot Reload
    engine.Reload(true)

    // 2. Pass to Fiber Config
    app := fiber.New(fiber.Config{
        Views: engine,
    })

    // 3. Render
    app.Get("/", func(c fiber.Ctx) error {
        return c.Render("index", fiber.Map{
            "Title": "Hello SSR",
        }, "layouts/main")
    })
}
```

### 32. Layouts and Partials
Most engines support a "Base Layout" concept.
- `c.Render("index", data, "layouts/main")`
- The `index.html` content is injected into the `{{embed}}` slot of `layouts/main.html`.

---

## Part 12: API Documentation (Swagger/OpenAPI)

Don't write docs manually. Generate them from your Go code comments.

### 33. Setup Swaggo
1. **Install Tool:** `go install github.com/swaggo/swag/cmd/swag@latest`
2. **Install Middleware:** `go get github.com/gofiber/swagger`

### 34. Annotations & Generation
Add comments to your `main.go` and handlers.

```go
// @title My Fiber API
// @version 1.0
// @description This is a sample swagger server.
// @host localhost:3000
// @BasePath /
func main() { ... }

// GetUser godoc
// @Summary Get a user
// @Description get user by ID
// @Tags users
// @Accept  json
// @Produce  json
// @Param id path int true "User ID"
// @Success 200 {object} User
// @Router /users/{id} [get]
func GetUser(c fiber.Ctx) error { ... }
```

**Run Generation:**
```bash
swag init
```

**Serve Documentation:**
```go
import "github.com/gofiber/swagger"
import _ "myapp/docs" // Import generated docs

app.Get("/swagger/*", swagger.HandlerDefault)
// Visit http://localhost:3000/swagger/index.html
```

---

## Part 13: Advanced I/O

### 35. File Uploads
Fiber makes `multipart/form-data` trivial.

```go
app.Post("/upload", func(c fiber.Ctx) error {
    // 1. Get header
    file, err := c.FormFile("document")
    if err != nil { return err }

    // 2. Validate (Size/Extension)
    if file.Size > 10 * 1024 * 1024 { // 10MB
        return c.Status(413).SendString("File too large")
    }

    // 3. Save to disk
    // Note: In production, upload to S3 here using file.Open()
    return c.SaveFile(file, fmt.Sprintf("./uploads/%s", file.Filename))
})
```

### 36. Response Streaming
For large datasets or AI responses, stream data instead of buffering it.

**Using `SendStream` (Reader):**
```go
app.Get("/download", func(c fiber.Ctx) error {
    f, _ := os.Open("large_video.mp4")
    // Automatically handles Chunked Transfer Encoding
    return c.SendStream(f) 
})
```

**Using `SetBodyStreamWriter` (Writer):**
Perfect for generated content or NDJSON.

```go
app.Get("/stream", func(c fiber.Ctx) error {
    c.Set("Content-Type", "application/x-ndjson")
    
    c.Context().SetBodyStreamWriter(func(w *bufio.Writer) {
        encoder := json.NewEncoder(w)
        for i := 0; i < 1000; i++ {
            encoder.Encode(Item{ID: i})
            w.Flush() // Essential!
        }
    })
    return nil
})
```

---

## Part 14: Testing Strategy & Mocking

Robust applications effectively use testing to prevent regressions.

### 37. Integration Testing using `app.Test`
Fiber provides an `app.Test` method to interact with your application without starting the server, perfect for CI pipelines.

```go
import "github.com/stretchr/testify/assert"

func Test_Index(t *testing.T) {
    app := SetupApp() // Configured with handlers
    
    // Create Request
    req := httptest.NewRequest("GET", "/", nil)
    req.Header.Set("X-API-Key", "test-key")

    // Execute (Timeout 1s)
    resp, err := app.Test(req, 1000)
    
    assert.NoError(t, err)
    assert.Equal(t, 200, resp.StatusCode)
}
```

### 38. Mocking `fiber.Ctx`
Since `fiber.Ctx` is an interface in v3, you can create a legitimate mock for unit testing handlers in isolation.

```go
// Define a MockCtx (using testify/mock or manual)
type MockCtx struct {
    fiber.Ctx
}
func (m *MockCtx) JSON(data any) error {
    return nil // Mock implementation
}

func Test_Handler_Isolation(t *testing.T) {
    mock := &MockCtx{}
    err := MyHandler(mock)
    assert.NoError(t, err)
}
```

---

## Part 15: Resilience & Circuit Breakers

In microservices, prevent cascading failures using a Circuit Breaker.

### 39. Circuit Breaker Middleware
Wraps routes to stop sending traffic to a failing service.

**Config:**
```go
import "github.com/gofiber/contrib/circuitbreaker"

app.Use(circuitbreaker.New(circuitbreaker.Config{
    FailureThreshold: 3,               // 3 errors = open circuit
    SuccessThreshold: 1,               // 1 success = close circuit
    Timeout:          10 * time.Second, // Wait 10s before retrying (Half-Open)
    IsFailure: func(err error) bool {
        // Only 5xx errors count as failure
        return err != nil 
    },
    OnOpen: func(c fiber.Ctx) {
        log.Warn("Circuit Open: Service Unavailable")
    },
}))
```

---

## Part 16: GraphQL Integration

REST isn't the only way. Fiber integrates with standard Go GraphQL implementations.

### 40. With `99designs/gqlgen`
Use the `adaptor` package to wrap the standard `http.Handler` from gqlgen.

```go
import "github.com/99designs/gqlgen/graphql/handler"

func main() {
    app := fiber.New()
    
    // Create gqlgen handler
    gqlHandler := handler.NewDefaultServer(NewExecutableSchema(Config{...}))

    // Use Adaptor to compatibility layer
    app.Post("/query", adaptor.HTTPHandler(gqlHandler))
    
    // Playground
    app.Get("/", adaptor.HTTPHandler(playground.Handler("GraphQL", "/query")))
    
    app.Listen(":3000")
}
```

---

## Part 17: Serverless Fiber (Lambda / Vercel)

Fiber is an excellent candidate for serverless due to its low memory footprint and fast startup.

### 41. AWS Lambda / Vercel
To run Fiber on AWS Lambda or Vercel, use the `adaptor` to expose the app as a standard `http.Handler`.

**Vercel / Netlify / Generic Adapter:**
```go
import "github.com/gofiber/fiber/v3/middleware/adaptor"

// Handler Exported for Vercel/Lambda environment
func Handler(w http.ResponseWriter, r *http.Request) {
    app := fiber.New()
    app.Get("/", func(c fiber.Ctx) error { return c.SendString("In the Cloud!") })
    
    // Adapt to standard net/http
    adaptor.FiberApp(app).ServeHTTP(w, r)
}
```

**AWS API Gateway:**
Use `aws-lambda-go-api-proxy/httpadapter` with the standard http adaptor.
```go
import "github.com/awslabs/aws-lambda-go-api-proxy/httpadapter"

func main() {
    app := fiber.New()
    // ... setup routes ...
    
    // Bridge Fiber to Lambda
    adapter := httpadapter.New(adaptor.FiberApp(app))
    lambda.Start(adapter.ProxyWithContext)
}
```

---

## Part 18: Scaled Realtime (Redis Pub/Sub)

Scaling WebSockets across multiple server instances requires a Pub/Sub mechanism so that a message sent to Pod A is broadcast to users connected to Pod B.

### 42. Redis Broadcast Pattern

```go
// 1. Subscribe in background routine
go func() {
    sub := redisClient.Subscribe(ctx, "chat_channel")
    ch := sub.Channel()
    for msg := range ch {
        // Broadcast to ALL local connections
        for _, conn := range localConnections {
            conn.WriteMessage(websocket.TextMessage, []byte(msg.Payload))
        }
    }
}()

// 2. Publish from Handler
app.Post("/message", func(c fiber.Ctx) error {
    msg := c.FormValue("msg")
    // Publish to Redis -> Triggers subscribers on ALL nodes
    redisClient.Publish(ctx, "chat_channel", msg)
    return c.SendStatus(200)
})
```

---

## Part 19: Internationalization (i18n)

Global apps need to serve content in user's native language.

### 43. Using `fiberi18n`
Middleware that parses `Accept-Language` headers and loads translation files.

```go
import "github.com/gofiber/contrib/fiberi18n"

app.Use(fiberi18n.New(fiberi18n.Config{
    RootPath:        "./locales", // ./locales/en.yaml, ./locales/es.yaml
    AcceptLanguages: []language.Tag{language.English, language.Spanish},
    DefaultLanguage: language.English,
}))

app.Get("/", func(c fiber.Ctx) error {
    // Usage in handler
    greeting := fiberi18n.MustLocalize(c, "welcome_message")
    return c.SendString(greeting)
})
```

---

## Part 20: gRPC Microservices

Run gRPC and REST on the same port using `cmux` (Connection Multiplexer).

### 44. Multiplexing HTTP & gRPC
This allows a single microservice to accept high-performance internal gRPC calls and external HTTP calls on port 3000.

```go
import "github.com/soheilhy/cmux"

func main() {
    // 1. Create Listener
    l, _ := net.Listen("tcp", ":3000")
    m := cmux.New(l)

    // 2. Matchers
    grpcL := m.Match(cmux.HTTP2HeaderField("content-type", "application/grpc"))
    httpL := m.Match(cmux.Any())

    // 3. Setup gRPC Server
    grpcS := grpc.NewServer()
    // pb.RegisterMyService(grpcS, &server{})
    go grpcS.Serve(grpcL)

    // 4. Setup Fiber (via Adapter)
    app := fiber.New()
    app.Get("/", func(c fiber.Ctx) error { return c.SendString("HTTP!") })
    
    // Serve Fiber on the HTTP listener
    go http.Serve(httpL, adaptor.FiberApp(app))

    // 5. Start Multiplexer
    m.Serve()
}
```

---

## Part 21: The HTMX Stack

Go + Fiber + HTMX is a powerful "New Wave" stack for building hyper-interactive web apps with low complexity.

### 45. Detecting HX-Requests
You can detect if a request comes from HTMX to toggle between full layouts and partial fragments.

```go
app.Get("/contacts", func(c fiber.Ctx) error {
    users := db.GetUsers()
    
    // If request from HTMX, render ONLY the list
    if c.Get("HX-Request") == "true" {
        return c.Render("partials/user_list", users)
    }
    
    // Otherwise, render full page layout
    return c.Render("pages/users", users, "layouts/main")
})
```

### 46. Out-of-Band (OOB) Swaps
You can return multiple HTML fragments in one response to update different parts of the screen.

```go
app.Post("/todo", func(c fiber.Ctx) error {
    // 1. Add item
    item := db.AddItem(c.FormValue("item"))
    
    // 2. Return new item row AND update counter elsewhere
    return c.SendString(fmt.Sprintf(`
        <li id="todo-%d">%s</li>
        <span id="counter" hx-swap-oob="true">%d items</span>
    `, item.ID, item.Name, db.Count()))
})
```

---

## Part 22: Async Tasks (Asynq)

For expensive operations (emailing, report generation), do not block the request. Use `hibiken/asynq` (Redis-backed).

### 47. Enqueuing Tasks
```go
import "github.com/hibiken/asynq"

func main() {
    client := asynq.NewClient(asynq.RedisClientOpt{Addr: ":6379"})
    
    app.Post("/email", func(c fiber.Ctx) error {
        // Enqueue Task
        task, _ := asynq.NewTask("email:send", []byte("user_id=123"))
        info, err := client.Enqueue(task)
        
        return c.JSON(fiber.Map{"task_id": info.ID})
    })
}
```

### 48. Processing Tasks (Worker)
Run this in a separate goroutine or binary.

```go
func startWorker() {
    srv := asynq.NewServer(asynq.RedisClientOpt{Addr: ":6379"}, asynq.Config{Concurrency: 10})
    
    srv.Run(asynq.HandlerFunc(func(ctx context.Context, t *asynq.Task) error {
        // Handle "email:send"
        log.Println("Sending email...", string(t.Payload()))
        return nil
    }))
}
```

---

## Part 23: Advanced Config (Viper)

Hardcoding ports and secrets is bad practice. Use `spf13/viper` to load from ENV, JSON, or YAML.

### 49. Loading Configuration
```go
import "github.com/spf13/viper"

type Config struct {
    Port  string `mapstructure:"PORT"`
    DBUrl string `mapstructure:"DATABASE_URL"`
}

func LoadConfig() (config Config) {
    viper.SetConfigFile(".env")
    viper.AutomaticEnv() // Read ENV variables
    
    viper.ReadInConfig()
    viper.Unmarshal(&config)
    return
}

func main() {
    cfg := LoadConfig()
    app.Listen(":" + cfg.Port)
}
```

---

## Part 24: Developer Experience (Air)

Stop restarting your server manually. Use `cosmtrek/air` for instant live reloading.

### 50. `.air.toml` Configuration
Create this file in your root.

```toml
root = "."
tmp_dir = "tmp"

[build]
  cmd = "go build -o ./tmp/main ."
  bin = "./tmp/main"
  delay = 500 # ms
  include_ext = ["go", "tpl", "html"]
  exclude_dir = ["assets", "tmp"]

[log]
  time = true

[color]
  main = "magenta"
  watcher = "cyan"
  build = "yellow"
```

**Run:**
```bash
air
```

---

## Part 25: Database Mastery (MongoDB / SQLite)

### 51. MongoDB Integration (Official Driver)
Fiber v3 pairs elegantly with `mongo-go-driver`. Use a singleton pattern.

```go
import "go.mongodb.org/mongo-driver/mongo"

func ConnectMongo() *mongo.Database {
    opts := options.Client().ApplyURI("mongodb://localhost:27017")
    client, _ := mongo.Connect(context.Background(), opts)
    return client.Database("app_db")
}

// Handler
func CreateUser(c fiber.Ctx) error {
    coll := db.Collection("users")
    user := &User{}
    
    if err := c.Bind().Body(user); err != nil {
         return err 
    }
    
    // IMPORTANT: Use c.UserContext() to respect timeout
    _, err := coll.InsertOne(c.UserContext(), user)
    return err
}
```

### 52. SQLite (CGO vs. Pure Go)
For edge deployments or embedded systems.

**Option A: `modernc.org/sqlite` (Pure Go)**
- Pros: No CGO, easy cross-compile (Windows to Linux/ARM).
- Cons: Slower (~2x) than CGO version for heavy writes.

**Option B: `mattn/go-sqlite3` (CGO)**
- Pros: Standard, fastest performance.
- Cons: Requires GCC to build.

**WAL Mode (Critical for Concurrency):**
By default, SQLite locks the whole DB on write. Enable Write-Ahead Logging.

```go
db, _ := sql.Open("sqlite", "file:data.db?_pragma=journal_mode(WAL)&_pragma=busy_timeout(5000)")
```

---

## Part 26: Cloud Auth Providers

### 53. Firebase Auth Middleware
Don't roll your own auth if you don't have to. Verify ID Tokens from Firebase.

```go
import "firebase.google.com/go/v4/auth"

func FirebaseAuth(app *firebase.App) fiber.Handler {
    client, _ := app.Auth(context.Background())

    return func(c fiber.Ctx) error {
        // 1. Get Token
        token := strings.TrimPrefix(c.Get("Authorization"), "Bearer ")
        
        // 2. Verify
        decoded, err := client.VerifyIDToken(c.UserContext(), token)
        if err != nil {
            return c.Status(401).SendString("Invalid Token")
        }

        // 3. Store User ID
        c.Locals("uid", decoded.UID)
        return c.Next()
    }
}
```

### 54. Auth0 / OIDC Integration
Using `jwtware` with a JWKS endpoint (auto-rotates keys).

```go
app.Use(jwtware.New(jwtware.Config{
    JWKSetURLs: []string{
        "https://YOUR_DOMAIN.auth0.com/.well-known/jwks.json",
    },
    SigningMethod: "RS256",
}))
```

---

## Part 27: Low-Level Tuning

For systems doing 100k+ RPS, default settings may throttle you.

### 55. Fiber / Fasthttp Configuration
Adjust these in `fiber.New()`.

| Setting | Recommendation (High Traffic) | Why? |
| :--- | :--- | :--- |
| `ReadBufferSize` | `8 * 1024` (8KB) | Reduce memory per conn if headers are small. |
| `WriteBufferSize` | `4 * 1024` (4KB) | Flush data faster. |
| `MaxConnsPerHost` | `512` | Throttle outgoing clients to prevent port exhaustion. |
| `ReduceMemoryUsage` | `true` | Aggressively release memory (increases CPU usage slightly). |

### 56. System Tuning (`GOMAXPROCS`)
Fiber is CPU-bound.
- **Microservice:** `runtime.GOMAXPROCS(1)` fits well in small K8s pods (1 vCPU).
- **Monolith:** Leave default (all cores).
- **Affinity:** On Linux, use `taskset` to pin processes to cores for L2 cache locality.

---

## Part 28: Audit & Compliance

### 57. Audit Logger Middleware
For FinTech/HealthTech, you must log *who* did *what*.

```go
func AuditLogger(c fiber.Ctx) error {
    // 1. Process Request
    err := c.Next()
    
    // 2. Log Metadata (After request processed)
    log.Info("Audit Log",
        "user_id", c.Locals("uid"),
        "method", c.Method(),
        "path", c.OriginalURL(),
        "status", c.Response().StatusCode(),
        "ip", c.IP(),
        // REDACT Sensitive Data!
        "auth_header", "[REDACTED]", 
    )
    return err
}
```

### 58. Data Redaction
Never log `password`, `token`, or `ssn` bodies. Use a custom logger that scans JSON keys and masks values before printing.

---

## Part 29: Event-Driven Architecture

In modern architectures, services communicate via events.

### 59. Kafka Integration (Produce/Consume)
Using `segmentio/kafka-go` (Pure Go).

**Producer Pattern:**
```go
import "github.com/segmentio/kafka-go"

// Initialize Writer (Singleton)
var kafkaWriter = &kafka.Writer{
    Addr:     kafka.TCP("localhost:9092"),
    Topic:    "user-events",
    Balancer: &kafka.LeastBytes{},
}

func ProduceHandler(c fiber.Ctx) error {
    // Use Fiber's UserContext for timeouts
    err := kafkaWriter.WriteMessages(c.UserContext(),
        kafka.Message{
            Key:   []byte(c.Params("id")),
            Value: c.Body(),
        },
    )
    return err
}
```

**Consumer Worker:**
Consumers should run in a separate goroutine or service, not in the Fiber handler.

```go
func StartConsumer() {
    r := kafka.NewReader(kafka.ReaderConfig{
        Brokers: []string{"localhost:9092"},
        Topic:   "user-events",
        GroupID: "my-group",
    })

    for {
        m, err := r.ReadMessage(context.Background())
        if err != nil { break }
        fmt.Printf("message at offset %d: %s = %s\n", m.Offset, string(m.Key), string(m.Value))
    }
}
```

---

## Part 30: Multi-Tenancy Patterns

SaaS applications often need to isolate data per tenant.

### 60. Subdomain Tenant Resolution
Extract tenant ID from `tenant.app.com` and store in context.

```go
func TenantMiddleware(c fiber.Ctx) error {
    // 1. Extract Subdomain
    parts := c.Subdomains()
    if len(parts) == 0 {
        return c.Status(404).SendString("Tenant Not Found")
    }
    tenantID := parts[0] // e.g. "acme"

    // 2. Store in Context (Immutable)
    c.Locals("tenant_id", tenantID)

    return c.Next()
}
```

### 61. Data Isolation Strategy
Pass the `tenant_id` to your repository layer.

```go
func GetOrders(c fiber.Ctx) error {
    tenantID := c.Locals("tenant_id").(string)
    
    // Repository method forces tenant_id check
    orders := repo.FindAllOrders(tenantID) 
    // SELECT * FROM orders WHERE tenant_id = $1
    
    return c.JSON(orders)
}
```

---

## Part 31: Feature Flags

Decouple deploy from release using Feature Flags.

### 62. OpenFeature Integration
Standardize your flagging.

```go
import "github.com/open-feature/go-sdk/openfeature"

func FeatureFlagMiddleware(c fiber.Ctx) error {
    client := openfeature.GetClient("my-app")
    
    // Evaluate flag "new-dashboard" for this user
    enabled, _ := client.BooleanValue(
        context.Background(), 
        "new-dashboard", 
        false, 
        openfeature.NewEvaluationContext(c.Locals("user_id").(string), nil),
    )

    if enabled {
        return c.Next() // Continue to new feature
    }
    return c.Redirect("/legacy-dashboard") // Fallback
}
```

---

## Part 32: Bot Protection

Protect public forms from automated abuse using Cloudflare Turnstile (a user-friendly CAPTCHA alternative).

### 63. Turnstile Validation Middleware
Validate the token server-side before processing the form.

```go
type TurnstileRequest struct {
    Secret   string `json:"secret"`
    Response string `json:"response"`
    RemoteIP string `json:"remoteip"`
}

type TurnstileResponse struct {
    Success bool `json:"success"`
}

func ValidateTurnstile(c fiber.Ctx) error {
    token := c.FormValue("cf-turnstile-response")
    ip := c.IP()

    // Call Cloudflare API
    agent := fiber.Post("https://challenges.cloudflare.com/turnstile/v0/siteverify")
    args := fiber.AcquireArgs()
    args.Set("secret", "YOUR_SECRET_KEY")
    args.Set("response", token)
    args.Set("remoteip", ip)

    // Check response
    _, body, errs := agent.Form(args).Bytes()
    if len(errs) > 0 {
        return c.Status(403).SendString("Bot check failed")
    }
    
    var result TurnstileResponse
    json.Unmarshal(body, &result)
    
    if !result.Success {
        return c.Status(403).SendString("Invalid CAPTCHA")
    }

    return c.Next()
}
```

---

## Part 33: WebAssembly (The Go Stack)

Go can compile to `.wasm`. Fiber can serve it efficiently.

### 64. Serving `.wasm` with Compression
Ensure the correct MIME type and enable B Brotli/Gzip compression.

```go
func main() {
    app := fiber.New()
    
    // Serve static directory with compression enabled
    app.Static("/", "./assets", fiber.Static{
        Compress: true, // Will serve .wasm.gz or .wasm.br if available
        ModifyResponse: func(c fiber.Ctx) error {
            if strings.HasSuffix(c.Path(), ".wasm") {
                c.Set("Content-Type", "application/wasm")
            }
            return nil
        },
    })
}
```

**Antigravity Tip:**
Compile your Go frontend with `tinygo` to reduce `main.wasm` size from ~2MB to ~300KB.

---

## Part 34: IoT & MQTT Bridging

Connect your Fiber API to millions of devices.

### 65. MQTT Client Integration
Using `paho.mqtt.golang`.

```go
import mqtt "github.com/eclipse/paho.mqtt.golang"

var mqttClient mqtt.Client

func InitMQTT() {
    opts := mqtt.NewClientOptions().AddBroker("tcp://broker.hivemq.com:1883")
    mqttClient = mqtt.NewClient(opts)
    if token := mqttClient.Connect(); token.Wait() && token.Error() != nil {
        panic(token.Error())
    }
}

// REST -> MQTT Bridge
func PublishControl(c fiber.Ctx) error {
    deviceID := c.Params("id")
    command := c.Body() // e.g. "TURN_ON"
    
    token := mqttClient.Publish("devices/"+deviceID+"/control", 0, false, command)
    token.Wait()
    
    return c.SendStatus(200)
}
```

---

## Part 35: Financial Ops (Stripe)

Handling money? You must verify webhooks to prevent spoofing.

### 66. Stripe Webhook Verification
This handler **must** read the raw body signature.

```go
import "github.com/stripe/stripe-go/v78/webhook"

func StripeWebhook(c fiber.Ctx) error {
    const MaxBodyBytes = 65536
    payload := c.Body()
    sigHeader := c.Get("Stripe-Signature")
    secret := os.Getenv("STRIPE_WEBHOOK_SECRET")

    // Verify Signature
    event, err := webhook.ConstructEvent(payload, sigHeader, secret)
    if err != nil {
        return c.Status(400).SendString("Invalid Signature")
    }

    // Handle Event
    switch event.Type {
    case "payment_intent.succeeded":
        // Unlock user feature
    case "customer.subscription.deleted":
        // Revoke access
    }

    return c.SendStatus(200)
}
```

---

## Part 36: Geolocation & Blocking

Restrict access by country or customize content.

### 67. MaxMind GeoIP2 Middleware
Load the MMDB file once on startup.

```go
import "github.com/oschwald/geoip2-golang"

func GeoMiddleware(db *geoip2.Reader) fiber.Handler {
    return func(c fiber.Ctx) error {
        ip := net.ParseIP(c.IP())
        record, _ := db.City(ip)
        
        // Store Country Code
        c.Locals("country", record.Country.IsoCode) // "US", "DE"
        
        // Example: Block Country
        if record.Country.IsoCode == "RU" {
            return c.Status(403).SendString("Service unavailable in your region")
        }

        return c.Next()
    }
}
```

---

## Part 37: Distributed Locking

When running multiple replicas of Fiber (e.g., in K8s), you must prevent scheduled tasks or mutations from running concurrently twice.

### 68. Redis Lock Implementation
Use `bsm/redislock` to ensure atomicity.

```go
import "github.com/bsm/redislock"

func ProcessJob(c fiber.Ctx) error {
    // 1. Try key acquisition (Non-blocking)
    lock, err := locker.Obtain(c.UserContext(), "job:billing", 10*time.Minute, nil)
    
    if err == redislock.ErrNotObtained {
        return c.Status(423).SendString("Job already running")
    } else if err != nil {
        return err
    }
    defer lock.Release(c.UserContext())

    // 2. Critical Section
    PerformExpensiveBilling()
    
    return c.SendStatus(200)
}
```

---

## Part 38: Data Encryption (AES-GCM)

GDPR/HIPAA compliance often requires encryption at rest for PII.

### 69. AES-GCM 256 Implementation
Standard authenticated encryption.

```go
var secretKey = []byte("YOUR_32_BYTE_HEX_KEY_HERE_!!!!!!") // 32 bytes = AES-256

func Encrypt(plaintext []byte) string {
    c, _ := aes.NewCipher(secretKey)
    gcm, _ := cipher.NewGCM(c)
    nonce := make([]byte, gcm.NonceSize())
    io.ReadFull(rand.Reader, nonce)
    return hex.EncodeToString(gcm.Seal(nonce, nonce, plaintext, nil))
}

func Decrypt(ciphertextHex string) string {
    data, _ := hex.DecodeString(ciphertextHex)
    c, _ := aes.NewCipher(secretKey)
    gcm, _ := cipher.NewGCM(c)
    nonceSize := gcm.NonceSize()
    nonce, ciphertext := data[:nonceSize], data[nonceSize:]
    plaintext, _ := gcm.Open(nil, nonce, ciphertext, nil)
    return string(plaintext)
}
```

---

## Part 39: Job Scheduling

Run background tasks (cleanup, digest emails) inside your app.

### 70. Cron Integration
Using `robfig/cron/v3`.

```go
import "github.com/robfig/cron/v3"

func StartScheduler() {
    c := cron.New()
    
    // Run every day at midnight
    c.AddFunc("0 0 * * *", func() {
        log.Println("Running daily cleanup...")
        db.Exec("DELETE FROM logs WHERE created_at < NOW() - INTERVAL '30 DAYS'")
    })
    
    c.Start()
}

func main() {
    go StartScheduler()
    // ... start Fiber ...
}
```

---

## Part 40: Database Migrations

Schema changes should be automated, reversible, and versioned.

### 71. Using `golang-migrate`
Run migrations on startup to ensure DB is in sync with code.

```go
import (
    "github.com/golang-migrate/migrate/v4"
    _ "github.com/golang-migrate/migrate/v4/database/postgres"
    _ "github.com/golang-migrate/migrate/v4/source/file"
)

func MigrateDB() {
    m, err := migrate.New(
        "file://migrations",
        "postgres://user:pass@localhost:5432/db?sslmode=disable",
    )
    if err != nil {
        log.Fatal(err)
    }
    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        log.Fatal(err)
    }
    log.Println("Database migrated successfully")
}
```

---

## Part 41: Advanced Caching Patterns

Naive caching (manual `Set`/`Get`) leads to code duplication. Use Generics in Go 1.25+ for a type-safe Cache-Aside implementation.

### 72. Generic Cache-Aside
Wrapper that automatically handles "Get -> Miss -> Fetch -> Set" logic.

```go
// Fetcher function type
type Fetcher[T any] func() (T, error)

func CacheAside[T any](c fiber.Ctx, key string, ttl time.Duration, fetch Fetcher[T]) (T, error) {
    var data T
    
    // 1. Try Cache
    val, err := redisClient.Get(c.UserContext(), key).Bytes()
    if err == nil {
        json.Unmarshal(val, &data)
        return data, nil
    }

    // 2. Fetch from DB (Cache Miss)
    data, err = fetch()
    if err != nil {
        return data, err
    }

    // 3. Set Cache (Async to not block response)
    go func() {
        bytes, _ := json.Marshal(data)
        redisClient.Set(context.Background(), key, bytes, ttl)
    }()

    return data, nil
}

// Usage
func GetProduct(c fiber.Ctx) error {
    product, _ := CacheAside(c, "product:123", 10*time.Minute, func() (Product, error) {
        return repo.GetProduct(123)
    })
    return c.JSON(product)
}
```

---

## Part 42: API Gateway Pattern

Fiber is fast enough to act as an API Gateway, aggregating downstream services.

### 73. Using `middleware/proxy`
Reverse proxy traffic to microservices.

```go
import "github.com/gofiber/fiber/v3/middleware/proxy"

func main() {
    app := fiber.New()

    // 1. Auth Service Route
    app.Post("/auth/*", proxy.Balancer(proxy.Config{
        Servers: []string{"http://auth-service:3001"},
    }))

    // 2. Billing Service (Round Robin Load Balancing)
    app.All("/billing/*", proxy.Balancer(proxy.Config{
        Servers: []string{
            "http://billing-1:3002",
            "http://billing-2:3002",
        },
    }))

    // 3. Rate Limit Gateway
    app.Use(limiter.New(limiter.Config{Max: 1000})) 

    app.Listen(":80")
}
```

---

## Part 43: Transactional Emails

Sending nice HTML emails (Reset Password, Welcome) is standard.

### 74. Hermes + Gomail
Use `hermes` to generate responsive HTML tables/buttons.

```go
import (
    "github.com/matcornic/hermes/v2"
    "gopkg.in/gomail.v2"
)

func SendWelcomeEmail(email string, name string) {
    // 1. Generate HTML
    h := hermes.Hermes{
        Product: hermes.Product{
            Name: "MyApp",
            Link: "https://myapp.com",
            Logo: "https://myapp.com/logo.png",
        },
    }
    
    emailBody := hermes.Email{
        Body: hermes.Body{
            Name: name,
            Intros: []string{"Welcome to the future of Go!"},
            Actions: []hermes.Action{
                {
                    Instructions: "To get started, please click here:",
                    Button: hermes.Button{
                        Color: "#22BC66",
                        Text:  "Confirm Account",
                        Link:  "https://myapp.com/confirm",
                    },
                },
            },
        },
    }

    htmlBody, _ := h.GenerateHTML(emailBody)

    // 2. Send via SMTP
    m := gomail.NewMessage()
    m.SetHeader("From", "no-reply@myapp.com")
    m.SetHeader("To", email)
    m.SetHeader("Subject", "Welcome!")
    m.SetBody("text/html", htmlBody)

    d := gomail.NewDialer("smtp.example.com", 587, "user", "pass")
    if err := d.DialAndSend(m); err != nil {
        log.Println("Email failed:", err)
    }
}
```

---

## Part 44: Panic Recovery & Stack Traces

In production, if a nil pointer exception happens, the server must NOT crash, but you MUST know where it happened.

### 75. Advanced Recovery Config
Customize `recover` middleware to print stack traces only in Dev/Staging.

```go
import "github.com/gofiber/fiber/v3/middleware/recover"

app.Use(recover.New(recover.Config{
    EnableStackTrace: true,
    StackTraceHandler: func(c fiber.Ctx, e interface{}) {
        // Log to Sentry / CloudWatch / File
        log.Error("PANIC RECOVERED", 
            "error", e,
            "url", c.OriginalURL(),
            "stack", string(debug.Stack()), // Full stack trace
        )
    },
}))
```

---

## Part 45: PDF Reports

Generate Invoices or Reports on the fly.

### 76. Using `maroto` (High-Level)
Built on `gofpdf`, but with a grid system (Row/Col).

```go
import "github.com/johnfercher/maroto/pkg/pdf"
import "github.com/johnfercher/maroto/pkg/props"

func GenerateInvoice(c fiber.Ctx) error {
    m := pdf.NewMaroto(consts.Portrait, consts.A4)
    
    // Header
    m.Row(20, func() {
        m.Col(12, func() {
            m.Text("INVOICE #12345", props.Text{
                Top: 5,
                Style: consts.Bold,
                Align: consts.Center,
            })
        })
    })

    // Content
    m.Row(10, func() {
        m.Col(12, func() {
            m.Text("Product: Go Fiber Consulting", props.Text{Top: 2})
        })
    })

    // Output to Buffer
    buffer, err := m.Output()
    if err != nil {
        return c.Status(500).SendString("PDF Gen Failed")
    }

    // Stream to Client
    c.Set("Content-Type", "application/pdf")
    return c.SendStream(bytes.NewReader(buffer.Bytes()))
}
```

---

## Part 46: Excel Exports

Don't crash the server trying to export 1M rows.

### 77. Using `excelize` (Streaming)
Use `StreamWriter` for O(1) memory usage.

```go
import "github.com/xuri/excelize/v2"

func ExportUsers(c fiber.Ctx) error {
    f := excelize.NewFile()
    defer f.Close()

    // 1. Get Stream Writer
    sw, _ := f.NewStreamWriter("Sheet1")
    
    // 2. Write Header
    sw.SetRow("A1", []interface{}{"ID", "Name", "Email", "Created At"})

    // 3. Write Rows (Batch)
    users := db.GetUsersCursor() // Imagine a DB cursor
    rowID := 2
    for users.Next() {
        u := users.Scan()
        cell, _ := excelize.CoordinatesToCellName(1, rowID)
        sw.SetRow(cell, []interface{}{u.ID, u.Name, u.Email, u.CreatedAt})
        rowID++
    }

    sw.Flush()

    // 4. Send Response
    c.Set("Content-Type", "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")
    c.Set("Content-Disposition", "attachment; filename=users.xlsx")
    
    return f.WriteTo(c.Response().BodyWriter())
}
```

---

## Part 47: Image Processing

Resize user avatars before saving to S3.

### 78. Using `disintegration/imaging`
High-quality resizing using Lanczos filter.

```go
import "github.com/disintegration/imaging"

func UploadAvatar(c fiber.Ctx) error {
    file, _ := c.FormFile("avatar")
    src, _ := file.Open()
    defer src.Close()

    // 1. Decode
    img, err := imaging.Decode(src)
    if err != nil { return err }

    // 2. Resize (Thumbnail)
    thumb := imaging.Resize(img, 128, 128, imaging.Lanczos)

    // 3. Save to Buffer (for S3 upload)
    buf := new(bytes.Buffer)
    imaging.Encode(buf, thumb, imaging.JPEG)

    // Upload 'buf' to S3...
    return c.SendString("Avatar uploaded")
}
```

---

## Part 48: 2FA (TOTP)

Add Google Authenticator support.

### 79. Using `pquerna/otp`
Generate secrets and QR codes.

```go
import (
    "github.com/pquerna/otp"
    "github.com/pquerna/otp/totp"
)

// 1. Setup (Generate Secret & QR)
func Setup2FA(c fiber.Ctx) error {
    key, _ := totp.Generate(totp.GenerateOpts{
        Issuer:      "MyApp",
        AccountName: "user@example.com",
    })
    
    // Save key.Secret() to DB!
    user.TwoFASecret = key.Secret()
    db.Save(user)

    // Return QR Code (Base64) to frontend
    var buf bytes.Buffer
    img, _ := key.Image(200, 200)
    png.Encode(&buf, img)
    
    return c.JSON(fiber.Map{
        "secret": key.Secret(),
        "qr":     base64.StdEncoding.EncodeToString(buf.Bytes()),
    })
}

// 2. Validate
func Verify2FA(c fiber.Ctx) error {
    code := c.FormValue("code") // 123456
    secret := user.TwoFASecret // From DB
    
    valid := totp.Validate(code, secret)
    if !valid {
        return c.Status(401).SendString("Invalid Code")
    }
    
    return c.SendString("Logged In!")
}
```

---

## Part 49: Real-Time Media (WebRTC)

Building Zoom/Google Meet clones in Go using Pion.

### 80. Simple Signaling Server
Exchange SDP offers/answers over Fiber Websockets.

```go
import "github.com/pion/webrtc/v3"

// Map to store peer connections
var peers = make(map[string]*webrtc.PeerConnection)

func SignalingHandler(c *websocket.Conn) {
    defer c.Close()

    // 1. Setup Pion PeerConnection
    config := webrtc.Configuration{
        ICEServers: []webrtc.ICEServer{
            {URLs: []string{"stun:stun.l.google.com:19302"}},
        },
    }
    peerConnection, _ := webrtc.NewPeerConnection(config)

    // 2. Handle ICE Candidates
    peerConnection.OnICECandidate(func(i *webrtc.ICECandidate) {
        if i != nil {
            c.WriteJSON(fiber.Map{"type": "candidate", "candidate": i.ToJSON()})
        }
    })

    // 3. Loop: Read SDP from Client
    for {
        var msg map[string]interface{}
        c.ReadJSON(&msg)
        
        switch msg["type"] {
        case "offer":
            // Set Remote Description & Create Answer (Omitted for brevity)
        case "candidate":
            // Add ICE Candidate
        }
    }
}
```

---

## Part 50: Embedded DB (Badger)

Build single-binary apps without external dependencies (Postgres/Redis).

### 81. Using `dgraph-io/badger`
Fast, persistent Key-Value store. Use it for local caching or primary DB.

```go
import "github.com/dgraph-io/badger/v4"

func InitBadger() *badger.DB {
    db, err := badger.Open(badger.DefaultOptions("/tmp/badger"))
    if err != nil { log.Fatal(err) }
    return db
}

func CacheUser(db *badger.DB, c fiber.Ctx) error {
    userID := c.Params("id")
    
    // Read/Write Transaction
    err := db.Update(func(txn *badger.Txn) error {
        return txn.Set([]byte("user:"+userID), []byte("John Doe"))
    })
    
    return err
}

func GetUser(db *badger.DB, c fiber.Ctx) error {
    userID := c.Params("id")
    var valCopy []byte
    
    // Read-Only Transaction
    db.View(func(txn *badger.Txn) error {
        item, err := txn.Get([]byte("user:"+userID))
        if err != nil { return err }
        
        valCopy, err = item.ValueCopy(nil)
        return err
    })
    
    return c.SendString(string(valCopy))
}
```

---

## Part 51: Self-Updating Binaries

Deploy CLI tools or agents that update themselves.

### 82. Using `minio/selfupdate`
Check a URL for a new binary and hot-swap.

```go
import "github.com/minio/selfupdate"

func CheckForUpdates() {
    resp, err := http.Get("https://myapp.com/api/latest-version")
    if err != nil { return }
    defer resp.Body.Close()

    // If version > current:
    err = selfupdate.Apply(resp.Body, selfupdate.Options{
        Checksum: []byte("expected-sha256-hash"),
    })
    
    if err != nil {
        log.Println("Update failed:", err)
    } else {
        log.Println("Update successful! Restarting...")
        os.Exit(0)
    }
}

// Add an endpoint to trigger manual update
func UpdateHandler(c fiber.Ctx) error {
    go CheckForUpdates()
    return c.SendString("Update initiated...")
}
```

---

## Part 52: Hardware I/O

Fiber controlling industrial machinery or IoT gateways.

### 83. Reading Serial Ports (`tarm/serial`)
Read data from USB/RS232 devices and stream to WebSocket clients.

```go
import "github.com/tarm/serial"

func StreamSerialData(c *websocket.Conn) {
    // 1. Open /dev/ttyUSB0
    config := &serial.Config{Name: "/dev/ttyUSB0", Baud: 115200}
    s, err := serial.OpenPort(config)
    if err != nil {
        log.Println("Serial Error:", err)
        return
    }
    defer s.Close()

    buf := make([]byte, 128)
    for {
        // 2. Read Hardware
        n, err := s.Read(buf)
        if err != nil { break }
        
        // 3. Push to Web
        data := string(buf[:n])
        if err := c.WriteMessage(websocket.TextMessage, []byte(data)); err != nil {
            break
        }
    }
}
```

---

## Part 53: Durable Execution (Temporal)

Build invincible applications that survive outages, restarts, and crashes without losing state.

### 84. Temporal + Fiber Integration
Initiate workflows from REST, process via Workers.

```go
import "go.temporal.io/sdk/client"

// 1. Fiber Handler (Starter)
func StartOrderWorkflow(c fiber.Ctx) error {
    // Initialize Temporal Client (Singleton)
    temporalClient, _ := client.Dial(client.Options{})
    defer temporalClient.Close()

    options := client.StartWorkflowOptions{
        ID:        "order-123",
        TaskQueue: "ORDER_QUEUE",
    }

    // Hand off to Temporal
    we, err := temporalClient.ExecuteWorkflow(context.Background(), options, "OrderProcessingWorkflow", c.Body())
    if err != nil {
        return c.Status(500).SendString(err.Error())
    }

    return c.JSON(fiber.Map{"workflowID": we.GetID(), "runID": we.GetRunID()})
}

// 2. The Workflow (Deterministic Logic - runs in Worker)
func OrderProcessingWorkflow(ctx workflow.Context, order Order) error {
    options := workflow.ActivityOptions{StartToCloseTimeout: time.Minute}
    ctx = workflow.WithActivityOptions(ctx, options)

    // Execute Activities (Retried automatically on failure)
    var result string
    err := workflow.ExecuteActivity(ctx, "ProcessPayment", order).Get(ctx, &result)
    if err != nil {
        return err // Temporal handles retries
    }
    
    return nil
}
```

---

## Part 54: Policy-as-Code (OPA)

Decouple authorization logic from business logic using Open Policy Agent (Rego).

### 85. OPA Middleware
Enforce complex RBAC/ABAC rules.

```go
import "github.com/gofiber/contrib/opafiber"

func main() {
    app := fiber.New()

    // Apply OPA Middleware
    app.Use(opafiber.New(opafiber.Config{
        URL: "http://localhost:8181/v1/data/authz/allow", // OPA Sidecar
        InputCreationMethod: func(c fiber.Ctx) (map[string]interface{}, error) {
            return map[string]interface{}{
                "method": c.Method(),
                "path":   c.Path(),
                "user":   c.Locals("user"), // From Auth Middleware
            }, nil
        },
    }))

    // Handlers don't need to check permissions anymore!
    app.Get("/admin", func(c fiber.Ctx) error {
        return c.SendString("Welcome Admin!")
    })
}
```

**Rego Policy (example.rego):**
```rego
package authz
default allow = false
allow {
    input.method == "GET"
    input.path == "/admin"
    input.user.role == "admin"
}
```

---

## Part 55: Fuzz Testing (Go 1.18+)

Find crashes and security vulnerabilities automatically.

### 86. Native Fuzzing Handlers
Test your API with millions of random inputs.

```go
func FuzzMyHandler(f *testing.F) {
    app := fiber.New()
    app.Post("/calc", func(c fiber.Ctx) error {
        var req struct { Divisor int }
        if err := c.Bind().Body(&req); err != nil { return err }
        // Potential panic: Divide by Zero
        return c.SendString(fmt.Sprintf("%d", 100/req.Divisor))
    })

    // Seed Corpus
    f.Add(10)
    f.Add(0)

    f.Fuzz(func(t *testing.T, divisor int) {
        reqBody := fmt.Sprintf(`{"Divisor": %d}`, divisor)
        req := httptest.NewRequest("POST", "/calc", strings.NewReader(reqBody))
        req.Header.Set("Content-Type", "application/json")
        
        resp, _ := app.Test(req, -1)
        
        // If app.Test panics, Fuzzing detects it!
        if resp.StatusCode == 500 {
            // Found a bug!
        }
    })
}
```

---

## Part 56: Service Mesh Integration

When running in Istio/Linkerd, you must propagate headers to maintain Distributed Tracing.

### 87. Trace Header Propagation
Forward `x-request-id` and `b3` headers to downstream services.

```go
func ProxyHandler(c fiber.Ctx) error {
    // 1. Create Outgoing Request
    agent := fiber.Get("http://billing-service/invoice")

    // 2. Propagate Key Headers (Zipkin/B3)
    headersToPropagate := []string{
        "x-request-id",
        "x-b3-traceid",
        "x-b3-spanid",
        "x-b3-parentspanid",
        "x-b3-sampled",
        "x-b3-flags",
    }

    for _, h := range headersToPropagate {
        if val := c.Get(h); val != "" {
            agent.Set(h, val)
        }
    }

    // 3. Execute
    status, _, _ := agent.Bytes()
    return c.SendStatus(status)
}
```

---

## Part 57: Domain-Driven Design (DDD)

Stop building "Anemic" models. Put logic where it belongs.

### 88. Aggregate & Entity Pattern
Don't use `gorm.Model` in your domain.

```go
// Domain Entity (Pure Go - No JSON/DB tags)
type Order struct {
    ID         uuid.UUID
    Items      []OrderItem
    Status     OrderStatus
    TotalPrice Money
}

// Business Logic in Entity
func (o *Order) AddItem(item OrderItem) error {
    if o.Status != StatusDraft {
        return errors.New("cannot change finalized order")
    }
    o.Items = append(o.Items, item)
    o.RecalculateTotal()
    return nil
}

// Interface Adapter (Fiber Handler)
// Maps DTO -> Domain Entity -> Service
func AddItemHandler(c fiber.Ctx) error {
    var dto AddItemDTO
    c.Bind().Body(&dto)

    order, _ := repo.FindByID(dto.OrderID)
    
    // Logic happens here
    if err := order.AddItem(dto.ToDomain()); err != nil {
        return c.Status(400).SendString(err.Error())
    }

    repo.Save(order)
    return c.SendStatus(200)
}
```

---

## Part 58: Hexagonal Architecture (Ports & Adapters)

Decouple Fiber from your Core. Fiber is just an "Adapter" (Driving Side).

### 89. Interface Decoupling
Depend on interfaces, not structs.

```go
// PORT: Driving (The "What")
type OrderService interface {
    PlaceOrder(ctx context.Context, cmd PlaceOrderCommand) error
}

// ADAPTER: Driving (The "How" - Fiber)
type OrderHandler struct {
    svc OrderService
}

func (h *OrderHandler) PlaceOrder(c fiber.Ctx) error {
    // Adapter logic: HTTP -> Domain
    return h.svc.PlaceOrder(c.UserContext(), cmd)
}

// PORT: Driven (Repository)
type OrderRepo interface {
    Save(ctx context.Context, order Order) error
}

// ADAPTER: Driven (Postgres)
type PostgresOrderRepo struct {
    db *pgxpool.Pool
} // implements OrderRepo
```

---

## Part 59: Property-Based Testing

Don't write `TableDrivenTests` for everything. Let the computer find edge cases.

### 90. Using `leanovate/gopter`
Randomly generate 1000s of inputs to break invariants.

```go
import (
    "github.com/leanovate/gopter"
    "github.com/leanovate/gopter/gen"
    "github.com/leanovate/gopter/prop"
)

func Test_DiscountEngine(t *testing.T) {
    parameters := gopter.DefaultTestParameters()
    parameters.MinSuccessfulTests = 1000 // Run 1000 times
    
    properties := gopter.NewProperties(parameters)
    
    properties.Property("Discount never exceeds total price", prop.ForAll(
        func(price float64, discount float64) bool {
            final := CalculatePrice(price, discount)
            return final >= 0 && final <= price
        },
        gen.Float64Range(1, 1000), // Price
        gen.Float64Range(0, 100),  // Discount %
    ))
    
    properties.TestingRun(t)
}
```

---

## Part 60: Contract Testing (Pact)

Stop breaking the frontend with backend changes.

### 91. Consumer-Driven Contracts
Verify your microservices actually work together.

```go
import "github.com/pact-foundation/pact-go/dsl"

// Service A (Consumer) Test
func TestLogin(t *testing.T) {
    pact := dsl.Pact{
        Consumer: "FrontendApp",
        Provider: "AuthService",
    }

    pact.
        AddInteraction().
        Given("User exists").
        UponReceiving("A login request").
        WithRequest(dsl.Request{
            Method: "POST",
            Path:   dsl.String("/login"),
        }).
        WillRespondWith(dsl.Response{
            Status: 200,
            Body:   dsl.Like(map[string]interface{}{"token": "x.y.z"}),
        })

    pact.Verify(t, func() error {
        // Run actual code against Mock Server
        return Login("http://localhost:" + strconv.Itoa(pact.Server.Port))
    })
}
```

---

## Part 61: AI Integration (LLMs)

Add GenAI capabilities to your Fiber endpoints.

### 92. Fiber + LangChainGo (Ollama)
Chat with local LLMs (Llama 3, Mistral) via Fiber.

```go
import (
    "github.com/tmc/langchaingo/llms/ollama"
    "github.com/tmc/langchaingo/schema"
)

func ChatHandler(c fiber.Ctx) error {
    prompt := c.FormValue("prompt")

    // 1. Connect to local Ollama instance
    llm, err := ollama.New(ollama.WithModel("llama3"))
    if err != nil { return c.Status(500).SendString(err.Error()) }

    // 2. Generate Response
    ctx := context.Background()
    completion, err := llm.Call(ctx, prompt)
    if err != nil { return c.Status(500).SendString(err.Error()) }

    return c.JSON(fiber.Map{"response": completion})
}

// Streaming Response (Token by Token)
func StreamChatHandler(c fiber.Ctx) error {
    llm, _ := ollama.New(ollama.WithModel("llama3"))
    
    return c.SendStream(func(w io.Writer) error {
        _, err := llm.Call(context.Background(), "Tell me a story", 
            llms.WithStreamingFunc(func(ctx context.Context, chunk []byte) error {
                w.Write(chunk) // Flush to HTTP Stream
                return nil
            }),
        )
        return err
    })
}
```

---

## Part 62: Network Performance Tuning

Optimize the raw TCP connection before Fiber even sees it.

### 93. Custom `net.ListenConfig`
Tune TCP KeepAlive and Buffers for high-throughput low-latency.

```go
func main() {
    app := fiber.New(fiber.Config{
        Prefork: true,
    })

    // Custom Listener Configuration
    lc := net.ListenConfig{
        KeepAlive: 30 * time.Second, // Aggressive KeepAlive
        Control:   func(network, address string, c syscall.RawConn) error {
            return c.Control(func(fd uintptr) {
                // Linux Optimizations (SO_REUSEPORT already handled by Prefork)
                // Set TCP User Timeout (adjust as needed via syscall)
                // syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_USER_TIMEOUT, 5000)
            })
        },
    }

    ln, err := lc.Listen(context.Background(), "tcp", ":3000")
    if err != nil { log.Fatal(err) }

    app.Listener(ln) // Attach custom listener
}
```

---

## Part 63: GPU/Compute Offloading

Don't block the Event Loop with heavy math.

### 94. CGO + CUDA Binding
Offload Matrix Multiplication to Nvidia GPU.

```go
/*
#cgo LDFLAGS: -L/usr/local/cuda/lib64 -lcudart
#include <cuda_runtime.h>
void gpu_matrix_mul(float* A, float* B, float* C, int N); // Defined in .cu file
*/
import "C"

func GPUHandler(c fiber.Ctx) error {
    // 1. Prepare Data
    // ... distribute matrices A and B ...

    // 2. Offload to C/CUDA (Blocking Call - run in goroutine if long)
    go func() {
        C.gpu_matrix_mul(...) 
    }()

    return c.SendString("Job submitted to GPU")
}
```
*Note: Requires `nvcc` and CUDA toolkit installed on the host.*

---

## Part 64: Distributed Rate Limiting (Lua)

Atomic sliding window rate limiting across a cluster using Redis + Lua.

### 95. The "Sliding Window" Script
More accurate than Fixed Window.

```go
// luaScript for Redis
const slidingWindowLua = `
local key = KEYS[1]
local window = tonumber(ARGV[1])
local limit = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

-- 1. Remove old timestamps
redis.call('ZREMRANGEBYSCORE', key, 0, now - window)

-- 2. Count Request history
local count = redis.call('ZCARD', key)

if count < limit then
    -- 3. Add new request
    redis.call('ZADD', key, now, now)
    redis.call('EXPIRE', key, window/1000 + 1)
    return 1 -- Allowed
else
    return 0 -- Blocked
end
`

func RateLimitMiddleware(redisClient *redis.Client) fiber.Handler {
    return func(c fiber.Ctx) error {
        key := "rate:" + c.IP()
        now := time.Now().UnixMilli()
        
        // Window: 60s, Limit: 100 reqs
        allowed, _ := redisClient.Eval(c.Context(), slidingWindowLua, []string{key}, 60000, 100, now).Bool()
        
        if !allowed {
            return c.Status(429).SendString("Too Many Requests")
        }
        return c.Next()
    }
}
```

---

## Part 65: Zero-Trust Security (mTLS)

Don't trust the network. Verify every service connection with certificates.

### 96. Mutual TLS (Client Cert Auth)
Configure Fiber to reject connections without a valid client certificate.

```go
func main() {
    // 1. Load CA responsible for signing client certs
    caCert, _ := os.ReadFile("ca.crt")
    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)

    // 2. Configure TLS
    tlsConfig := &tls.Config{
        ClientCAs:  caCertPool,
        ClientAuth: tls.RequireAndVerifyClientCert, // Enforce mTLS
    }

    // 3. Start Server with mTLS
    ln, _ := tls.Listen("tcp", ":443", tlsConfig)
    
    app := fiber.New()
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Secure Connection Established!")
    })
    
    app.Listener(ln)
}
```

---

## Part 66: Real-Time Updates (SSE)

Push updates to the browser without WebSockets.

### 97. Server-Sent Events (SSE)
One-way stream: Perfect for stock tickers, notifications, or progress bars.

```go
import "github.com/valyala/fasthttp"

func SSEHandler(c fiber.Ctx) error {
    c.Set("Content-Type", "text/event-stream")
    c.Set("Cache-Control", "no-cache")
    c.Set("Connection", "keep-alive")

    c.Context().SetBodyStreamWriter(func(w *bufio.Writer) {
        for {
            msg := fmt.Sprintf("data: %s\n\n", time.Now().String())
            fmt.Fprintf(w, msg)
            w.Flush()
            time.Sleep(1 * time.Second)
        }
    })

    return nil
}
```

---

## Part 67: GraphQL Subscriptions

Real-time GraphQL over WebSockets using `gqlgen`.

### 98. Websocket Transport Adapter
Connect Fiber to `gqlgen`'s websocket handler.

```go
import "github.com/99designs/gqlgen/graphql/handler/transport"

func GraphQLSubscriptionHandler(c fiber.Ctx) error {
    srv := handler.New(generated.NewExecutableSchema(generated.Config{Resolvers: &graph.Resolver{}}))
    
    // Add WebSocket Transport
    srv.AddTransport(&transport.Websocket{
        KeepAlivePingInterval: 10 * time.Second,
    })

    // Adapt to Fiber (fasthttp)
    adaptor.HTTPHandler(srv)(c)
    return nil
}
```

---

## Part 68: Decentralized Storage (IPFS)

Store files on the permanent web.

### 99. IPFS Kubo Client
Upload files to IPFS and retrieve the CID.

```go
import "github.com/ipfs/kubo/client/rpc"

func UploadToIPFS(c fiber.Ctx) error {
    // Connect to local IPFS node
    ipfs, _ := rpc.NewApiWithClient("localhost:5001", &http.Client{})

    file, _ := c.FormFile("document")
    f, _ := file.Open()
    defer f.Close()

    // Add to IPFS
    cid, _ := ipfs.Unixfs().Add(c.Context(), f)
    
    return c.JSON(fiber.Map{"cid": cid.String()})
}
```

---

## Part 69: Security Hardening (Helmet)

Secure your HTTP headers automatically.

### 100. Helmet Middleware
Set CSP, HSTS, X-Frame-Options, and more.

```go
import "github.com/gofiber/fiber/v3/middleware/helmet"

func main() {
    app := fiber.New()

    app.Use(helmet.New(helmet.Config{
        XSSProtection:             "1; mode=block",
        ContentTypeNosniff:        "nosniff",
        XFrameOptions:             "SAMEORIGIN",
        HSTSMaxAge:                31536000,
        HSTSExcludeSubdomains:     false,
        ContentSecurityPolicy:     "default-src 'self'",
        CSPReportOnly:             false,
    }))

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Secured via Helmet!")
    })

    app.Listen(":3000")
}
```

---

## Part 70: Interactive CLI Tools

Build admin tools for your Fiber app.

### 101. Using `AlecAivazis/survey`
Rich interactive prompts for admins.

```go
import "github.com/AlecAivazis/survey/v2"

func AdminCLI() {
    // 1. Define Questions
    var qs = []*survey.Question{
        {
            Name: "username",
            Prompt: &survey.Input{Message: "Admin Username:"},
        },
        {
            Name: "action",
            Prompt: &survey.Select{
                Message: "Choose Action:",
                Options: []string{"Backup DB", "Flush Cache", "Restart Server"},
            },
        },
    }

    // 2. Ask
    answers := struct {
        Username string
        Action   string
    }{}

    err := survey.Ask(qs, &answers)
    if err != nil { return }

    // 3. Execute
    fmt.Printf("User %s chose to %s\n", answers.Username, answers.Action)
    // Call Fiber internal API or Service layer here...
}
```

---

## Part 71: Audit Logging

Record *who* did *what* and *when*.

### 102. Audit Middleware
Log sensitive actions to a secure immutable log.

```go
type AuditLog struct {
    UserID    string    `json:"user_id"`
    Action    string    `json:"action"`
    Resource  string    `json:"resource"`
    Timestamp time.Time `json:"timestamp"`
    IP        string    `json:"ip"`
}

func AuditMiddleware(c fiber.Ctx) error {
    // Proceed with request
    if err := c.Next(); err != nil { return err }

    // Log only unsafe methods (POST, PUT, DELETE)
    if c.Method() != "GET" {
        logEntry := AuditLog{
            UserID:    c.Locals("user_id").(string),
            Action:    c.Method(),
            Resource:  c.Path(),
            Timestamp: time.Now(),
            IP:        c.IP(),
        }
        
        // Push filter to ElasticSearch / Splunk / DB
        go SaveAuditLog(logEntry)
    }
    return nil
}
```

---

## Part 72: Zero-Downtime Shutdown

Don't drop connections when you redeploy.

### 103. Graceful Shutdown
Handle SIGINT/SIGTERM to finish pending requests.

```go
import (
    "os"
    "os/signal"
    "syscall"
)

func main() {
    app := fiber.New()
    
    // Start Server in Goroutine
    go func() {
        if err := app.Listen(":3000"); err != nil {
            log.Panic(err)
        }
    }()

    // Wait for Interrupt Signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit 

    log.Println("Shutting down server...")

    // Shutdown with Timeout
    if err := app.ShutdownWithContext(context.Background()); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }

    log.Println("Server exited properly")
}
```

---

## Part 73: Distributed Transactions (Saga)

Manage consistency across microservices without ACID.

### 104. Saga Pattern with DTM
Coordinate multi-service transactions (e.g., Order -> Payment -> Inventory).

```go
// Using "github.com/dtm-labs/dtm/client/dtmcli"

func SubmitSaga(c fiber.Ctx) error {
    dtmServer := "http://localhost:36789/api/dtmsvr"
    gid := dtmcli.MustGenGid(dtmServer)
    
    // Define Saga
    saga := dtmcli.NewSaga(dtmServer, gid).
        // Step 1: Deduct Money (Compensate: Refund)
        Add(
            "http://localhost:3001/api/transOut", 
            "http://localhost:3001/api/transOutRevert", 
            &TransReq{Amount: 100},
        ).
        // Step 2: Add Inventory (Compensate: Release)
        Add(
            "http://localhost:3002/api/stockIn", 
            "http://localhost:3002/api/stockInRevert", 
            &StockReq{ItemID: 1, Qty: 1},
        )

    // Submit
    if err := saga.Submit(); err != nil {
        return c.Status(500).SendString(err.Error())
    }
    
    return c.JSON(fiber.Map{"gid": gid})
}
```
*Requires running DTM server.*

---

## Part 74: Profiling & Optimization (pprof)

Find memory leaks and CPU hotspots immediately.

### 105. Pprof Middleware
Expose standard Go profiling endpoints safely.

```go
import (
    "github.com/gofiber/fiber/v3/middleware/pprof"
)

func main() {
    app := fiber.New()

    // 1. Enable Pprof (Default: /debug/pprof/)
    app.Use(pprof.New())

    // 2. Load Test Logic (Simulate CPU load)
    app.Get("/cpu-hog", func(c fiber.Ctx) error {
        for i := 0; i < 1000000; i++ { _ = i * i }
        return c.SendString("Done")
    })

    app.Listen(":3000")
}
```
*Run: `go tool pprof http://localhost:3000/debug/pprof/profile`*

---

## Part 75: High-Performance Benchmarking

Don't guess. Measure your handler latency in nanoseconds.

### 106. Writing `testing.B` for Fiber
Benchmark direct handler execution without network overhead.

```go
// main_test.go
func Benchmark_OrderHandler(b *testing.B) {
    app := SetupApp() // Initializes Fiber
    
    req := httptest.NewRequest("GET", "/orders/123", nil)
    
    b.ResetTimer() // Reset setup time
    b.ReportAllocs()

    for i := 0; i < b.N; i++ {
        // Run request directly against app (Bypass TCP)
        resp, _ := app.Test(req, -1) 
        _ = resp.Body.Close()
    }
}
```
*Run: `go test -bench=. -benchmem`*

---

## Part 76: Advanced Middleware Composition

Dynamic chains and conditional logic.

### 107. Conditional Middleware (`Skip`)
Run Auth only for specific routes or methods.

```go
import "github.com/gofiber/fiber/v3/middleware/skip"

func main() {
    app := fiber.New()

    // 1. Logger: Runs everywhere EXCEPT /health
    app.Use(logger.New(logger.Config{
        Next: func(c fiber.Ctx) bool {
            return c.Path() == "/health"
        },
    }))

    // 2. Auth: Only runs on /admin/* prefix
    app.Use(skip.New(AuthMiddleware(), func(c fiber.Ctx) bool {
        return !strings.HasPrefix(c.Path(), "/admin")
    }))

    app.Listen(":3000")
}
```

---

## Part 77: gRPC Gateway

Use Fiber as the HTTP / REST entry point for your internal gRPC microservices.

### 108. Fiber -> gRPC Client
Translate JSON requests to Protobuf and call backend services.

```go
// proto/user.pb.go (Generated)

func CreateUserHandler(grpcClient pb.UserServiceClient) fiber.Handler {
    return func(c fiber.Ctx) error {
        // 1. Bind JSON Payload
        var dto CreateUserDTO
        if err := c.Bind().Body(&dto); err != nil {
            return err
        }

        // 2. Call gRPC Service
        // Map DTO -> Protobuf
        req := &pb.CreateUserRequest{
            Username: dto.Username,
            Email:    dto.Email,
        }

        resp, err := grpcClient.CreateUser(c.Context(), req)
        if err != nil {
            // Map gRPC status codes to HTTP
            return c.Status(500).SendString(err.Error())
        }

        // 3. Return Response
        return c.JSON(resp)
    }
}
```
*Note: Run efficient gRPC connection pooling in `main.go`.*

---

## Part 78: Secure Webhooks (HMAC)

Verify generic webhooks (Stripe, GitHub, Slack) securely.

### 109. HMAC Signature Verification Middleware
Reject payloads that weren't signed with your secret.

```go
import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
)

func VerifySignature(secret string) fiber.Handler {
    return func(c fiber.Ctx) error {
        // 1. Get Signature from Header (e.g., X-Hub-Signature-256)
        gotSig := c.Get("X-Signature")
        
        // 2. Compute Expected HMAC
        body := c.Body() // Raw Body
        mac := hmac.New(sha256.New, []byte(secret))
        mac.Write(body)
        expectedSig := hex.EncodeToString(mac.Sum(nil))

        // 3. Constant Time Comparison
        if !hmac.Equal([]byte(gotSig), []byte(expectedSig)) {
            return c.Status(401).SendString("Invalid Signature")
        }

        return c.Next()
    }
}
```

---

## Part 79: Localization (i18n)

Serve multiple languages from a single Fiber app.

### 110. Using `nicksnyder/go-i18n` with Fiber
Auto-detect language via `Accept-Language` header.

```go
import "github.com/nicksnyder/go-i18n/v2/i18n"

func main() {
    // 1. Load Bundles
    bundle := i18n.NewBundle(language.English)
    bundle.RegisterUnmarshalFunc("json", json.Unmarshal)
    bundle.LoadMessageFile("locales/en.json")
    bundle.LoadMessageFile("locales/es.json")

    app := fiber.New()

    app.Get("/", func(c fiber.Ctx) error {
        // 2. Detect Language
        lang := c.Get("Accept-Language")
        localizer := i18n.NewLocalizer(bundle, lang)

        // 3. Translate
        msg, _ := localizer.Localize(&i18n.LocalizeConfig{
            MessageID: "welcome_message",
            DefaultMessage: &i18n.Message{
                ID:    "welcome_message",
                Other: "Welcome to our API!",
            },
        })

        return c.JSON(fiber.Map{"message": msg})
    })
    
    app.Listen(":3000")
}
```

---

## Part 80: Cloud Storage Patterns (S3 Presigned)

Don't proxy file uploads. Let the client upload directly to S3.

### 111. Generating Presigned URLs
Securely authorize browser uploads without server load.

```go
import (
    "github.com/aws/aws-sdk-go-v2/service/s3"
)

func GenerateUploadURL(s3Client *s3.PresignClient) fiber.Handler {
    return func(c fiber.Ctx) error {
        filename := c.Query("filename")
        
        // 1. Generate Presigned PUT Request (Valid for 15 mins)
        req, err := s3Client.PresignPutObject(context.TODO(), &s3.PutObjectInput{
            Bucket: aws.String("my-bucket"),
            Key:    aws.String("uploads/" + filename),
        }, s3.WithPresignExpires(15*time.Minute))

        if err != nil { return c.SendStatus(500) }

        // 2. Return URL to Frontend
        return c.JSON(fiber.Map{
            "upload_url": req.URL,
            "method":     "PUT",
        })
    }
}
```
*Frontend: `fetch(upload_url, { method: 'PUT', body: file })`*

---

## Part 81: Production Deployment (Systemd)

Run Fiber as a robust background service on Linux.

### 112. Systemd Unit File
Auto-restart on failure, manage logs, and set environment limits.

```ini
# /etc/systemd/system/myapp.service

[Unit]
Description=My Fiber App
After=network.target

[Service]
User=www-data
Group=www-data
# Environment variables via file or inline
EnvironmentFile=/etc/myapp/.env
Environment=GO_ENV=production

# Binary path
ExecStart=/usr/local/bin/myapp
WorkingDirectory=/var/www/myapp

# Restart policy
Restart=always
RestartSec=3

# Security limits
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```
*Deploy: `systemctl enable --now myapp`*

---

## Part 82: Docker Optimization

Ship tiny, secure containers.

### 113. Multi-Stage Build (Distroless)
Separate build tools from runtime. Final image < 20MB.

```dockerfile
# Stage 1: Build
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
# CGO_ENABLED=0 for static binary
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

# Stage 2: Runtime (Distroless or Scratch)
# Distroless includes CA certs and tzdata
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/myapp /
CMD ["/myapp"]
```

---

## Part 83: Horizontal Scaling (Nginx)

Load balance across multiple Fiber instances (or Prefork processes).

### 114. Nginx Upstream Configuration
Distribute traffic round-robin to a cluster of Fiber nodes.

```nginx
# /etc/nginx/nginx.conf

http {
    upstream fiber_cluster {
        least_conn; # Send to least busy instance
        server 127.0.0.1:3001;
        server 127.0.0.1:3002;
        server 127.0.0.1:3003;
        keepalive 64;
    }

    server {
        listen 80;
        server_name api.example.com;

        location / {
            proxy_pass http://fiber_cluster;
            proxy_http_version 1.1;
            proxy_set_header Connection ""; # Important for KeepAlive
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

---

## Part 84: Infrastructure as Code (Ansible)

Deploy your binary to 100 servers in seconds.

### 115. Ansible Playbook for Go
Automate the binary upload and service restart.

```yaml
# deploy.yml
- hosts: webservers
  become: yes
  tasks:
    - name: Stop Service
      systemd:
        name: myapp
        state: stopped
      ignore_errors: yes

    - name: Upload Binary
      copy:
        src: ./bin/myapp-linux-amd64
        dest: /usr/local/bin/myapp
        mode: '0755'

    - name: Upload Config
      template:
        src: ./templates/myapp.service.j2
        dest: /etc/systemd/system/myapp.service
      notify: Reload Systemd

    - name: Start Service
      systemd:
        name: myapp
        state: started
        enabled: yes

  handlers:
    - name: Reload Systemd
      systemd:
        daemon_reload: yes
```

---

## Part 85: Kubernetes (K8s)

Orchestrate your Fiber containers at scale.

### 116. Deployment Manifest
Standard K8s configuration for a scalable Fiber service.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fiber-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fiber-app
  template:
    metadata:
      labels:
        app: fiber-app
    spec:
      containers:
      - name: fiber-app
        image: myregistry/fiber-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: GO_ENV
          value: "production"
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: fiber-service
spec:
  selector:
    app: fiber-app
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
```
*Apply: `kubectl apply -f deployment.yaml`*

---

## Part 86: Serverless (AWS Lambda)

Pay only for what you use. Run Fiber on Lambda.

### 117. AWS Lambda Adapter
Use `aws-lambda-go-api-proxy` to wrap Fiber.

```go
package main

import (
    "context"
    "github.com/aws/aws-lambda-go/events"
    "github.com/aws/aws-lambda-go/lambda"
    fiberadapter "github.com/awslabs/aws-lambda-go-api-proxy/fiber"
    "github.com/gofiber/fiber/v2"
)

var fiberLambda *fiberadapter.FiberLambda

func init() {
    app := fiber.New()
    app.Get("/", func(c *fiber.Ctx) error {
        return c.SendString("Hello from AWS Lambda!")
    })
    
    // Initialize Adapter
    fiberLambda = fiberadapter.New(app)
}

func Handler(ctx context.Context, req events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
    // Proxy request to Fiber
    return fiberLambda.ProxyWithContext(ctx, req)
}

func main() {
    lambda.Start(Handler)
}
```

---

## Part 87: Cross-Platform Compilation

Build binaries for every OS from your laptop.

### 118. Build Commands (`GOOS` / `GOARCH`)
Write once, run anywhere.

```bash
# Linux (x86_64) - Standard Server
GOOS=linux GOARCH=amd64 go build -o bin/server-linux main.go

# Windows (x86_64)
GOOS=windows GOARCH=amd64 go build -o bin/server.exe main.go

# macOS (Apple Silicon / M1/M2)
GOOS=darwin GOARCH=arm64 go build -o bin/server-mac-m1 main.go

# Raspberry Pi (ARM64)
GOOS=linux GOARCH=arm64 go build -o bin/server-rpi main.go

# Verify binary
file bin/server-linux
# Output: ELF 64-bit LSB executable, x86-64...
```

---

## Part 88: Desktop Apps (FiberWebGUI)

Turn your Fiber website into a native-ish Desktop App.

### 119. Fiber + WebView
Wraps your app in a Chrome window locally.

```go
import (
    "github.com/gofiber/fiber/v2"
    "github.com/toeverything/fiberwebgui"
)

func main() {
    app := fiber.New()
    
    // Serve your Frontend Artifacts
    app.Static("/", "./dist")
    
    app.Get("/api/hello", func(c *fiber.Ctx) error {
        return c.JSON(fiber.Map{"msg": "Hello from Desktop Backend!"})
    })

    // Start as Desktop App
    fiberwebgui.New(fiberwebgui.Config{
        App:       app,
        Address:   "localhost:3000",
        Width:     800,
        Height:    600,
        Title:     "My Fiber Desktop App",
    }).Run()
}
```
*Creates a window pointing to localhost:3000.*

---

## Part 89: Real-Time Analytics (ClickHouse)

Process billions of rows for your dashboard in milliseconds.

### 120. Using `clickhouse-go`
Insert high-volume event streams efficiently.

```go
import (
    "github.com/ClickHouse/clickhouse-go/v2"
    "github.com/ClickHouse/clickhouse-go/v2/lib/driver"
)

var chConn driver.Conn

func ConnectClickHouse() {
    var err error
    chConn, err = clickhouse.Open(&clickhouse.Options{
        Addr: []string{"127.0.0.1:9000"},
        Auth: clickhouse.Auth{
            Database: "analytics",
            Username: "default",
        },
    })
    if err != nil { panic(err) }
}

func TrackEvent(c fiber.Ctx) error {
    // 1. Async Batch Insert (Pattern)
    go func(path, ip string) {
        ctx := context.Background()
        // direct insert for simplicity; use batching in prod
        _ = chConn.Exec(ctx, `INSERT INTO pageviews (url, ip, created_at) VALUES (?, ?, now())`, path, ip)
    }(c.Path(), c.IP())

    return c.Next()
}
```
*Note: Use `chConn.PrepareBatch` for production high throughput.*

---

## Part 90: Full-Text Search (Elasticsearch)

Add google-like search to your app.

### 121. Indexing & Searching
Sync Fiber data to Elastic.

```go
import "github.com/elastic/go-elasticsearch/v8"

func SearchHandler(es *elasticsearch.Client) fiber.Handler {
    return func(c fiber.Ctx) error {
        query := c.Query("q")
        
        // 1. Build Search Body
        var buf bytes.Buffer
        searchQuery := map[string]interface{}{
            "query": map[string]interface{}{
                "multi_match": map[string]interface{}{
                    "query":  query,
                    "fields": []string{"title", "content"},
                },
            },
        }
        json.NewEncoder(&buf).Encode(searchQuery)

        // 2. Execute Search
        res, err := es.Search(
            es.Search.WithContext(c.Context()),
            es.Search.WithIndex("articles"),
            es.Search.WithBody(&buf),
        )
        if err != nil { return c.Status(500).SendString(err.Error()) }
        defer res.Body.Close()

        // 3. Pipe Raw JSON to Client
        c.Set("Content-Type", "application/json")
        return c.SendStream(res.Body)
    }
}
```

---

## Part 91: Graph Databases (Neo4j)

Model complex relationships (Social Networks, Recommendations).

### 122. Cypher Queries with Fiber
Execute graph traversal logic.

```go
import "github.com/neo4j/neo4j-go-driver/v5/neo4j"

func GetFriends(driver neo4j.DriverWithContext) fiber.Handler {
    return func(c fiber.Ctx) error {
        userID := c.Params("id")
        ctx := c.Context()

        session := driver.NewSession(ctx, neo4j.SessionConfig{AccessMode: neo4j.AccessModeRead})
        defer session.Close(ctx)

        // Run Cypher
        result, _ := session.Run(ctx, `
            MATCH (u:User {id: $id})-[:FRIEND]->(f:User)
            RETURN f.name AS name, f.email AS email
        `, map[string]interface{}{"id": userID})

        var friends []fiber.Map
        for result.Next(ctx) {
            rec := result.Record()
            name, _ := rec.Get("name")
            email, _ := rec.Get("email")
            friends = append(friends, fiber.Map{"name": name, "email": email})
        }

        return c.JSON(friends)
    }
}
```

---

## Part 92: Event Streaming (NATS JetStream)

Next-gen messaging for microservices. Persistence + replays.

### 123. Publish & Subscribe
Decouple services with a high-performance stream.

```go
import "github.com/nats-io/nats.go"

func SetupNATS() {
    nc, _ := nats.Connect(nats.DefaultURL)
    js, _ := nc.JetStream()

    // 1. Create Stream
    js.AddStream(&nats.StreamConfig{
        Name:     "ORDERS",
        Subjects: []string{"ORDERS.*"},
    })

    // 2. Publisher (Fiber Handler)
    app.Post("/orders", func(c fiber.Ctx) error {
        // Persisted publish
        _, err := js.Publish("ORDERS.created", c.Body())
        return err
    })

    // 3. Consumer (Worker)
    js.Subscribe("ORDERS.created", func(m *nats.Msg) {
        log.Printf("Processing Order: %s", string(m.Data))
        m.Ack() // Acknowledge
    })
}
```

---

## Part 93: Distributed Configuration (Etcd)

Manage runtime config across your cluster without redeploying.

### 124. Watch for Changes With Etcd
Update feature flags or limits dynamically.

```go
import clientv3 "go.etcd.io/etcd/client/v3"

func WatchConfig() {
    cli, _ := clientv3.New(clientv3.Config{Endpoints: []string{"localhost:2379"}})
    defer cli.Close()

    // 1. Initial Get
    resp, _ := cli.Get(context.Background(), "/myapp/config/max_users")
    UpdateLocalConfig(resp.Kvs[0].Value)

    // 2. Watch for Updates
    rch := cli.Watch(context.Background(), "/myapp/config/max_users")
    for wresp := range rch {
        for _, ev := range wresp.Events {
            log.Printf("Config updated: %s %q : %q\n", ev.Type, ev.Kv.Key, ev.Kv.Value)
            UpdateLocalConfig(ev.Kv.Value)
        }
    }
}
```

---

## Part 94: Service Discovery (Consul)

Let clients find you automatically.

### 125. Fiber Health Check for Consul
Expose an endpoint Consul can ping.

```go
import "github.com/gofiber/fiber/v2/middleware/healthcheck"

// 1. Register with Consul (at startup)
func RegisterService() {
    config := api.DefaultConfig()
    client, _ := api.NewClient(config)
    
    registration := &api.AgentServiceRegistration{
        ID:      "fiber-app-1",
        Name:    "fiber-app",
        Port:    3000,
        Address: "127.0.0.1",
        Check: &api.AgentServiceCheck{
            HTTP:     "http://127.0.0.1:3000/livez",
            Interval: "10s",
            Timeout:  "1s",
        },
    }
    client.Agent().ServiceRegister(registration)
}

func main() {
    app := fiber.New()
    
    // 2. Expose Health Check
    app.Use(healthcheck.New(healthcheck.Config{
        LivenessProbe: func(c *fiber.Ctx) bool {
            return true
        },
        LivenessEndpoint: "/livez",
    }))

    app.Listen(":3000")
}
```

---

## Part 95: Secrets Management (Vault)

Don't leak database passwords in env vars.

### 126. Read Secrets from HashiCorp Vault
Fetch dynamic credentials on startup.

```go
import "github.com/hashicorp/vault/api"

func GetDatabasePassword() string {
    config := api.DefaultConfig()
    client, _ := api.NewClient(config)
    client.SetToken("your-root-token") // In prod, use Kubernetes Auth

    // Read Secret
    secret, _ := client.Logical().Read("secret/data/myapp/database")
    data := secret.Data["data"].(map[string]interface{})
    
    return data["password"].(string)
}
```

---

## Part 96: Distributed Tracing (OpenTelemetry)

See the full path of a request across Microservices.

### 127. OpenTelemetry Middleware (`otelfiber`)
Auto-instrument every request with a Trace ID.

```go
import (
    "github.com/gofiber/contrib/otelfiber"
    "go.opentelemetry.io/otel"
)

func main() {
    // 1. Setup Tracer (Jaeger/Zipkin exporter)
    tp := initTracer() 
    defer tp.Shutdown(context.Background())
    otel.SetTracerProvider(tp)

    app := fiber.New()

    // 2. Use Middleware
    app.Use(otelfiber.Middleware())

    app.Get("/", func(c *fiber.Ctx) error {
        // Trace propagates to child functions automagically via Context
        return c.SendString("Traced Request")
    })

    app.Listen(":3000")
}
```
*Run Jaeger to visualize traces.*

---

## Part 97: WebAssemblyWithFiber (Client-Side)

Serve Go-based frontends (Wasm) from your Fiber backend.

### 128. Serving the Wasm Frontend
Fiber hosts the `wasm_exec.js` and `.wasm` binary.

```go
func main() {
    app := fiber.New()

    // 1. Serve Static Assets (index.html, wasm_exec.js)
    app.Static("/", "./client/dist")

    // 2. Serve WASM Binary with Correct MIME
    app.Get("/main.wasm", func(c *fiber.Ctx) error {
        c.Set("Content-Type", "application/wasm")
        return c.SendFile("./client/dist/main.wasm")
    })

    app.Listen(":3000")
}
```
*Client Build: `GOOS=js GOARCH=wasm go build -o client/dist/main.wasm client/main.go`*

---

## Part 98: Advanced Generics (Go 1.18+)

Type-safe response wrappers for Fiber v2/v3.

### 129. Generic JSON Response Wrapper
Standardize API responses without `interface{}`.

```go
type APIResponse[T any] struct {
    Success bool   `json:"success"`
    Data    T      `json:"data"`
    Error   string `json:"error,omitempty"`
}

func SendSuccess[T any](c *fiber.Ctx, data T) error {
    return c.JSON(APIResponse[T]{
        Success: true,
        Data:    data,
    })
}

// Usage
func GetUser(c *fiber.Ctx) error {
    user := User{Name: "Alice"}
    return SendSuccess(c, user) 
    // Compiler ensures types are correct!
}
```

---

## Part 99: The Zen of Fiber (Best Practices)

A summary of the 100-part journey.

### 130. Core Tenets
1. **Prefork in Production**: Use `Prefork: true` for 10x throughput on multi-core.
2. **Zero Allocation**: Reuse memory (e.g., `c.Locals`, `sync.Pool`) where possible.
3. **Structured Logging**: Always use `zerolog` or `slog` behind an interface.
4. **graceful Shutdown**: Never kill connections on deploy (`app.ShutdownWithContext`).
5. **Security First**: Helmet, Rate Limiter, and CORS are mandatory, not optional.

---

## Part 100: The Future (HTTP/3 & QUIC)

Preparing for the next generation of the web.

### 131. HTTP/3 Readiness
While Fiber (Fasthttp) waits for native HTTP/3, prepare via Proxy.

```nginx
# Nginx / Cloudflare acting as HTTP/3 Gateway
server {
    listen 443 quic reuseport;
    listen 443 ssl http2;
    
    server_name api.example.com;
    
    # Forward to Fiber (HTTP/1.1 or HTTP/2)
    location / {
        proxy_pass http://127.0.0.1:3000;
        add_header Alt-Svc 'h3=":443"; ma=86400';
    }
}
```
*Keep an eye on `gofiber/fiber/v3` roadmap for native QUIC integration.*

---

## Part 101: Goroutines in Handlers

Don't block the request. Offload work safely.

### 132. Fire-and-Forget
Spawn a goroutine for non-critical tasks (logs, metrics).

```go
app.Post("/signup", func(c *fiber.Ctx) error {
    // 1. Capture data needed for goroutine (Strings are safe, Ctx is NOT)
    email := c.FormValue("email")
    
    // 2. Spawn Goroutine
    go func(userEmail string) {
        // DO NOT use 'c' here! It will be recycled.
        SendWelcomeEmail(userEmail)
    }(email)

    return c.SendString("Signup processing...")
})
```

---

## Part 102: Channels & Worker Pools

Control concurrency limits.

### 133. Buffered Channels
Queue jobs without blocking until full.

```go
// 1. Work Queue
type Job struct {
    ReqID string
    Data  []byte
}
var jobQueue = make(chan Job, 100) // Buffer 100 jobs

func StartWorkerPool() {
    for i := 0; i < 5; i++ { // 5 Workers
        go func(id int) {
            for job := range jobQueue {
                ProcessJob(id, job)
            }
        }(i)
    }
}

app.Post("/process", func(c *fiber.Ctx) error {
    select {
    case jobQueue <- Job{ReqID: c.Get("X-Request-ID"), Data: c.Body()}:
        return c.Accepted()
    default:
        return c.Status(503).SendString("Queue Full")
    }
})
```

---

## Part 103: Mutex & Sync

Thread-safe state management.

### 134. RWMutex for Shared Cache
Safe concurrent reads and writes.

```go
import "sync"

type SafeCounter struct {
    mu    sync.RWMutex
    count int
}

func (s *SafeCounter) Inc() {
    s.mu.Lock()
    defer s.mu.Unlock()
    s.count++
}

func (s *SafeCounter) Get() int {
    s.mu.RLock() // Multiple readers allowed
    defer s.mu.RUnlock()
    return s.count
}

var visits SafeCounter

app.Use(func(c *fiber.Ctx) error {
    visits.Inc()
    return c.Next()
})
```

---

## Part 104: The Scheduler (Ticker/Cron)

Run recurring internal tasks.

### 135. Time.Ticker
Simple periodic execution.

```go
func StartScheduler() {
    ticker := time.NewTicker(1 * time.Minute)
    go func() {
        for range ticker.C {
            // Task: Cleanup expired sessions
            CleanCache()
        }
    }()
}
```

---

## Part 105: WaitGroups

Parallel processing with join.

### 136. Fetch Multiple APIs
Run in parallel, wait for all to finish.

```go
app.Get("/dashboard", func(c *fiber.Ctx) error {
    var wg sync.WaitGroup
    var userData User
    var orders []Order

    wg.Add(2)

    go func() {
        defer wg.Done()
        userData = FetchUser()
    }()

    go func() {
        defer wg.Done()
        orders = FetchOrders()
    }()

    wg.Wait() // Block until both APIs return

    return c.JSON(fiber.Map{
        "user":   userData,
        "orders": orders,
    })
})
```

---

## Part 106: Blueprint: E-Commerce API

A Hexagonal Architecture layout for specific domains.

### 137. Domain Design
Directory structure for `github.com/my/shop`:

yes```text
/cmd/api/main.go
/internal
  /domain       (Interfaces & Entities)
  /service      (Business Logic)
  /repository   (Database Access)
  /handler      (Fiber Handlers)
```

### 138. Product Service Implementation
Decoupled business logic.

```go
// internal/domain/product.go
type Product struct {
    ID    int     `json:"id"`
    Name  string  `json:"name"`
    Price float64 `json:"price"`
}

type ProductRepository interface {
    GetByID(ctx context.Context, id int) (*Product, error)
}

// internal/service/product_service.go
type ProductService struct {
    repo domain.ProductRepository
}

func (s *ProductService) GetProduct(ctx context.Context, id int) (*domain.Product, error) {
    if id <= 0 { return nil, errors.New("invalid id") }
    return s.repo.GetByID(ctx, id)
}
```

---

## Part 107: Blueprint: Real-Time Chat

Scalable WebSocket Hub with Redis Pub/Sub.

### 139. The Hub
Manage active connections and broadcast.

```go
type Hub struct {
    clients    map[*Client]bool
    broadcast  chan []byte
    register   chan *Client
    unregister chan *Client
}

func (h *Hub) Run() {
    for {
        select {
        case client := <-h.register:
            h.clients[client] = true
        case client := <-h.unregister:
            if _, ok := h.clients[client]; ok {
                delete(h.clients, client)
                client.conn.Close()
            }
        case message := <-h.broadcast:
            // Send to all connected clients
            for client := range h.clients {
                client.send <- message
            }
        }
    }
}
```

### 140. Redis Adapter
Scale across multiple servers.

```go
func SubscribeRedis(h *Hub, redisClient *redis.Client) {
    pubsub := redisClient.Subscribe(ctx, "chat_room_general")
    ch := pubsub.Channel()

    go func() {
        for msg := range ch {
            h.broadcast <- []byte(msg.Payload)
        }
    }()
}
```

---

## Part 108: Blueprint: URL Shortener

High-performance redirector (Redis-backed).

### 141. Shorten Logic
Base62 encoding and Redis storage.

```go
func ShortenURL(c *fiber.Ctx) error {
    type Req struct { URL string `json:"url"` }
    var req Req
    if err := c.BodyParser(&req); err != nil { return c.SendStatus(400) }

    // 1. Generate ID (Snowflake or Random)
    id := GenerateID() 
    shortCode := Base62Encode(id)

    // 2. Store in Redis
    // Key: short:abcde -> https://google.com
    RedisClient.Set(ctx, "short:"+shortCode, req.URL, 24*time.Hour)

    // 3. Init Analytics
    RedisClient.Set(ctx, "stats:"+shortCode, 0, 24*time.Hour)

    return c.JSON(fiber.Map{"short_url": "http://lo.cal/" + shortCode})
}
```

### 142. Redirect & Track
Zero-latency redirection.

```go
func Redirect(c *fiber.Ctx) error {
    code := c.Params("code")
    
    // 1. Fast Lookup
    originalURL, err := RedisClient.Get(ctx, "short:"+code).Result()
    if err == redis.Nil { return c.SendStatus(404) }

    // 2. Async Analytics (Fire-and-forget)
    go func() {
        RedisClient.Incr(context.Background(), "stats:"+code)
    }()

    return c.Redirect(originalURL, 301)
}
```

---

## Part 109: Common Errors & Fixes

Solve the errors that keep you up at night.

### 143. "Request Entity Too Large" (413)
Fiber's default body limit is 4MB.

```go
// Fix: Increase globally
app := fiber.New(fiber.Config{
    BodyLimit: 50 * 1024 * 1024, // 50MB
})
```

### 144. "Concurrent Map Writes"
You are writing to `fiber.Ctx` or a global map from a goroutine.

```go
// BAD:
go func() { c.Locals("key", "val") }() 

// GOOD:
ctx := c.Context() // Copy context if needed (less common for Locals)
// Or use a sync.Map / mutex for global state
```

---

## Part 110: Debugging Performance (pprof)

Find memory leaks and CPU spikes.

### 145. Live Profiling
Attach the pprof middleware.

```go
import "github.com/gofiber/fiber/v2/middleware/pprof"

func main() {
    app := fiber.New()
    app.Use(pprof.New()) // Adds /debug/pprof routes
    
    app.Listen(":3000")
}
```
*Analyze:* `go tool pprof http://localhost:3000/debug/pprof/heap`

---

## Part 111: Migration Guide (v2 -> v3)

Major breaking changes to watch for.

### 146. Key Changes Checklist
1.  **Go Version**: v3 requires Go 1.25+.
2.  **`app.Listen`**: Configuration moved to `fiber.Config`.
3.  **Packages**: `gofiber/utils` and `gofiber/adaptor` merged or refactored.
4.  **Loop Variable**: Go 1.22+ semantics apply (no more `id := id` inside loops).
5.  **Hooks**: Use `OnAndHooks` for robust startup/shutdown logic.

### 147. Middleware Signature
Context is now an interface (in some v3 betas) or refactored struct.

```go
// v2
func(c *fiber.Ctx) error

// v3 (Check latest RC)
// Often remains similar, but internal methods changed.
```

---

## Part 112: The FAQ

Quick answers to persistent questions.

### 148. "Can I use `net/http` middleware?"
Yes, use `adaptor.HTTPMiddleware`.

### 149. "How do I handle CORS?"
Always use `middleware/cors` as the **first** middleware.

### 150. "Why is my websocket disconnecting?"
Check your Nginx/LoadBalancer timeout settings (default 60s). Send PING/PONG frames.

---

## Part 113: Role-Playing Prompts (for LLMs)

Use these prompts to prime an AI with this document.

### 151. The "Fiber Architect" Persona
*Paste this at the start of your chat context:*

> "You are the **Fiber Architect**, a specialized AI assistant. You have digested the '1000 Series' Go Fiber documentation. Your goal is to provide production-grade, zero-allocation code solutions.
>
> **Rules:**
> 1. Always prefer `gofiber/fiber/v3` syntax.
> 2. Enforce 'Clean Architecture' (Handler -> Service -> Repository).
> 3. Use `ctx.Context()` when spawning goroutines to avoid data races.
> 4. Suggest 'Prefork' and 'goccy/go-json' for performance queries.
> 5. If I ask for a 'Blueprint', provide the full file structure."

---

## Part 114: RAG Keywords & Tags

Optimization for Vector Databases and Search.

### 152. Meta Tags
`#gofiber #golang #high-performance #web-framework #rest-api #websocket #zero-allocation #clean-architecture #ddd #hexagonal #kubernetes #docker #aws-lambda #middleware #security #authentication #jwt #oauth2 #database #gorm #sqlc #mongodb #redis #caching #observability #opentelemetry #prometheus #grafana`

### 153. Semantic Anchors
- **Performance:** `prefork`, `goccy/go-json`, `fasthttp`, `zero-copy`, `byte-buffer`
- **Security:** `helmet`, `cors`, `csrf`, `rate-limiter`, `mtls`, `vault`
- **Architecture:** `dependency-injection`, `uber-fx`, `repository-pattern`, `domain-driven-design`


















