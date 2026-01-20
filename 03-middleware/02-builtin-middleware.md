# বিল্ট-ইন মিডলওয়্যার

Fiber অনেক বিল্ট-ইন মিডলওয়্যার প্রদান করে। এগুলো সাধারণ কাজ সহজ করে দেয়।

## Logger - লগিং মিডলওয়্যার

```go
import "github.com/gofiber/fiber/v3/middleware/logger"

// ডিফল্ট কনফিগ
app.Use(logger.New())

// কাস্টম কনফিগ
app.Use(logger.New(logger.Config{
    Format:     "[${time}] ${status} - ${method} ${path} ${latency}\n",
    TimeFormat: "02-Jan-2006 15:04:05",
    TimeZone:   "Asia/Dhaka",
}))

// আউটপুট: [20-Jan-2024 14:30:00] 200 - GET /users 1.234ms
```

### কাস্টম আউটপুট
```go
import "os"

file, _ := os.OpenFile("./logs/app.log", os.O_RDWR|os.O_CREATE|os.O_APPEND, 0666)

app.Use(logger.New(logger.Config{
    Output: file,
}))
```

## Recover - প্যানিক রিকভারি

```go
import recoverer "github.com/gofiber/fiber/v3/middleware/recover"

// ডিফল্ট
app.Use(recoverer.New())

// কাস্টম কনফিগ
app.Use(recoverer.New(recoverer.Config{
    EnableStackTrace: true,
}))

// এখন প্যানিক হলেও সার্ভার ক্র্যাশ হবে না
app.Get("/panic", func(c fiber.Ctx) error {
    panic("কিছু একটা ভুল হয়েছে!")
})
```

## CORS - Cross-Origin Resource Sharing

```go
import "github.com/gofiber/fiber/v3/middleware/cors"

// ডিফল্ট (সব অরিজিন অনুমোদিত)
app.Use(cors.New())

// কাস্টম কনফিগ
app.Use(cors.New(cors.Config{
    AllowOrigins: "https://example.com, https://app.example.com",
    AllowMethods: "GET,POST,PUT,DELETE",
    AllowHeaders: "Origin, Content-Type, Accept, Authorization",
    AllowCredentials: true,
    MaxAge: 3600,
}))
```

## Compress - রেসপন্স কম্প্রেশন

```go
import "github.com/gofiber/fiber/v3/middleware/compress"

// ডিফল্ট
app.Use(compress.New())

// কাস্টম লেভেল
app.Use(compress.New(compress.Config{
    Level: compress.LevelBestSpeed, // দ্রুত কম্প্রেশন
    // অথবা
    // Level: compress.LevelBestCompression, // সেরা কম্প্রেশন
}))
```

## BasicAuth - বেসিক অথেনটিকেশন

```go
import "github.com/gofiber/fiber/v3/middleware/basicauth"

app.Use(basicauth.New(basicauth.Config{
    Users: map[string]string{
        "admin": "password123",
        "user":  "user123",
    },
    Realm: "Restricted",
    Unauthorized: func(c fiber.Ctx) error {
        return c.Status(401).SendString("অননুমোদিত")
    },
}))
```

## KeyAuth - API Key অথেনটিকেশন

```go
import "github.com/gofiber/fiber/v3/middleware/keyauth"

app.Use(keyauth.New(keyauth.Config{
    KeyLookup: "header:X-API-Key",
    Validator: func(c fiber.Ctx, key string) (bool, error) {
        // ডাটাবেস থেকে কী চেক করুন
        return key == "valid-api-key", nil
    },
}))

// অথবা Query প্যারামিটার থেকে
app.Use(keyauth.New(keyauth.Config{
    KeyLookup: "query:api_key",
    Validator: validateAPIKey,
}))
```

## RequestID - রিকোয়েস্ট আইডেন্টিফায়ার

```go
import "github.com/gofiber/fiber/v3/middleware/requestid"

app.Use(requestid.New())

// রিকোয়েস্ট আইডি পান
app.Get("/", func(c fiber.Ctx) error {
    id := requestid.FromContext(c)
    return c.SendString("Request ID: " + id)
})
```

## RateLimiter - রেট লিমিটিং

```go
import "github.com/gofiber/fiber/v3/middleware/limiter"

// প্রতি মিনিটে ২০টি রিকোয়েস্ট
app.Use(limiter.New(limiter.Config{
    Max:        20,
    Expiration: 1 * time.Minute,
    KeyGenerator: func(c fiber.Ctx) string {
        return c.IP()
    },
    LimitReached: func(c fiber.Ctx) error {
        return c.Status(429).JSON(fiber.Map{
            "error": "অনেক বেশি রিকোয়েস্ট, পরে চেষ্টা করুন",
        })
    },
}))
```

## Timeout - টাইমআউট

```go
import "github.com/gofiber/fiber/v3/middleware/timeout"

app.Get("/slow", timeout.New(func(c fiber.Ctx) error {
    // দীর্ঘ কাজ
    time.Sleep(5 * time.Second)
    return c.SendString("সম্পন্ন")
}, timeout.Config{
    Timeout: 2 * time.Second,
}))

// ২ সেকেন্ডের বেশি সময় লাগলে টাইমআউট
```

## Cache - রেসপন্স ক্যাশিং

```go
import "github.com/gofiber/fiber/v3/middleware/cache"

// ৫ মিনিট ক্যাশ
app.Use(cache.New(cache.Config{
    Expiration: 5 * time.Minute,
    CacheControl: true,
}))

// নির্দিষ্ট রাউটে ক্যাশ
app.Get("/api/products", cache.New(cache.Config{
    Expiration: 10 * time.Minute,
}), getProducts)
```

## Helmet - সিকিউরিটি হেডার

```go
import "github.com/gofiber/fiber/v3/middleware/helmet"

// ডিফল্ট সিকিউরিটি হেডার
app.Use(helmet.New())

// সেট করা হেডার:
// X-XSS-Protection: 0
// X-Content-Type-Options: nosniff
// X-Download-Options: noopen
// X-Frame-Options: SAMEORIGIN
// X-DNS-Prefetch-Control: off
// X-Permitted-Cross-Domain-Policies: none
// Referrer-Policy: no-referrer
```

## CSRF - Cross-Site Request Forgery সুরক্ষা

```go
import "github.com/gofiber/fiber/v3/middleware/csrf"

app.Use(csrf.New(csrf.Config{
    KeyLookup:      "header:X-CSRF-Token",
    CookieName:     "csrf_",
    CookieSameSite: "Strict",
    Expiration:     1 * time.Hour,
}))

// টোকেন পান
app.Get("/csrf-token", func(c fiber.Ctx) error {
    token := csrf.TokenFromContext(c)
    return c.JSON(fiber.Map{"token": token})
})
```

## Session - সেশন ম্যানেজমেন্ট

```go
import "github.com/gofiber/fiber/v3/middleware/session"

store := session.New()

app.Get("/login", func(c fiber.Ctx) error {
    sess, _ := store.Get(c)
    
    sess.Set("userId", "123")
    sess.Set("username", "rahim")
    sess.Save()
    
    return c.SendString("লগইন সফল")
})

app.Get("/profile", func(c fiber.Ctx) error {
    sess, _ := store.Get(c)
    
    userId := sess.Get("userId")
    if userId == nil {
        return c.Status(401).SendString("লগইন করুন")
    }
    
    return c.SendString("ইউজার: " + userId.(string))
})

app.Get("/logout", func(c fiber.Ctx) error {
    sess, _ := store.Get(c)
    sess.Destroy()
    return c.SendString("লগআউট সফল")
})
```

## Redirect - রিডাইরেক্ট

```go
import "github.com/gofiber/fiber/v3/middleware/redirect"

app.Use(redirect.New(redirect.Config{
    Rules: map[string]string{
        "/old-page":  "/new-page",
        "/old-api/*": "/api/v2/$1",
    },
    StatusCode: 301, // Permanent Redirect
}))
```

## Rewrite - URL রিরাইট

```go
import "github.com/gofiber/fiber/v3/middleware/rewrite"

app.Use(rewrite.New(rewrite.Config{
    Rules: map[string]string{
        "/api/v1/*": "/api/v2/$1",
        "/old":      "/new",
    },
}))
```

## ETag - কন্টেন্ট ক্যাশিং

```go
import "github.com/gofiber/fiber/v3/middleware/etag"

app.Use(etag.New())

// ব্রাউজার ETag মিলে গেলে 304 Not Modified পাঠাবে
```

## সম্পূর্ণ সিকিউর সেটআপ

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/cors"
    "github.com/gofiber/fiber/v3/middleware/helmet"
    "github.com/gofiber/fiber/v3/middleware/limiter"
    "github.com/gofiber/fiber/v3/middleware/logger"
    recoverer "github.com/gofiber/fiber/v3/middleware/recover"
    "github.com/gofiber/fiber/v3/middleware/requestid"
)

func main() {
    app := fiber.New()

    // সিকিউরিটি মিডলওয়্যার (অর্ডার গুরুত্বপূর্ণ)
    app.Use(recoverer.New())
    app.Use(requestid.New())
    app.Use(logger.New())
    app.Use(helmet.New())
    app.Use(cors.New(cors.Config{
        AllowOrigins: "https://myapp.com",
        AllowMethods: "GET,POST,PUT,DELETE",
    }))
    app.Use(limiter.New(limiter.Config{
        Max:        100,
        Expiration: 1 * time.Minute,
    }))

    // রাউট
    app.Get("/", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "message": "সিকিউর API",
            "requestId": requestid.FromContext(c),
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

---

[← আগের: মিডলওয়্যার বেসিক](01-middleware-basics.md) | [পরবর্তী: কাস্টম মিডলওয়্যার →](03-custom-middleware.md)
