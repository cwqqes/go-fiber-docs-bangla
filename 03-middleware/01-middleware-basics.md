# মিডলওয়্যার বেসিক

## মিডলওয়্যার কী?

মিডলওয়্যার হলো এমন ফাংশন যা রিকোয়েস্ট এবং রেসপন্সের মাঝখানে চলে। এটি রিকোয়েস্ট প্রসেস করার আগে বা পরে কোড চালাতে পারে।

```
রিকোয়েস্ট → [মিডলওয়্যার ১] → [মিডলওয়্যার ২] → [হ্যান্ডলার] → রেসপন্স
                    ↓                ↓
               লগিং         অথেনটিকেশন
```

## বেসিক মিডলওয়্যার

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    // সিম্পল মিডলওয়্যার
    app.Use(func(c fiber.Ctx) error {
        log.Println("রিকোয়েস্ট এসেছে:", c.Path())
        return c.Next() // পরবর্তী হ্যান্ডলারে যান
    })

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("হোম পেজ")
    })

    app.Listen(":3000")
}
```

## c.Next() এর গুরুত্ব

`c.Next()` পরবর্তী মিডলওয়্যার বা হ্যান্ডলারকে কল করে। এটি না কল করলে চেইন থেমে যাবে।

```go
app.Use(func(c fiber.Ctx) error {
    log.Println("মিডলওয়্যার ১ - আগে")
    
    err := c.Next() // পরবর্তী হ্যান্ডলার চালান
    
    log.Println("মিডলওয়্যার ১ - পরে")
    return err
})

app.Use(func(c fiber.Ctx) error {
    log.Println("মিডলওয়্যার ২")
    return c.Next()
})

app.Get("/", func(c fiber.Ctx) error {
    log.Println("হ্যান্ডলার")
    return c.SendString("হ্যালো")
})

// আউটপুট:
// মিডলওয়্যার ১ - আগে
// মিডলওয়্যার ২
// হ্যান্ডলার
// মিডলওয়্যার ১ - পরে
```

## মিডলওয়্যার চেইনিং

```go
// একাধিক মিডলওয়্যার একসাথে
app.Use(
    loggerMiddleware,
    authMiddleware,
    corsMiddleware,
)

// অথবা আলাদাভাবে
app.Use(loggerMiddleware)
app.Use(authMiddleware)
app.Use(corsMiddleware)
```

## নির্দিষ্ট পাথে মিডলওয়্যার

```go
// শুধু /api পাথে
app.Use("/api", func(c fiber.Ctx) error {
    log.Println("API রিকোয়েস্ট")
    return c.Next()
})

// একাধিক পাথে
app.Use([]string{"/api", "/admin"}, func(c fiber.Ctx) error {
    log.Println("API বা Admin রিকোয়েস্ট")
    return c.Next()
})
```

## মিডলওয়্যার দিয়ে রিকোয়েস্ট মডিফাই

```go
// হেডার সেট করুন
app.Use(func(c fiber.Ctx) error {
    c.Set("X-Custom-Header", "MyValue")
    c.Set("X-Request-ID", generateID())
    return c.Next()
})

// লোকাল ভ্যালু সেট করুন
app.Use(func(c fiber.Ctx) error {
    fiber.Locals[string](c, "requestTime", time.Now().String())
    return c.Next()
})
```

## রিকোয়েস্ট ব্লক করা

```go
// অথেনটিকেশন মিডলওয়্যার
func authMiddleware(c fiber.Ctx) error {
    token := c.Get("Authorization")
    
    if token == "" {
        // c.Next() কল না করায় রিকোয়েস্ট এখানেই থামবে
        return c.Status(401).JSON(fiber.Map{
            "error": "টোকেন দরকার",
        })
    }
    
    if !isValidToken(token) {
        return c.Status(403).JSON(fiber.Map{
            "error": "অবৈধ টোকেন",
        })
    }
    
    return c.Next()
}
```

## স্কিপ মিডলওয়্যার

```go
import "github.com/gofiber/fiber/v3/middleware/skip"

// GET রিকোয়েস্টে মিডলওয়্যার স্কিপ করুন
app.Use(skip.New(authMiddleware, func(c fiber.Ctx) bool {
    return c.Method() == fiber.MethodGet
}))

// নির্দিষ্ট পাথে স্কিপ
app.Use(skip.New(authMiddleware, func(c fiber.Ctx) bool {
    return c.Path() == "/login" || c.Path() == "/register"
}))
```

## রাউট-স্পেসিফিক মিডলওয়্যার

```go
// রাউট ডিফাইন করার সময় মিডলওয়্যার যোগ করুন
app.Get("/admin", authMiddleware, adminHandler)

// একাধিক মিডলওয়্যার
app.Post("/secure-action", 
    authMiddleware, 
    rateLimitMiddleware, 
    validateMiddleware, 
    handler,
)
```

## গ্রুপে মিডলওয়্যার

```go
// API গ্রুপে মিডলওয়্যার
api := app.Group("/api", loggerMiddleware)

// নেস্টেড গ্রুপে অতিরিক্ত মিডলওয়্যার
admin := api.Group("/admin", authMiddleware, adminOnlyMiddleware)
admin.Get("/dashboard", dashboardHandler)
```

## এরর হ্যান্ডলিং মিডলওয়্যার

```go
// Recover মিডলওয়্যার - প্যানিক ধরে
import "github.com/gofiber/fiber/v3/middleware/recover"

app.Use(recoverer.New())

// কাস্টম এরর হ্যান্ডলার
app := fiber.New(fiber.Config{
    ErrorHandler: func(c fiber.Ctx, err error) error {
        code := fiber.StatusInternalServerError
        
        // Fiber এরর হলে কোড নিন
        if e, ok := err.(*fiber.Error); ok {
            code = e.Code
        }
        
        return c.Status(code).JSON(fiber.Map{
            "error": err.Error(),
        })
    },
})
```

## মিডলওয়্যার অর্ডার গুরুত্বপূর্ণ

```go
// সঠিক অর্ডার
app.Use(recoverer.New())     // ১. প্রথমে - প্যানিক ধরতে
app.Use(logger.New())        // ২. লগিং
app.Use(cors.New())          // ৩. CORS
app.Use(rateLimiter.New())   // ৪. রেট লিমিট
app.Use(authMiddleware)      // ৫. অথেনটিকেশন

// রাউট
app.Get("/", handler)
```

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
)

// লগার মিডলওয়্যার
func loggerMiddleware(c fiber.Ctx) error {
    start := time.Now()
    
    err := c.Next()
    
    log.Printf("[%s] %s %s - %dms",
        c.Method(),
        c.Path(),
        c.IP(),
        time.Since(start).Milliseconds(),
    )
    
    return err
}

// অথ মিডলওয়্যার
func authMiddleware(c fiber.Ctx) error {
    token := c.Get("Authorization")
    if token == "" {
        return c.Status(401).JSON(fiber.Map{
            "error": "অননুমোদিত",
        })
    }
    
    // ইউজার তথ্য সেট করুন
    fiber.Locals[string](c, "userId", "user123")
    return c.Next()
}

// CORS মিডলওয়্যার
func corsMiddleware(c fiber.Ctx) error {
    c.Set("Access-Control-Allow-Origin", "*")
    c.Set("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE")
    c.Set("Access-Control-Allow-Headers", "Content-Type,Authorization")
    
    if c.Method() == "OPTIONS" {
        return c.SendStatus(204)
    }
    
    return c.Next()
}

func main() {
    app := fiber.New()

    // গ্লোবাল মিডলওয়্যার
    app.Use(loggerMiddleware)
    app.Use(corsMiddleware)

    // পাবলিক রাউট
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("হোম পেজ")
    })

    app.Get("/health", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{"status": "OK"})
    })

    // প্রোটেক্টেড রাউট
    protected := app.Group("/api", authMiddleware)
    protected.Get("/profile", func(c fiber.Ctx) error {
        userId := fiber.Locals[string](c, "userId")
        return c.JSON(fiber.Map{
            "userId": userId,
            "name":   "ইউজার",
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

## টেস্ট করুন

```bash
# পাবলিক রাউট
curl http://localhost:3000/
curl http://localhost:3000/health

# প্রোটেক্টেড রাউট (401)
curl http://localhost:3000/api/profile

# প্রোটেক্টেড রাউট (সফল)
curl -H "Authorization: Bearer token" http://localhost:3000/api/profile

# OPTIONS রিকোয়েস্ট
curl -X OPTIONS http://localhost:3000/api/profile
```

---

[← আগের: স্ট্যাটিক ফাইল](../02-routing/04-static-files.md) | [পরবর্তী: বিল্ট-ইন মিডলওয়্যার →](02-builtin-middleware.md)
