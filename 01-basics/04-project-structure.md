# প্রজেক্ট স্ট্রাকচার

## ছোট প্রজেক্ট (Simple)

```
my-fiber-app/
├── go.mod
├── go.sum
└── main.go
```

## মাঝারি প্রজেক্ট (Medium)

```
my-fiber-app/
├── go.mod
├── go.sum
├── main.go
├── handlers/
│   ├── user.go
│   ├── product.go
│   └── auth.go
├── models/
│   ├── user.go
│   └── product.go
├── middleware/
│   ├── auth.go
│   └── logger.go
├── routes/
│   └── routes.go
├── config/
│   └── config.go
└── utils/
    └── helpers.go
```

## বড় প্রজেক্ট (Large - Clean Architecture)

```
my-fiber-app/
├── cmd/
│   └── api/
│       └── main.go              # Entry point
├── internal/
│   ├── handlers/                # HTTP handlers
│   │   ├── user_handler.go
│   │   └── product_handler.go
│   ├── services/                # Business logic
│   │   ├── user_service.go
│   │   └── product_service.go
│   ├── repositories/            # Database access
│   │   ├── user_repo.go
│   │   └── product_repo.go
│   ├── models/                  # Data models
│   │   ├── user.go
│   │   └── product.go
│   └── middleware/              # Custom middleware
│       ├── auth.go
│       └── cors.go
├── pkg/                         # Shared packages
│   ├── database/
│   │   └── postgres.go
│   └── validator/
│       └── validator.go
├── config/
│   └── config.go
├── routes/
│   └── routes.go
├── .env
├── go.mod
└── go.sum
```

## উদাহরণ কোড

### main.go
```go
package main

import (
    "log"
    "my-fiber-app/config"
    "my-fiber-app/routes"
    "github.com/gofiber/fiber/v3"
)

func main() {
    // কনফিগ লোড করুন
    cfg := config.Load()

    // অ্যাপ তৈরি করুন
    app := fiber.New(fiber.Config{
        AppName: cfg.AppName,
    })

    // রাউট সেটআপ করুন
    routes.Setup(app)

    // সার্ভার শুরু করুন
    log.Fatal(app.Listen(":" + cfg.Port))
}
```

### config/config.go
```go
package config

import "os"

type Config struct {
    AppName  string
    Port     string
    DBHost   string
    DBPort   string
    DBName   string
    DBUser   string
    DBPass   string
}

func Load() *Config {
    return &Config{
        AppName:  getEnv("APP_NAME", "My Fiber App"),
        Port:     getEnv("PORT", "3000"),
        DBHost:   getEnv("DB_HOST", "localhost"),
        DBPort:   getEnv("DB_PORT", "5432"),
        DBName:   getEnv("DB_NAME", "myapp"),
        DBUser:   getEnv("DB_USER", "postgres"),
        DBPass:   getEnv("DB_PASS", "password"),
    }
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}
```

### routes/routes.go
```go
package routes

import (
    "my-fiber-app/handlers"
    "my-fiber-app/middleware"
    "github.com/gofiber/fiber/v3"
)

func Setup(app *fiber.App) {
    // পাবলিক রাউট
    app.Get("/", handlers.Home)
    app.Get("/health", handlers.HealthCheck)

    // API গ্রুপ
    api := app.Group("/api")
    
    // v1 API
    v1 := api.Group("/v1")
    
    // ইউজার রাউট
    users := v1.Group("/users")
    users.Get("/", handlers.GetUsers)
    users.Get("/:id", handlers.GetUser)
    users.Post("/", handlers.CreateUser)
    users.Put("/:id", handlers.UpdateUser)
    users.Delete("/:id", handlers.DeleteUser)

    // প্রোটেক্টেড রাউট
    protected := v1.Group("/admin", middleware.Auth)
    protected.Get("/dashboard", handlers.AdminDashboard)
}
```

### handlers/user.go
```go
package handlers

import "github.com/gofiber/fiber/v3"

// GetUsers সব ইউজার রিটার্ন করে
func GetUsers(c fiber.Ctx) error {
    users := []fiber.Map{
        {"id": 1, "name": "রহিম"},
        {"id": 2, "name": "করিম"},
    }
    return c.JSON(users)
}

// GetUser একজন ইউজার রিটার্ন করে
func GetUser(c fiber.Ctx) error {
    id := c.Params("id")
    return c.JSON(fiber.Map{
        "id":   id,
        "name": "ইউজার " + id,
    })
}

// CreateUser নতুন ইউজার তৈরি করে
func CreateUser(c fiber.Ctx) error {
    type CreateUserRequest struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }

    var req CreateUserRequest
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{
            "error": "Invalid request body",
        })
    }

    return c.Status(201).JSON(fiber.Map{
        "message": "ইউজার তৈরি হয়েছে",
        "name":    req.Name,
        "email":   req.Email,
    })
}

// UpdateUser ইউজার আপডেট করে
func UpdateUser(c fiber.Ctx) error {
    id := c.Params("id")
    return c.JSON(fiber.Map{
        "message": "ইউজার " + id + " আপডেট হয়েছে",
    })
}

// DeleteUser ইউজার ডিলিট করে
func DeleteUser(c fiber.Ctx) error {
    id := c.Params("id")
    return c.JSON(fiber.Map{
        "message": "ইউজার " + id + " ডিলিট হয়েছে",
    })
}

// Home হোম পেজ
func Home(c fiber.Ctx) error {
    return c.SendString("স্বাগতম!")
}

// HealthCheck সার্ভার হেলথ চেক
func HealthCheck(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "status": "OK",
    })
}

// AdminDashboard অ্যাডমিন ড্যাশবোর্ড
func AdminDashboard(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "message": "অ্যাডমিন ড্যাশবোর্ড",
    })
}
```

### middleware/auth.go
```go
package middleware

import "github.com/gofiber/fiber/v3"

// Auth মিডলওয়্যার - অথেনটিকেশন চেক করে
func Auth(c fiber.Ctx) error {
    token := c.Get("Authorization")
    
    if token == "" {
        return c.Status(401).JSON(fiber.Map{
            "error": "অননুমোদিত অ্যাক্সেস",
        })
    }

    // টোকেন ভ্যালিডেশন লজিক এখানে
    // ...

    return c.Next()
}
```

### models/user.go
```go
package models

import "time"

type User struct {
    ID        uint      `json:"id" gorm:"primaryKey"`
    Name      string    `json:"name"`
    Email     string    `json:"email" gorm:"unique"`
    Password  string    `json:"-"` // JSON এ দেখাবে না
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

type CreateUserRequest struct {
    Name     string `json:"name" validate:"required,min=3"`
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=6"`
}

type UpdateUserRequest struct {
    Name  string `json:"name" validate:"min=3"`
    Email string `json:"email" validate:"email"`
}
```

## .env ফাইল উদাহরণ

```env
APP_NAME=My Fiber App
PORT=3000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=postgres
DB_PASS=secret123
JWT_SECRET=your-super-secret-key
```

## বেস্ট প্র্যাকটিস

1. **সেপারেশন অফ কনসার্নস** - প্রতিটি ফোল্ডারের নির্দিষ্ট দায়িত্ব থাকবে
2. **ইন্টারফেস ব্যবহার করুন** - টেস্টিং সহজ হবে
3. **এনভায়রনমেন্ট ভেরিয়েবল** - সিক্রেট কোডে রাখবেন না
4. **এরর হ্যান্ডলিং** - প্রতিটি এরর সঠিকভাবে হ্যান্ডল করুন
5. **লগিং** - সঠিক লগিং ব্যবহার করুন

---

[← আগের: Hello World](03-hello-world.md) | [পরবর্তী: বেসিক রাউটিং →](../02-routing/01-basic-routing.md)
