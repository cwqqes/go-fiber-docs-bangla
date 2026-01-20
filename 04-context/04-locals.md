# Locals - রিকোয়েস্ট-স্কোপড ডেটা

## Locals কী?

`Locals` হলো রিকোয়েস্ট-স্কোপড ডেটা স্টোরেজ। এটি একটি রিকোয়েস্টের মধ্যে বিভিন্ন মিডলওয়্যার এবং হ্যান্ডলারের মধ্যে ডেটা শেয়ার করতে ব্যবহৃত হয়।

```
রিকোয়েস্ট → [মিডলওয়্যার: Locals সেট] → [হ্যান্ডলার: Locals পড়া] → রেসপন্স
```

## বেসিক ব্যবহার (Fiber v3)

Fiber v3 তে জেনেরিক ফাংশন ব্যবহার করা হয়:

```go
// সেট করা
fiber.Locals[string](c, "key", "value")

// পড়া
value := fiber.Locals[string](c, "key")
```

## উদাহরণ

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    // মিডলওয়্যার - ইউজার সেট করে
    app.Use(func(c fiber.Ctx) error {
        // string টাইপ
        fiber.Locals[string](c, "username", "rahim")
        
        // int টাইপ
        fiber.Locals[int](c, "userId", 123)
        
        // struct টাইপ
        type User struct {
            ID   int
            Name string
        }
        fiber.Locals[User](c, "user", User{ID: 1, Name: "রহিম"})
        
        return c.Next()
    })

    // হ্যান্ডলার - Locals পড়ে
    app.Get("/profile", func(c fiber.Ctx) error {
        username := fiber.Locals[string](c, "username")
        userId := fiber.Locals[int](c, "userId")
        
        return c.JSON(fiber.Map{
            "username": username,
            "userId":   userId,
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

## অথেনটিকেশনে ব্যবহার

```go
// অথ মিডলওয়্যার
func authMiddleware(c fiber.Ctx) error {
    token := c.Get("Authorization")
    
    if token == "" {
        return c.Status(401).JSON(fiber.Map{
            "error": "টোকেন প্রয়োজন",
        })
    }
    
    // টোকেন ভেরিফাই করুন এবং ইউজার তথ্য পান
    user, err := verifyToken(token)
    if err != nil {
        return c.Status(401).JSON(fiber.Map{
            "error": "অবৈধ টোকেন",
        })
    }
    
    // ইউজার তথ্য Locals-এ সেভ করুন
    fiber.Locals[User](c, "user", user)
    fiber.Locals[string](c, "userId", user.ID)
    fiber.Locals[string](c, "role", user.Role)
    
    return c.Next()
}

// প্রোফাইল হ্যান্ডলার
app.Get("/api/profile", authMiddleware, func(c fiber.Ctx) error {
    user := fiber.Locals[User](c, "user")
    
    return c.JSON(fiber.Map{
        "user": user,
    })
})

// অ্যাডমিন হ্যান্ডলার
app.Get("/api/admin", authMiddleware, func(c fiber.Ctx) error {
    role := fiber.Locals[string](c, "role")
    
    if role != "admin" {
        return c.Status(403).JSON(fiber.Map{
            "error": "অ্যাক্সেস নিষিদ্ধ",
        })
    }
    
    return c.JSON(fiber.Map{
        "message": "অ্যাডমিন ড্যাশবোর্ড",
    })
})
```

## রিকোয়েস্ট মেটাডেটা

```go
// রিকোয়েস্ট শুরুর সময় সেভ
app.Use(func(c fiber.Ctx) error {
    fiber.Locals[time.Time](c, "startTime", time.Now())
    fiber.Locals[string](c, "requestId", uuid.New().String())
    
    return c.Next()
})

// রেসপন্সের আগে সময় ক্যালকুলেট
app.Use(func(c fiber.Ctx) error {
    err := c.Next()
    
    startTime := fiber.Locals[time.Time](c, "startTime")
    duration := time.Since(startTime)
    
    requestId := fiber.Locals[string](c, "requestId")
    
    // হেডারে যোগ করুন
    c.Set("X-Request-ID", requestId)
    c.Set("X-Response-Time", duration.String())
    
    return err
})
```

## ডাটাবেস কানেকশন

```go
// ডাটাবেস কানেকশন সেট
app.Use(func(c fiber.Ctx) error {
    db := getDBConnection()
    fiber.Locals[*sql.DB](c, "db", db)
    
    return c.Next()
})

// হ্যান্ডলারে ব্যবহার
app.Get("/users", func(c fiber.Ctx) error {
    db := fiber.Locals[*sql.DB](c, "db")
    
    rows, _ := db.Query("SELECT * FROM users")
    // ...
    
    return c.JSON(users)
})
```

## ট্রান্সঅ্যাকশন

```go
// ট্রান্সঅ্যাকশন মিডলওয়্যার
func transactionMiddleware(c fiber.Ctx) error {
    db := fiber.Locals[*sql.DB](c, "db")
    
    tx, err := db.Begin()
    if err != nil {
        return c.Status(500).SendString("ট্রান্সঅ্যাকশন শুরু ব্যর্থ")
    }
    
    fiber.Locals[*sql.Tx](c, "tx", tx)
    
    err = c.Next()
    
    if err != nil {
        tx.Rollback()
        return err
    }
    
    return tx.Commit()
}

app.Post("/transfer", transactionMiddleware, func(c fiber.Ctx) error {
    tx := fiber.Locals[*sql.Tx](c, "tx")
    
    // ট্রান্সঅ্যাকশন অপারেশন
    tx.Exec("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
    tx.Exec("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
    
    return c.JSON(fiber.Map{"message": "ট্রান্সফার সফল"})
})
```

## মিডলওয়্যার থেকে ডেটা অ্যাক্সেস ফাংশন

```go
package middleware

import "github.com/gofiber/fiber/v3"

const (
    UserKey      = "user"
    UserIDKey    = "userId"
    RequestIDKey = "requestId"
)

type User struct {
    ID    string
    Name  string
    Email string
    Role  string
}

// হেল্পার ফাংশন
func SetUser(c fiber.Ctx, user User) {
    fiber.Locals[User](c, UserKey, user)
}

func GetUser(c fiber.Ctx) User {
    return fiber.Locals[User](c, UserKey)
}

func GetUserID(c fiber.Ctx) string {
    user := GetUser(c)
    return user.ID
}

func GetRequestID(c fiber.Ctx) string {
    return fiber.Locals[string](c, RequestIDKey)
}

// ব্যবহার
app.Get("/profile", func(c fiber.Ctx) error {
    user := middleware.GetUser(c)
    return c.JSON(user)
})
```

## Fiber v2 vs v3

### Fiber v2 (পুরাতন)
```go
// সেট
c.Locals("key", "value")

// পড়া (টাইপ অ্যাসার্শন দরকার)
value := c.Locals("key").(string)
```

### Fiber v3 (নতুন - জেনেরিক)
```go
// সেট
fiber.Locals[string](c, "key", "value")

// পড়া (টাইপ-সেফ)
value := fiber.Locals[string](c, "key")
```

## সতর্কতা

1. **শুধু রিকোয়েস্ট-স্কোপ**: Locals শুধু বর্তমান রিকোয়েস্টে কাজ করে
2. **গোরুটিন সতর্কতা**: গোরুটিনে Context পাস করবেন না
3. **টাইপ মিসম্যাচ**: সঠিক টাইপ ব্যবহার করুন

```go
// ভুল - বিপজ্জনক
go func() {
    user := fiber.Locals[User](c, "user") // সমস্যা হতে পারে!
}()

// সঠিক
user := fiber.Locals[User](c, "user")
go func(u User) {
    // u ব্যবহার করুন
}(user)
```

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/google/uuid"
)

type User struct {
    ID    string
    Name  string
    Role  string
}

func main() {
    app := fiber.New()

    // রিকোয়েস্ট মেটাডেটা
    app.Use(func(c fiber.Ctx) error {
        fiber.Locals[string](c, "requestId", uuid.New().String())
        fiber.Locals[time.Time](c, "startTime", time.Now())
        return c.Next()
    })

    // অথ মিডলওয়্যার
    authMiddleware := func(c fiber.Ctx) error {
        token := c.Get("Authorization")
        if token == "" {
            return c.Status(401).JSON(fiber.Map{"error": "অননুমোদিত"})
        }
        
        // ডামি ইউজার
        user := User{ID: "123", Name: "রহিম", Role: "admin"}
        fiber.Locals[User](c, "user", user)
        
        return c.Next()
    }

    // পাবলিক রাউট
    app.Get("/", func(c fiber.Ctx) error {
        requestId := fiber.Locals[string](c, "requestId")
        return c.JSON(fiber.Map{
            "message":   "হোম",
            "requestId": requestId,
        })
    })

    // প্রোটেক্টেড রাউট
    api := app.Group("/api", authMiddleware)
    
    api.Get("/me", func(c fiber.Ctx) error {
        user := fiber.Locals[User](c, "user")
        requestId := fiber.Locals[string](c, "requestId")
        
        return c.JSON(fiber.Map{
            "user":      user,
            "requestId": requestId,
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

---

[← আগের: রেসপন্স মেথড](03-response-methods.md) | [পরবর্তী: গোরুটিন বেসিক →](../05-goroutines/01-goroutine-basics.md)
