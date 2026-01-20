# থার্ড-পার্টি মিডলওয়্যার

Fiber এর জন্য অনেক থার্ড-পার্টি মিডলওয়্যার রয়েছে যা gofiber/contrib প্যাকেজে পাওয়া যায়।

## ইনস্টল করুন

```bash
go get github.com/gofiber/contrib/...
```

## JWT - JSON Web Token

```go
import "github.com/gofiber/contrib/jwt"

app.Use("/api", jwtware.New(jwtware.Config{
    SigningKey: jwtware.SigningKey{Key: []byte("secret")},
}))

// টোকেন জেনারেট করুন
import "github.com/golang-jwt/jwt/v5"

func login(c fiber.Ctx) error {
    // ইউজার ভেরিফাই করুন...
    
    claims := jwt.MapClaims{
        "user_id": "123",
        "name":    "Rahim",
        "exp":     time.Now().Add(time.Hour * 24).Unix(),
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    tokenString, _ := token.SignedString([]byte("secret"))
    
    return c.JSON(fiber.Map{
        "token": tokenString,
    })
}

// টোকেন থেকে ইউজার পান
app.Get("/profile", func(c fiber.Ctx) error {
    user := c.Locals("user").(*jwt.Token)
    claims := user.Claims.(jwt.MapClaims)
    name := claims["name"].(string)
    
    return c.SendString("স্বাগতম " + name)
})
```

## Swagger - API ডকুমেন্টেশন

```go
import "github.com/gofiber/contrib/swagger"

// @title My API
// @version 1.0
// @description API ডকুমেন্টেশন
// @host localhost:3000
// @BasePath /api

app.Use(swagger.New(swagger.Config{
    BasePath: "/api",
    FilePath: "./docs/swagger.json",
    Path:     "docs",
}))
```

## Otelfiber - OpenTelemetry

```go
import "github.com/gofiber/contrib/otelfiber"

// ট্রেসিং সেটআপ
tp := initTracer()
defer tp.Shutdown(context.Background())

app.Use(otelfiber.Middleware())

app.Get("/users/:id", func(c fiber.Ctx) error {
    // স্প্যান তৈরি করুন
    _, span := tracer.Start(c.UserContext(), "getUser")
    defer span.End()
    
    return c.SendString("User data")
})
```

## Fiberzap - Zap Logger

```go
import (
    "github.com/gofiber/contrib/fiberzap/v2"
    "go.uber.org/zap"
)

logger, _ := zap.NewProduction()

app.Use(fiberzap.New(fiberzap.Config{
    Logger: logger,
}))
```

## Fiberzerolog - Zerolog

```go
import (
    "github.com/gofiber/contrib/fiberzerolog"
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
)

logger := zerolog.New(os.Stderr).With().Timestamp().Logger()

app.Use(fiberzerolog.New(fiberzerolog.Config{
    Logger: &logger,
}))
```

## Fibersentry - Sentry Error Tracking

```go
import (
    "github.com/getsentry/sentry-go"
    "github.com/gofiber/contrib/fibersentry"
)

sentry.Init(sentry.ClientOptions{
    Dsn: "your-sentry-dsn",
})

app.Use(fibersentry.New(fibersentry.Config{
    Repanic: true,
}))

app.Get("/error", func(c fiber.Ctx) error {
    // এরর Sentry-তে পাঠান
    if hub := fibersentry.GetHubFromContext(c); hub != nil {
        hub.CaptureMessage("Something went wrong")
    }
    return c.SendString("Error logged")
})
```

## PASETO - Platform-Agnostic Security Tokens

```go
import "github.com/gofiber/contrib/paseto"

app.Use(pasetoware.New(pasetoware.Config{
    SymmetricKey: []byte("your-32-byte-secret-key-here!!!"),
    TokenLookup:  "header:Authorization",
}))

// টোকেন পান
app.Get("/protected", func(c fiber.Ctx) error {
    payload := pasetoware.FromContext(c)
    return c.JSON(payload)
})
```

## Casbin - Authorization

```go
import (
    "github.com/gofiber/contrib/casbin"
    "github.com/casbin/casbin/v2"
)

enforcer, _ := casbin.NewEnforcer("model.conf", "policy.csv")

app.Use(casbin.New(casbin.Config{
    Enforcer: enforcer,
    Lookup: func(c fiber.Ctx) string {
        return c.Locals("userId").(string)
    },
}))
```

## New Relic - APM

```go
import "github.com/gofiber/contrib/fibernewrelic"

cfg := fibernewrelic.Config{
    License: "your-new-relic-license",
    AppName: "My Fiber App",
    Enabled: true,
}

app.Use(fibernewrelic.New(cfg))
```

## WebSocket

```go
import "github.com/gofiber/contrib/websocket"

app.Use("/ws", func(c fiber.Ctx) error {
    if websocket.IsWebSocketUpgrade(c) {
        return c.Next()
    }
    return fiber.ErrUpgradeRequired
})

app.Get("/ws/:id", websocket.New(func(c *websocket.Conn) {
    for {
        mt, msg, err := c.ReadMessage()
        if err != nil {
            break
        }
        
        log.Printf("মেসেজ পেয়েছি: %s", msg)
        
        if err := c.WriteMessage(mt, msg); err != nil {
            break
        }
    }
}))
```

## gRPC Gateway

```go
import "github.com/gofiber/contrib/fibergrpc"

// gRPC সার্ভার কানেক্ট করুন
conn, _ := grpc.Dial("localhost:50051", grpc.WithInsecure())

app.Use("/grpc", fibergrpc.New(fibergrpc.Config{
    Connection: conn,
}))
```

## i18n - ইন্টারন্যাশনালাইজেশন

```go
import "github.com/gofiber/contrib/fiberi18n/v2"

app.Use(fiberi18n.New(&fiberi18n.Config{
    RootPath:        "./locales",
    DefaultLanguage: language.Bengali,
    AcceptLanguages: []language.Tag{language.Bengali, language.English},
}))

app.Get("/", func(c fiber.Ctx) error {
    msg := fiberi18n.MustLocalize(c, &i18n.LocalizeConfig{
        MessageID: "welcome",
    })
    return c.SendString(msg)
})
```

## HCaptcha - Bot Protection

```go
import "github.com/gofiber/contrib/hcaptcha"

captcha := hcaptcha.New(hcaptcha.Config{
    SecretKey: "your-secret-key",
})

app.Post("/submit", captcha, func(c fiber.Ctx) error {
    return c.SendString("ফর্ম সাবমিট সফল")
})
```

## Circuit Breaker

```go
import "github.com/gofiber/contrib/circuitbreaker"

cb := circuitbreaker.New(circuitbreaker.Config{
    FailureThreshold: 3,
    Timeout:          5 * time.Second,
    SuccessThreshold: 2,
})

app.Use(circuitbreaker.Middleware(cb))

// হেলথ চেক
app.Get("/health/circuit", cb.HealthHandler())
```

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "log"
    "time"
    
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/contrib/jwt"
    "github.com/gofiber/contrib/fiberzap/v2"
    "go.uber.org/zap"
)

func main() {
    app := fiber.New()
    
    // লগার
    logger, _ := zap.NewProduction()
    app.Use(fiberzap.New(fiberzap.Config{
        Logger: logger,
    }))
    
    // পাবলিক রাউট
    app.Post("/login", loginHandler)
    app.Post("/register", registerHandler)
    
    // প্রোটেক্টেড রাউট
    api := app.Group("/api")
    api.Use(jwtware.New(jwtware.Config{
        SigningKey: jwtware.SigningKey{Key: []byte("secret")},
    }))
    
    api.Get("/profile", profileHandler)
    api.Get("/orders", ordersHandler)
    
    log.Fatal(app.Listen(":3000"))
}

func loginHandler(c fiber.Ctx) error {
    // লগইন লজিক
    return c.JSON(fiber.Map{"token": "jwt-token"})
}

func registerHandler(c fiber.Ctx) error {
    // রেজিস্ট্রেশন লজিক
    return c.JSON(fiber.Map{"message": "নিবন্ধন সফল"})
}

func profileHandler(c fiber.Ctx) error {
    user := c.Locals("user")
    return c.JSON(fiber.Map{"user": user})
}

func ordersHandler(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"orders": []string{}})
}
```

## থার্ড-পার্টি প্যাকেজ লিস্ট

| প্যাকেজ | ব্যবহার |
|---------|---------|
| jwt | JWT অথেনটিকেশন |
| swagger | API ডকুমেন্টেশন |
| otelfiber | OpenTelemetry ট্রেসিং |
| fiberzap | Zap লগিং |
| fiberzerolog | Zerolog লগিং |
| fibersentry | Sentry এরর ট্র্যাকিং |
| paseto | PASETO টোকেন |
| casbin | অথরাইজেশন |
| fibernewrelic | New Relic APM |
| websocket | WebSocket সাপোর্ট |
| hcaptcha | Bot প্রটেকশন |
| circuitbreaker | সার্কিট ব্রেকার |
| fiberi18n | ইন্টারন্যাশনালাইজেশন |

---

[← আগের: কাস্টম মিডলওয়্যার](03-custom-middleware.md) | [পরবর্তী: কনটেক্সট বেসিক →](../04-context/01-context-basics.md)
