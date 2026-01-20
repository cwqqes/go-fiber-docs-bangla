# ডিপ্লয়মেন্ট (Deployment)

## Production Config

```go
package main

import (
    "log"
    "os"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/compress"
    "github.com/gofiber/fiber/v3/middleware/cors"
    "github.com/gofiber/fiber/v3/middleware/helmet"
    "github.com/gofiber/fiber/v3/middleware/limiter"
    "github.com/gofiber/fiber/v3/middleware/logger"
    "github.com/gofiber/fiber/v3/middleware/recover"
)

func main() {
    // Production config
    app := fiber.New(fiber.Config{
        // Performance
        Prefork:       true,  // Multiple processes
        ServerHeader:  "",    // Hide server header
        StrictRouting: true,
        CaseSensitive: true,
        
        // Limits
        BodyLimit:     4 * 1024 * 1024, // 4MB
        ReadTimeout:   10 * time.Second,
        WriteTimeout:  10 * time.Second,
        IdleTimeout:   120 * time.Second,
        
        // Error handling
        ErrorHandler: func(c fiber.Ctx, err error) error {
            code := fiber.StatusInternalServerError
            if e, ok := err.(*fiber.Error); ok {
                code = e.Code
            }
            return c.Status(code).JSON(fiber.Map{
                "error": err.Error(),
            })
        },
    })

    // Security middleware
    app.Use(recover.New())
    app.Use(helmet.New())
    app.Use(cors.New(cors.Config{
        AllowOrigins: []string{os.Getenv("ALLOWED_ORIGINS")},
        AllowMethods: []string{"GET", "POST", "PUT", "DELETE"},
    }))
    
    // Rate limiting
    app.Use(limiter.New(limiter.Config{
        Max:        100,
        Expiration: 1 * time.Minute,
    }))
    
    // Compression
    app.Use(compress.New(compress.Config{
        Level: compress.LevelBestSpeed,
    }))
    
    // Logging
    app.Use(logger.New(logger.Config{
        Format: "${time} | ${status} | ${latency} | ${ip} | ${method} | ${path}\n",
        Output: os.Stdout,
    }))

    // Routes
    setupRoutes(app)

    // Graceful shutdown
    port := os.Getenv("PORT")
    if port == "" {
        port = "3000"
    }
    
    log.Fatal(app.Listen(":" + port))
}
```

## Environment Variables

```go
package config

import (
    "log"
    "os"
    "github.com/joho/godotenv"
)

type Config struct {
    Port        string
    DatabaseURL string
    JWTSecret   string
    RedisURL    string
    Environment string
}

func Load() *Config {
    if os.Getenv("ENVIRONMENT") != "production" {
        if err := godotenv.Load(); err != nil {
            log.Println("No .env file found")
        }
    }

    return &Config{
        Port:        getEnv("PORT", "3000"),
        DatabaseURL: getEnv("DATABASE_URL", ""),
        JWTSecret:   getEnv("JWT_SECRET", ""),
        RedisURL:    getEnv("REDIS_URL", ""),
        Environment: getEnv("ENVIRONMENT", "development"),
    }
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

func (c *Config) IsProduction() bool {
    return c.Environment == "production"
}
```

## Dockerfile

```dockerfile
# Build stage
FROM golang:1.22-alpine AS builder

WORKDIR /app

# Dependencies
COPY go.mod go.sum ./
RUN go mod download

# Build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o main .

# Runtime stage
FROM alpine:3.19

RUN apk --no-cache add ca-certificates tzdata

WORKDIR /app

COPY --from=builder /app/main .

# Non-root user
RUN adduser -D -g '' appuser
USER appuser

EXPOSE 3000

CMD ["./main"]
```

## Docker Compose

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - PORT=3000
      - DATABASE_URL=postgres://user:pass@db:5432/myapp?sslmode=disable
      - REDIS_URL=redis:6379
      - JWT_SECRET=${JWT_SECRET}
      - ENVIRONMENT=production
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - app

volumes:
  postgres_data:
  redis_data:
```

## Nginx Config

```nginx
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    # Basic settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # Gzip
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

    # Upstream
    upstream fiber_app {
        least_conn;
        server app:3000;
        keepalive 32;
    }

    # HTTP redirect
    server {
        listen 80;
        server_name example.com;
        return 301 https://$server_name$request_uri;
    }

    # HTTPS
    server {
        listen 443 ssl http2;
        server_name example.com;

        ssl_certificate /etc/nginx/certs/cert.pem;
        ssl_certificate_key /etc/nginx/certs/key.pem;
        ssl_protocols TLSv1.2 TLSv1.3;

        location / {
            limit_req zone=api burst=20 nodelay;
            
            proxy_pass http://fiber_app;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
        }

        location /health {
            proxy_pass http://fiber_app/health;
        }
    }
}
```

## Health Check

```go
package main

import (
    "context"
    "time"
    "github.com/gofiber/fiber/v3"
    "gorm.io/gorm"
    "github.com/redis/go-redis/v9"
)

type HealthChecker struct {
    DB    *gorm.DB
    Redis *redis.Client
}

func (h *HealthChecker) Check(c fiber.Ctx) error {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    status := fiber.Map{
        "status": "healthy",
        "checks": fiber.Map{},
    }

    // Database check
    sqlDB, _ := h.DB.DB()
    if err := sqlDB.PingContext(ctx); err != nil {
        status["status"] = "unhealthy"
        status["checks"].(fiber.Map)["database"] = "down"
    } else {
        status["checks"].(fiber.Map)["database"] = "up"
    }

    // Redis check
    if err := h.Redis.Ping(ctx).Err(); err != nil {
        status["status"] = "unhealthy"
        status["checks"].(fiber.Map)["redis"] = "down"
    } else {
        status["checks"].(fiber.Map)["redis"] = "up"
    }

    if status["status"] == "unhealthy" {
        return c.Status(503).JSON(status)
    }

    return c.JSON(status)
}

// Kubernetes probes
func (h *HealthChecker) LivenessProbe(c fiber.Ctx) error {
    return c.SendString("OK")
}

func (h *HealthChecker) ReadinessProbe(c fiber.Ctx) error {
    sqlDB, _ := h.DB.DB()
    if err := sqlDB.Ping(); err != nil {
        return c.Status(503).SendString("Not Ready")
    }
    return c.SendString("Ready")
}
```

## Graceful Shutdown

```go
package main

import (
    "log"
    "os"
    "os/signal"
    "syscall"
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    // Routes
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello!")
    })

    // Graceful shutdown
    go func() {
        if err := app.Listen(":3000"); err != nil {
            log.Panic(err)
        }
    }()

    quit := make(chan os.Signal, 1)
    signal.Notify(quit, os.Interrupt, syscall.SIGTERM)
    <-quit

    log.Println("Shutting down server...")

    if err := app.ShutdownWithTimeout(30 * time.Second); err != nil {
        log.Fatalf("Server forced to shutdown: %v", err)
    }

    log.Println("Server exited properly")
}
```

## Systemd Service

```ini
# /etc/systemd/system/fiber-app.service
[Unit]
Description=Fiber App
After=network.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/fiber-app
ExecStart=/opt/fiber-app/main
Restart=on-failure
RestartSec=5
Environment=PORT=3000
Environment=ENVIRONMENT=production
EnvironmentFile=/opt/fiber-app/.env

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable fiber-app
sudo systemctl start fiber-app
```

## Logging

```go
package main

import (
    "os"
    "github.com/gofiber/fiber/v3"
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
)

func setupLogger(env string) {
    if env == "development" {
        log.Logger = log.Output(zerolog.ConsoleWriter{Out: os.Stdout})
    } else {
        zerolog.TimeFieldFormat = zerolog.TimeFormatUnix
        log.Logger = zerolog.New(os.Stdout).With().Timestamp().Logger()
    }
}

func LoggerMiddleware() fiber.Handler {
    return func(c fiber.Ctx) error {
        start := time.Now()
        
        err := c.Next()
        
        log.Info().
            Str("method", c.Method()).
            Str("path", c.Path()).
            Int("status", c.Response().StatusCode()).
            Dur("latency", time.Since(start)).
            Str("ip", c.IP()).
            Msg("request")
        
        return err
    }
}
```

## Monitoring (Prometheus)

```go
package main

import (
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/adaptor"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    requestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        },
        []string{"method", "path", "status"},
    )

    requestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "path"},
    )
)

func init() {
    prometheus.MustRegister(requestsTotal)
    prometheus.MustRegister(requestDuration)
}

func MetricsMiddleware() fiber.Handler {
    return func(c fiber.Ctx) error {
        start := time.Now()
        
        err := c.Next()
        
        duration := time.Since(start).Seconds()
        status := strconv.Itoa(c.Response().StatusCode())
        
        requestsTotal.WithLabelValues(c.Method(), c.Path(), status).Inc()
        requestDuration.WithLabelValues(c.Method(), c.Path()).Observe(duration)
        
        return err
    }
}

func main() {
    app := fiber.New()

    app.Use(MetricsMiddleware())

    app.Get("/metrics", adaptor.HTTPHandler(promhttp.Handler()))

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello!")
    })

    app.Listen(":3000")
}
```

## CI/CD (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-go@v5
        with:
          go-version: '1.22'
      
      - name: Test
        run: go test -v ./...

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: username/fiber-app:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            docker pull username/fiber-app:latest
            docker-compose up -d
```

## Kubernetes

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
        image: username/fiber-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: PORT
          value: "3000"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        livenessProbe:
          httpGet:
            path: /health/live
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 10
        resources:
          limits:
            cpu: "500m"
            memory: "128Mi"
          requests:
            cpu: "250m"
            memory: "64Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: fiber-app
spec:
  selector:
    app: fiber-app
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

## সারসংক্ষেপ

| Topic | Tool/Method |
|-------|-------------|
| Build | Docker, Go build |
| Reverse Proxy | Nginx, Caddy |
| Process Manager | Systemd, PM2 |
| Container | Docker Compose, K8s |
| CI/CD | GitHub Actions, GitLab CI |
| Monitoring | Prometheus, Grafana |
| Logging | Zerolog, Zap |

---

[← আগের: ওয়েবসকেট](05-websockets.md) | [🏠 হোম](../README.md)
