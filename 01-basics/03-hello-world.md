# Hello World - প্রথম Fiber অ্যাপ্লিকেশন

## সবচেয়ে সহজ উদাহরণ

```go
package main

import "github.com/gofiber/fiber/v3"

func main() {
    // নতুন Fiber অ্যাপ তৈরি
    app := fiber.New()

    // রুট পাথে GET রিকোয়েস্ট হ্যান্ডল করুন
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })

    // পোর্ট 3000 এ সার্ভার শুরু করুন
    app.Listen(":3000")
}
```

## কোড বিশ্লেষণ

### ১. প্যাকেজ ইম্পোর্ট
```go
import "github.com/gofiber/fiber/v3"
```
- Fiber প্যাকেজ ইম্পোর্ট করা হয়েছে

### ২. অ্যাপ তৈরি
```go
app := fiber.New()
```
- `fiber.New()` একটি নতুন Fiber অ্যাপ্লিকেশন ইনস্ট্যান্স তৈরি করে
- এটি সমস্ত রাউট এবং মিডলওয়্যার ধারণ করে

### ৩. রাউট ডিফাইন করা
```go
app.Get("/", func(c fiber.Ctx) error {
    return c.SendString("Hello, World!")
})
```
- `app.Get()` - HTTP GET মেথডের জন্য রাউট
- `"/"` - রুট পাথ (http://localhost:3000/)
- `func(c fiber.Ctx)` - হ্যান্ডলার ফাংশন
- `c.SendString()` - ক্লায়েন্টকে স্ট্রিং পাঠায়

### ৪. সার্ভার শুরু
```go
app.Listen(":3000")
```
- পোর্ট 3000 এ HTTP সার্ভার শুরু করে

## বিভিন্ন HTTP মেথড

```go
package main

import "github.com/gofiber/fiber/v3"

func main() {
    app := fiber.New()

    // GET রিকোয়েস্ট
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("GET রিকোয়েস্ট")
    })

    // POST রিকোয়েস্ট
    app.Post("/", func(c fiber.Ctx) error {
        return c.SendString("POST রিকোয়েস্ট")
    })

    // PUT রিকোয়েস্ট
    app.Put("/user", func(c fiber.Ctx) error {
        return c.SendString("PUT রিকোয়েস্ট")
    })

    // DELETE রিকোয়েস্ট
    app.Delete("/user", func(c fiber.Ctx) error {
        return c.SendString("DELETE রিকোয়েস্ট")
    })

    // PATCH রিকোয়েস্ট
    app.Patch("/user", func(c fiber.Ctx) error {
        return c.SendString("PATCH রিকোয়েস্ট")
    })

    app.Listen(":3000")
}
```

## JSON রেসপন্স

```go
package main

import "github.com/gofiber/fiber/v3"

func main() {
    app := fiber.New()

    app.Get("/json", func(c fiber.Ctx) error {
        // Map থেকে JSON
        return c.JSON(fiber.Map{
            "message": "স্বাগতম!",
            "status":  "success",
            "code":    200,
        })
    })

    app.Get("/user", func(c fiber.Ctx) error {
        // Struct থেকে JSON
        type User struct {
            Name  string `json:"name"`
            Email string `json:"email"`
            Age   int    `json:"age"`
        }

        user := User{
            Name:  "রহিম",
            Email: "rahim@example.com",
            Age:   25,
        }

        return c.JSON(user)
    })

    app.Listen(":3000")
}
```

## Status Code সহ রেসপন্স

```go
app.Get("/created", func(c fiber.Ctx) error {
    return c.Status(201).SendString("রিসোর্স তৈরি হয়েছে")
})

app.Get("/error", func(c fiber.Ctx) error {
    return c.Status(500).JSON(fiber.Map{
        "error": "সার্ভার এরর",
    })
})

app.Get("/notfound", func(c fiber.Ctx) error {
    return c.SendStatus(404) // শুধু স্ট্যাটাস কোড
})
```

## কনফিগারেশন সহ অ্যাপ

```go
package main

import (
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    // কাস্টম কনফিগারেশন
    app := fiber.New(fiber.Config{
        AppName:      "My Fiber App v1.0",
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
    })

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("কনফিগার করা অ্যাপ!")
    })

    app.Listen(":3000")
}
```

## চালানোর নির্দেশনা

### ১. ফাইল সেভ করুন
`main.go` নামে সেভ করুন

### ২. টার্মিনালে চালান
```bash
go run main.go
```

### ৩. ব্রাউজারে যান
```
http://localhost:3000
```

### ৪. cURL দিয়ে টেস্ট করুন
```bash
# GET রিকোয়েস্ট
curl http://localhost:3000

# POST রিকোয়েস্ট
curl -X POST http://localhost:3000

# JSON রেসপন্স
curl http://localhost:3000/json
```

## অনুশীলনী

1. নতুন একটি রাউট `/about` তৈরি করুন যা আপনার নাম রিটার্ন করবে
2. `/api/status` রাউটে JSON রিটার্ন করুন যাতে বর্তমান সময় থাকবে
3. বিভিন্ন HTTP মেথড টেস্ট করুন

---

[← আগের: ইনস্টলেশন](02-installation.md) | [পরবর্তী: প্রজেক্ট স্ট্রাকচার →](04-project-structure.md)
