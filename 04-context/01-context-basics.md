# কনটেক্সট বেসিক (Context Basics)

## Context কী?

`fiber.Ctx` হলো Fiber-এর সবচেয়ে গুরুত্বপূর্ণ অংশ। এটি HTTP রিকোয়েস্ট এবং রেসপন্স হ্যান্ডল করার জন্য সমস্ত মেথড প্রদান করে।

```go
app.Get("/", func(c fiber.Ctx) error {
    // c হলো Context
    return c.SendString("Hello!")
})
```

## রিকোয়েস্ট ইনফরমেশন

### পাথ এবং মেথড

```go
app.All("/info", func(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "method":       c.Method(),        // GET, POST, etc.
        "path":         c.Path(),          // /info
        "originalURL":  c.OriginalURL(),   // /info?page=1
        "protocol":     c.Protocol(),      // http বা https
        "hostname":     c.Hostname(),      // localhost
        "ip":           c.IP(),            // ক্লায়েন্ট IP
        "ips":          c.IPs(),           // প্রক্সি চেইনের সব IP
        "port":         c.Port(),          // 3000
        "secure":       c.Secure(),        // HTTPS কিনা
        "xhr":          c.XHR(),           // AJAX রিকোয়েস্ট কিনা
    })
})
```

### হেডার পড়া

```go
app.Get("/headers", func(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "contentType":   c.Get("Content-Type"),
        "authorization": c.Get("Authorization"),
        "userAgent":     c.Get("User-Agent"),
        "accept":        c.Get("Accept"),
        "customHeader":  c.Get("X-Custom-Header", "default"), // ডিফল্ট সহ
    })
})
```

### কুকি পড়া

```go
app.Get("/cookies", func(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "sessionId": c.Cookies("session_id"),
        "theme":     c.Cookies("theme", "light"), // ডিফল্ট সহ
    })
})
```

## প্যারামিটার অ্যাক্সেস

### পাথ প্যারামিটার

```go
app.Get("/user/:id", func(c fiber.Ctx) error {
    id := c.Params("id")
    return c.SendString("User ID: " + id)
})

app.Get("/user/:id/post/:postId", func(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "userId": c.Params("id"),
        "postId": c.Params("postId"),
    })
})
```

### ক্যোয়ারি প্যারামিটার

```go
app.Get("/search", func(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "q":     c.Query("q"),
        "page":  c.Query("page", "1"),      // ডিফল্ট ভ্যালু
        "limit": c.Query("limit", "10"),
    })
})

// সব ক্যোয়ারি প্যারামিটার
app.Get("/all-queries", func(c fiber.Ctx) error {
    queries := c.Queries()
    return c.JSON(queries)
})
```

## রেসপন্স পাঠানো

### স্ট্রিং রেসপন্স

```go
app.Get("/text", func(c fiber.Ctx) error {
    return c.SendString("সাধারণ টেক্সট")
})
```

### JSON রেসপন্স

```go
app.Get("/json", func(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "message": "সফল",
        "code":    200,
    })
})

// Struct থেকে JSON
type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

app.Get("/user", func(c fiber.Ctx) error {
    user := User{ID: 1, Name: "রহিম", Email: "rahim@example.com"}
    return c.JSON(user)
})
```

### স্ট্যাটাস কোড সহ

```go
app.Post("/create", func(c fiber.Ctx) error {
    return c.Status(201).JSON(fiber.Map{
        "message": "তৈরি হয়েছে",
    })
})

app.Get("/error", func(c fiber.Ctx) error {
    return c.Status(500).SendString("সার্ভার এরর")
})

// শুধু স্ট্যাটাস
app.Get("/ok", func(c fiber.Ctx) error {
    return c.SendStatus(200) // "OK" টেক্সট সহ
})
```

## হেডার সেট করা

```go
app.Get("/custom-headers", func(c fiber.Ctx) error {
    c.Set("X-Custom-Header", "MyValue")
    c.Set("Content-Type", "application/json")
    c.Set("Cache-Control", "no-cache")
    
    return c.JSON(fiber.Map{"status": "ok"})
})

// একাধিক হেডার
app.Get("/multi-headers", func(c fiber.Ctx) error {
    c.Response().Header.Set("X-Rate-Limit", "100")
    c.Response().Header.Set("X-Rate-Remaining", "99")
    
    return c.SendString("Headers set")
})
```

## কুকি সেট করা

```go
app.Get("/set-cookie", func(c fiber.Ctx) error {
    cookie := new(fiber.Cookie)
    cookie.Name = "session_id"
    cookie.Value = "abc123"
    cookie.Expires = time.Now().Add(24 * time.Hour)
    cookie.HTTPOnly = true
    cookie.Secure = true
    cookie.SameSite = "Strict"
    
    c.Cookie(cookie)
    
    return c.SendString("Cookie set!")
})

// কুকি ক্লিয়ার করা
app.Get("/clear-cookie", func(c fiber.Ctx) error {
    c.ClearCookie("session_id")
    return c.SendString("Cookie cleared!")
})
```

## রিডাইরেক্ট

```go
app.Get("/old-page", func(c fiber.Ctx) error {
    return c.Redirect("/new-page")
})

// স্ট্যাটাস কোড সহ
app.Get("/moved", func(c fiber.Ctx) error {
    return c.Redirect("/new-location", 301) // Permanent
})

// ব্যাক রিডাইরেক্ট
app.Get("/back", func(c fiber.Ctx) error {
    return c.RedirectBack("/")
})
```

## Next() - পরবর্তী হ্যান্ডলার

```go
app.Get("/", func(c fiber.Ctx) error {
    log.Println("প্রথম হ্যান্ডলার")
    return c.Next() // পরবর্তীতে যান
}, func(c fiber.Ctx) error {
    log.Println("দ্বিতীয় হ্যান্ডলার")
    return c.SendString("সম্পন্ন")
})
```

## Context Methods সারসংক্ষেপ

| মেথড | বর্ণনা |
|------|--------|
| `c.Method()` | HTTP মেথড |
| `c.Path()` | URL পাথ |
| `c.Params("key")` | পাথ প্যারামিটার |
| `c.Query("key")` | ক্যোয়ারি প্যারামিটার |
| `c.Get("header")` | হেডার পড়া |
| `c.Set("header", "value")` | হেডার সেট |
| `c.Cookies("name")` | কুকি পড়া |
| `c.Cookie(cookie)` | কুকি সেট |
| `c.IP()` | ক্লায়েন্ট IP |
| `c.SendString("text")` | টেক্সট রেসপন্স |
| `c.JSON(data)` | JSON রেসপন্স |
| `c.Status(code)` | স্ট্যাটাস কোড |
| `c.Redirect(url)` | রিডাইরেক্ট |
| `c.Next()` | পরবর্তী হ্যান্ডলার |

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Get("/api/info", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "method":    c.Method(),
            "path":      c.Path(),
            "ip":        c.IP(),
            "userAgent": c.Get("User-Agent"),
            "timestamp": time.Now(),
        })
    })

    app.Get("/user/:id", func(c fiber.Ctx) error {
        id := c.Params("id")
        fields := c.Query("fields", "all")
        
        return c.JSON(fiber.Map{
            "id":     id,
            "fields": fields,
        })
    })

    app.Post("/login", func(c fiber.Ctx) error {
        // সেশন কুকি সেট
        cookie := new(fiber.Cookie)
        cookie.Name = "session"
        cookie.Value = "user-session-token"
        cookie.Expires = time.Now().Add(24 * time.Hour)
        cookie.HTTPOnly = true
        c.Cookie(cookie)

        return c.JSON(fiber.Map{
            "message": "লগইন সফল",
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

---

[← আগের: থার্ড-পার্টি মিডলওয়্যার](../03-middleware/04-third-party.md) | [পরবর্তী: রিকোয়েস্ট হ্যান্ডলিং →](02-request-handling.md)
