# রাউট গ্রুপিং

## গ্রুপিং কী?

রাউট গ্রুপিং হলো সম্পর্কিত রাউটগুলোকে একত্রে সংগঠিত করা। এটি কোড পরিষ্কার রাখে এবং কমন মিডলওয়্যার প্রয়োগ সহজ করে।

## বেসিক গ্রুপ

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    // /api গ্রুপ তৈরি
    api := app.Group("/api")
    
    // /api/users
    api.Get("/users", func(c fiber.Ctx) error {
        return c.SendString("ইউজার লিস্ট")
    })
    
    // /api/products
    api.Get("/products", func(c fiber.Ctx) error {
        return c.SendString("প্রোডাক্ট লিস্ট")
    })

    log.Fatal(app.Listen(":3000"))
}
```

## নেস্টেড গ্রুপ

```go
app := fiber.New()

// প্রথম স্তর: /api
api := app.Group("/api")

// দ্বিতীয় স্তর: /api/v1
v1 := api.Group("/v1")
v1.Get("/users", getUsers)     // /api/v1/users
v1.Get("/products", getProducts) // /api/v1/products

// দ্বিতীয় স্তর: /api/v2
v2 := api.Group("/v2")
v2.Get("/users", getUsersV2)   // /api/v2/users
v2.Get("/products", getProductsV2) // /api/v2/products
```

## গ্রুপে মিডলওয়্যার যোগ করা

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

// অথেনটিকেশন মিডলওয়্যার
func authMiddleware(c fiber.Ctx) error {
    token := c.Get("Authorization")
    if token == "" {
        return c.Status(401).JSON(fiber.Map{
            "error": "অননুমোদিত",
        })
    }
    return c.Next()
}

// লগার মিডলওয়্যার
func loggerMiddleware(c fiber.Ctx) error {
    log.Printf("%s %s", c.Method(), c.Path())
    return c.Next()
}

func main() {
    app := fiber.New()

    // API গ্রুপে লগার মিডলওয়্যার
    api := app.Group("/api", loggerMiddleware)

    // পাবলিক রাউট (শুধু লগার)
    public := api.Group("/public")
    public.Get("/info", func(c fiber.Ctx) error {
        return c.SendString("পাবলিক ইনফো")
    })

    // প্রাইভেট রাউট (লগার + অথ)
    private := api.Group("/private", authMiddleware)
    private.Get("/dashboard", func(c fiber.Ctx) error {
        return c.SendString("প্রাইভেট ড্যাশবোর্ড")
    })
    private.Get("/profile", func(c fiber.Ctx) error {
        return c.SendString("প্রাইভেট প্রোফাইল")
    })

    log.Fatal(app.Listen(":3000"))
}
```

## API ভার্সনিং

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    api := app.Group("/api")

    // Version 1
    v1 := api.Group("/v1")
    setupV1Routes(v1)

    // Version 2 (নতুন ফিচার সহ)
    v2 := api.Group("/v2")
    setupV2Routes(v2)

    log.Fatal(app.Listen(":3000"))
}

func setupV1Routes(router fiber.Router) {
    router.Get("/users", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "version": "v1",
            "users":   []string{"Rahim", "Karim"},
        })
    })
}

func setupV2Routes(router fiber.Router) {
    router.Get("/users", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "version": "v2",
            "data": fiber.Map{
                "users": []fiber.Map{
                    {"id": 1, "name": "Rahim", "email": "rahim@example.com"},
                    {"id": 2, "name": "Karim", "email": "karim@example.com"},
                },
                "total": 2,
            },
        })
    })
}
```

## CRUD অপারেশন সংগঠিত করা

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    api := app.Group("/api/v1")

    // User routes
    users := api.Group("/users")
    users.Get("/", getAllUsers)       // GET /api/v1/users
    users.Get("/:id", getUser)        // GET /api/v1/users/:id
    users.Post("/", createUser)       // POST /api/v1/users
    users.Put("/:id", updateUser)     // PUT /api/v1/users/:id
    users.Delete("/:id", deleteUser)  // DELETE /api/v1/users/:id

    // Product routes
    products := api.Group("/products")
    products.Get("/", getAllProducts)
    products.Get("/:id", getProduct)
    products.Post("/", createProduct)
    products.Put("/:id", updateProduct)
    products.Delete("/:id", deleteProduct)

    // Order routes (নেস্টেড)
    orders := users.Group("/:userId/orders")
    orders.Get("/", getUserOrders)    // GET /api/v1/users/:userId/orders
    orders.Post("/", createOrder)     // POST /api/v1/users/:userId/orders

    log.Fatal(app.Listen(":3000"))
}

// হ্যান্ডলার ফাংশন
func getAllUsers(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"users": []string{}})
}

func getUser(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"user": c.Params("id")})
}

func createUser(c fiber.Ctx) error {
    return c.Status(201).JSON(fiber.Map{"message": "তৈরি হয়েছে"})
}

func updateUser(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"message": "আপডেট হয়েছে"})
}

func deleteUser(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"message": "মুছে ফেলা হয়েছে"})
}

func getAllProducts(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"products": []string{}})
}

func getProduct(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"product": c.Params("id")})
}

func createProduct(c fiber.Ctx) error {
    return c.Status(201).JSON(fiber.Map{"message": "তৈরি হয়েছে"})
}

func updateProduct(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"message": "আপডেট হয়েছে"})
}

func deleteProduct(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"message": "মুছে ফেলা হয়েছে"})
}

func getUserOrders(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "userId": c.Params("userId"),
        "orders": []string{},
    })
}

func createOrder(c fiber.Ctx) error {
    return c.Status(201).JSON(fiber.Map{
        "userId":  c.Params("userId"),
        "message": "অর্ডার তৈরি হয়েছে",
    })
}
```

## রোল-ভিত্তিক গ্রুপিং

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

// রোল চেক মিডলওয়্যার
func requireRole(role string) fiber.Handler {
    return func(c fiber.Ctx) error {
        userRole := c.Get("X-User-Role")
        if userRole != role {
            return c.Status(403).JSON(fiber.Map{
                "error": "এই রিসোর্সে অ্যাক্সেস নেই",
            })
        }
        return c.Next()
    }
}

func main() {
    app := fiber.New()

    // পাবলিক রাউট
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("হোম পেজ")
    })

    // ইউজার রাউট (লগড ইন ইউজার)
    user := app.Group("/user", requireRole("user"))
    user.Get("/profile", func(c fiber.Ctx) error {
        return c.SendString("ইউজার প্রোফাইল")
    })

    // মডারেটর রাউট
    moderator := app.Group("/mod", requireRole("moderator"))
    moderator.Get("/reports", func(c fiber.Ctx) error {
        return c.SendString("রিপোর্ট তালিকা")
    })

    // অ্যাডমিন রাউট
    admin := app.Group("/admin", requireRole("admin"))
    admin.Get("/dashboard", func(c fiber.Ctx) error {
        return c.SendString("অ্যাডমিন ড্যাশবোর্ড")
    })
    admin.Get("/users", func(c fiber.Ctx) error {
        return c.SendString("সব ইউজার")
    })
    admin.Delete("/users/:id", func(c fiber.Ctx) error {
        return c.SendString("ইউজার মুছে ফেলা হয়েছে")
    })

    log.Fatal(app.Listen(":3000"))
}
```

## মাউন্ট সাব-অ্যাপ

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    // মূল অ্যাপ
    app := fiber.New()
    
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("মূল অ্যাপ")
    })

    // API সাব-অ্যাপ
    apiApp := fiber.New()
    apiApp.Get("/users", func(c fiber.Ctx) error {
        return c.SendString("API Users")
    })
    apiApp.Get("/products", func(c fiber.Ctx) error {
        return c.SendString("API Products")
    })

    // সাব-অ্যাপ মাউন্ট করুন
    app.Use("/api", apiApp)
    // এখন /api/users এবং /api/products কাজ করবে

    log.Fatal(app.Listen(":3000"))
}
```

## টেস্ট করুন

```bash
# পাবলিক রাউট
curl http://localhost:3000/api/public/info

# প্রাইভেট রাউট (অথ ছাড়া - 401)
curl http://localhost:3000/api/private/dashboard

# প্রাইভেট রাউট (অথ সহ)
curl -H "Authorization: Bearer token123" http://localhost:3000/api/private/dashboard

# API ভার্সন
curl http://localhost:3000/api/v1/users
curl http://localhost:3000/api/v2/users

# নেস্টেড রাউট
curl http://localhost:3000/api/v1/users/123/orders
```

---

[← আগের: রাউট প্যারামিটার](02-route-parameters.md) | [পরবর্তী: স্ট্যাটিক ফাইল →](04-static-files.md)
