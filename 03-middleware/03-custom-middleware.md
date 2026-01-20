# কাস্টম মিডলওয়্যার

## কাস্টম মিডলওয়্যার তৈরি

নিজের প্রয়োজন অনুযায়ী মিডলওয়্যার তৈরি করা যায়।

## সিম্পল মিডলওয়্যার

```go
// ফাংশন হিসেবে
func myMiddleware(c fiber.Ctx) error {
    // রিকোয়েস্ট প্রসেসিংয়ের আগে
    log.Println("Before handler")
    
    // পরবর্তী হ্যান্ডলার কল করুন
    err := c.Next()
    
    // রিকোয়েস্ট প্রসেসিংয়ের পরে
    log.Println("After handler")
    
    return err
}

// ব্যবহার
app.Use(myMiddleware)
```

## কনফিগারেশন সহ মিডলওয়্যার

```go
package middleware

import "github.com/gofiber/fiber/v3"

// কনফিগ স্ট্রাক্ট
type LoggerConfig struct {
    Format     string
    TimeFormat string
    Output     io.Writer
    Next       func(c fiber.Ctx) bool
}

// ডিফল্ট কনফিগ
var LoggerConfigDefault = LoggerConfig{
    Format:     "[${time}] ${method} ${path}",
    TimeFormat: "2006-01-02 15:04:05",
    Output:     os.Stdout,
    Next:       nil,
}

// মিডলওয়্যার ফ্যাক্টরি
func NewLogger(config ...LoggerConfig) fiber.Handler {
    cfg := LoggerConfigDefault
    
    if len(config) > 0 {
        cfg = config[0]
        
        // ডিফল্ট ভ্যালু সেট করুন
        if cfg.Format == "" {
            cfg.Format = LoggerConfigDefault.Format
        }
        if cfg.TimeFormat == "" {
            cfg.TimeFormat = LoggerConfigDefault.TimeFormat
        }
        if cfg.Output == nil {
            cfg.Output = LoggerConfigDefault.Output
        }
    }
    
    return func(c fiber.Ctx) error {
        // স্কিপ চেক
        if cfg.Next != nil && cfg.Next(c) {
            return c.Next()
        }
        
        // লগিং লজিক
        start := time.Now()
        err := c.Next()
        duration := time.Since(start)
        
        log := strings.ReplaceAll(cfg.Format, "${time}", time.Now().Format(cfg.TimeFormat))
        log = strings.ReplaceAll(log, "${method}", c.Method())
        log = strings.ReplaceAll(log, "${path}", c.Path())
        log = strings.ReplaceAll(log, "${duration}", duration.String())
        
        fmt.Fprintln(cfg.Output, log)
        
        return err
    }
}

// ব্যবহার
app.Use(NewLogger())
app.Use(NewLogger(LoggerConfig{
    Format: "[${method}] ${path} - ${duration}",
}))
```

## অথেনটিকেশন মিডলওয়্যার

```go
package middleware

import (
    "strings"
    "github.com/gofiber/fiber/v3"
    "github.com/golang-jwt/jwt/v5"
)

type AuthConfig struct {
    Secret        string
    ContextKey    string
    TokenLookup   string // "header:Authorization" or "cookie:token"
    AuthScheme    string // "Bearer"
    SuccessHandler fiber.Handler
    ErrorHandler  func(c fiber.Ctx, err error) error
}

var AuthConfigDefault = AuthConfig{
    ContextKey:  "user",
    TokenLookup: "header:Authorization",
    AuthScheme:  "Bearer",
    ErrorHandler: func(c fiber.Ctx, err error) error {
        return c.Status(401).JSON(fiber.Map{
            "error": "অননুমোদিত",
        })
    },
}

func NewAuth(config AuthConfig) fiber.Handler {
    cfg := config
    
    if cfg.ContextKey == "" {
        cfg.ContextKey = AuthConfigDefault.ContextKey
    }
    if cfg.TokenLookup == "" {
        cfg.TokenLookup = AuthConfigDefault.TokenLookup
    }
    if cfg.AuthScheme == "" {
        cfg.AuthScheme = AuthConfigDefault.AuthScheme
    }
    if cfg.ErrorHandler == nil {
        cfg.ErrorHandler = AuthConfigDefault.ErrorHandler
    }
    
    return func(c fiber.Ctx) error {
        var tokenStr string
        
        // টোকেন এক্সট্র্যাক্ট করুন
        parts := strings.Split(cfg.TokenLookup, ":")
        switch parts[0] {
        case "header":
            auth := c.Get(parts[1])
            if strings.HasPrefix(auth, cfg.AuthScheme+" ") {
                tokenStr = auth[len(cfg.AuthScheme)+1:]
            }
        case "cookie":
            tokenStr = c.Cookies(parts[1])
        }
        
        if tokenStr == "" {
            return cfg.ErrorHandler(c, fiber.ErrUnauthorized)
        }
        
        // টোকেন ভ্যালিডেট করুন
        token, err := jwt.Parse(tokenStr, func(t *jwt.Token) (interface{}, error) {
            return []byte(cfg.Secret), nil
        })
        
        if err != nil || !token.Valid {
            return cfg.ErrorHandler(c, fiber.ErrUnauthorized)
        }
        
        // ইউজার ইনফো কনটেক্সটে সেট করুন
        claims := token.Claims.(jwt.MapClaims)
        fiber.Locals[jwt.MapClaims](c, cfg.ContextKey, claims)
        
        return c.Next()
    }
}

// ব্যবহার
app.Use("/api", NewAuth(AuthConfig{
    Secret: "my-secret-key",
}))
```

## রোল-বেসড অ্যাক্সেস কন্ট্রোল

```go
package middleware

import "github.com/gofiber/fiber/v3"

// রোল চেক মিডলওয়্যার
func RequireRole(roles ...string) fiber.Handler {
    return func(c fiber.Ctx) error {
        // কনটেক্সট থেকে ইউজার রোল নিন
        userRole := fiber.Locals[string](c, "userRole")
        
        // রোল চেক করুন
        for _, role := range roles {
            if userRole == role {
                return c.Next()
            }
        }
        
        return c.Status(403).JSON(fiber.Map{
            "error": "অ্যাক্সেস নিষিদ্ধ",
        })
    }
}

// ব্যবহার
admin := app.Group("/admin")
admin.Use(authMiddleware)
admin.Use(RequireRole("admin", "superadmin"))
admin.Get("/dashboard", dashboardHandler)
```

## রেট লিমিটার (কাস্টম)

```go
package middleware

import (
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
)

type RateLimiterConfig struct {
    Max        int
    Duration   time.Duration
    KeyFunc    func(c fiber.Ctx) string
    LimitReached func(c fiber.Ctx) error
}

type rateLimiter struct {
    requests map[string][]time.Time
    mu       sync.Mutex
}

func NewRateLimiter(config RateLimiterConfig) fiber.Handler {
    limiter := &rateLimiter{
        requests: make(map[string][]time.Time),
    }
    
    // ক্লিনআপ গোরুটিন
    go func() {
        for {
            time.Sleep(config.Duration)
            limiter.cleanup(config.Duration)
        }
    }()
    
    return func(c fiber.Ctx) error {
        key := config.KeyFunc(c)
        
        limiter.mu.Lock()
        now := time.Now()
        
        // পুরানো রিকোয়েস্ট ফিল্টার করুন
        var valid []time.Time
        for _, t := range limiter.requests[key] {
            if now.Sub(t) < config.Duration {
                valid = append(valid, t)
            }
        }
        
        // লিমিট চেক করুন
        if len(valid) >= config.Max {
            limiter.mu.Unlock()
            return config.LimitReached(c)
        }
        
        // নতুন রিকোয়েস্ট যোগ করুন
        limiter.requests[key] = append(valid, now)
        limiter.mu.Unlock()
        
        return c.Next()
    }
}

func (r *rateLimiter) cleanup(duration time.Duration) {
    r.mu.Lock()
    defer r.mu.Unlock()
    
    now := time.Now()
    for key, times := range r.requests {
        var valid []time.Time
        for _, t := range times {
            if now.Sub(t) < duration {
                valid = append(valid, t)
            }
        }
        if len(valid) == 0 {
            delete(r.requests, key)
        } else {
            r.requests[key] = valid
        }
    }
}

// ব্যবহার
app.Use(NewRateLimiter(RateLimiterConfig{
    Max:      10,
    Duration: time.Minute,
    KeyFunc: func(c fiber.Ctx) string {
        return c.IP()
    },
    LimitReached: func(c fiber.Ctx) error {
        return c.Status(429).JSON(fiber.Map{
            "error": "অনেক বেশি রিকোয়েস্ট",
        })
    },
}))
```

## রিকোয়েস্ট ভ্যালিডেশন মিডলওয়্যার

```go
package middleware

import (
    "github.com/go-playground/validator/v10"
    "github.com/gofiber/fiber/v3"
)

var validate = validator.New()

func ValidateBody[T any]() fiber.Handler {
    return func(c fiber.Ctx) error {
        var body T
        
        if err := c.Bind().JSON(&body); err != nil {
            return c.Status(400).JSON(fiber.Map{
                "error": "অবৈধ JSON",
            })
        }
        
        if err := validate.Struct(body); err != nil {
            var errors []string
            for _, err := range err.(validator.ValidationErrors) {
                errors = append(errors, err.Field()+" - "+err.Tag())
            }
            return c.Status(400).JSON(fiber.Map{
                "error":  "ভ্যালিডেশন ব্যর্থ",
                "fields": errors,
            })
        }
        
        // ভ্যালিডেটেড বডি সেভ করুন
        fiber.Locals[T](c, "body", body)
        
        return c.Next()
    }
}

// ব্যবহার
type CreateUserRequest struct {
    Name  string `json:"name" validate:"required,min=3"`
    Email string `json:"email" validate:"required,email"`
    Age   int    `json:"age" validate:"gte=18"`
}

app.Post("/users", ValidateBody[CreateUserRequest](), func(c fiber.Ctx) error {
    body := fiber.Locals[CreateUserRequest](c, "body")
    return c.JSON(fiber.Map{
        "message": "ইউজার তৈরি হয়েছে",
        "user":    body,
    })
})
```

## টাইমিং মিডলওয়্যার

```go
func TimingMiddleware(c fiber.Ctx) error {
    start := time.Now()
    
    err := c.Next()
    
    duration := time.Since(start)
    c.Set("X-Response-Time", duration.String())
    
    // স্লো রিকোয়েস্ট লগ করুন
    if duration > 500*time.Millisecond {
        log.Printf("SLOW REQUEST: %s %s - %v", c.Method(), c.Path(), duration)
    }
    
    return err
}
```

## মিডলওয়্যার টেস্টিং

```go
package middleware

import (
    "net/http/httptest"
    "testing"
    "github.com/gofiber/fiber/v3"
    "github.com/stretchr/testify/assert"
)

func TestAuthMiddleware(t *testing.T) {
    app := fiber.New()
    
    app.Use(NewAuth(AuthConfig{
        Secret: "test-secret",
    }))
    
    app.Get("/protected", func(c fiber.Ctx) error {
        return c.SendString("success")
    })
    
    // টোকেন ছাড়া - 401 হওয়া উচিত
    req := httptest.NewRequest("GET", "/protected", nil)
    resp, _ := app.Test(req)
    assert.Equal(t, 401, resp.StatusCode)
    
    // ভ্যালিড টোকেন সহ - 200 হওয়া উচিত
    req = httptest.NewRequest("GET", "/protected", nil)
    req.Header.Set("Authorization", "Bearer valid-jwt-token")
    resp, _ = app.Test(req)
    assert.Equal(t, 200, resp.StatusCode)
}
```

---

[← আগের: বিল্ট-ইন মিডলওয়্যার](02-builtin-middleware.md) | [পরবর্তী: থার্ড-পার্টি মিডলওয়্যার →](04-third-party.md)
