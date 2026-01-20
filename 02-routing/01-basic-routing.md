# বেসিক রাউটিং

## রাউটিং কী?

রাউটিং হলো কোন URL-এ কী রেসপন্স দেওয়া হবে তা নির্ধারণ করা। যখন ক্লায়েন্ট কোনো নির্দিষ্ট URL-এ রিকোয়েস্ট পাঠায়, সার্ভার সেই URL-এর জন্য নির্ধারিত হ্যান্ডলার ফাংশন চালায়।

## রাউট ডিফাইন করার ফরম্যাট

```go
app.Method(path string, handler func(fiber.Ctx) error)
```

- **app** - Fiber অ্যাপ ইনস্ট্যান্স
- **Method** - HTTP মেথড (Get, Post, Put, Delete ইত্যাদি)
- **path** - URL পাথ
- **handler** - রিকোয়েস্ট হ্যান্ডল করার ফাংশন

## HTTP মেথড সমূহ

```go
package main

import (
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    // GET - ডেটা পড়তে
    app.Get("/users", func(c fiber.Ctx) error {
        return c.SendString("সব ইউজার দেখুন")
    })

    // POST - নতুন ডেটা তৈরি করতে
    app.Post("/users", func(c fiber.Ctx) error {
        return c.SendString("নতুন ইউজার তৈরি করুন")
    })

    // PUT - সম্পূর্ণ ডেটা আপডেট করতে
    app.Put("/users/:id", func(c fiber.Ctx) error {
        return c.SendString("ইউজার সম্পূর্ণ আপডেট করুন")
    })

    // PATCH - আংশিক ডেটা আপডেট করতে
    app.Patch("/users/:id", func(c fiber.Ctx) error {
        return c.SendString("ইউজার আংশিক আপডেট করুন")
    })

    // DELETE - ডেটা মুছতে
    app.Delete("/users/:id", func(c fiber.Ctx) error {
        return c.SendString("ইউজার মুছুন")
    })

    // HEAD - হেডার পেতে (বডি ছাড়া)
    app.Head("/users", func(c fiber.Ctx) error {
        return c.SendStatus(200)
    })

    // OPTIONS - সাপোর্টেড মেথড জানতে
    app.Options("/users", func(c fiber.Ctx) error {
        c.Set("Allow", "GET, POST, PUT, DELETE")
        return c.SendStatus(204)
    })

    app.Listen(":3000")
}
```

## সব মেথডে একই হ্যান্ডলার

```go
// All - সব HTTP মেথডে কাজ করে
app.All("/api", func(c fiber.Ctx) error {
    return c.SendString("যেকোনো মেথড: " + c.Method())
})

// Add - নির্দিষ্ট মেথড গুলোতে কাজ করে
app.Add([]string{"GET", "POST"}, "/both", func(c fiber.Ctx) error {
    return c.SendString("GET অথবা POST")
})
```

## স্ট্যাটিক রাউট

```go
// সরাসরি পাথ ম্যাচ
app.Get("/", func(c fiber.Ctx) error {
    return c.SendString("হোম পেজ")
})

app.Get("/about", func(c fiber.Ctx) error {
    return c.SendString("আমাদের সম্পর্কে")
})

app.Get("/contact", func(c fiber.Ctx) error {
    return c.SendString("যোগাযোগ")
})

app.Get("/products/list", func(c fiber.Ctx) error {
    return c.SendString("পণ্যের তালিকা")
})
```

## রাউট চেইনিং

একই পাথে বিভিন্ন মেথড সংজ্ঞায়িত করতে `RouteChain` ব্যবহার করুন:

```go
app.RouteChain("/user/:id").
    Get(func(c fiber.Ctx) error {
        return c.SendString("ইউজার দেখুন: " + c.Params("id"))
    }).
    Put(func(c fiber.Ctx) error {
        return c.SendString("ইউজার আপডেট: " + c.Params("id"))
    }).
    Delete(func(c fiber.Ctx) error {
        return c.SendString("ইউজার মুছুন: " + c.Params("id"))
    })
```

## রাউটের নাম দেওয়া

```go
// রাউটে নাম দিন
app.Get("/user/:id", func(c fiber.Ctx) error {
    return c.SendString("ইউজার: " + c.Params("id"))
}).Name("user.show")

// নাম দিয়ে URL জেনারেট করুন
app.Get("/", func(c fiber.Ctx) error {
    url, _ := c.GetRouteURL("user.show", fiber.Map{"id": "123"})
    return c.SendString("ইউজার URL: " + url)
    // আউটপুট: /user/123
})
```

## রাউট তালিকা দেখা

```go
package main

import (
    "encoding/json"
    "fmt"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Get("/", handler).Name("home")
    app.Get("/users", handler).Name("users.list")
    app.Post("/users", handler).Name("users.create")
    app.Get("/users/:id", handler).Name("users.show")

    // সব রাউট প্রিন্ট করুন
    routes := app.GetRoutes(true) // true = Use রাউট বাদ দিন
    data, _ := json.MarshalIndent(routes, "", "  ")
    fmt.Println(string(data))

    app.Listen(":3000")
}

func handler(c fiber.Ctx) error {
    return c.SendStatus(200)
}
```

## রাউট প্রায়োরিটি

রাউট যে ক্রমে ডিফাইন করা হয়, সেই ক্রমে ম্যাচ হয়। প্রথম ম্যাচ হওয়া রাউটই চালানো হয়।

```go
// এটি প্রথমে চেক হবে
app.Get("/users/new", func(c fiber.Ctx) error {
    return c.SendString("নতুন ইউজার ফর্ম")
})

// এটি পরে চেক হবে
app.Get("/users/:id", func(c fiber.Ctx) error {
    return c.SendString("ইউজার ID: " + c.Params("id"))
})

// যদি উল্টো হয়, /users/new কখনো কাজ করবে না
// কারণ :id প্যারামিটার "new" কেও ম্যাচ করে ফেলবে
```

## 404 Not Found হ্যান্ডলার

```go
// সব রাউটের শেষে রাখুন
app.Use(func(c fiber.Ctx) error {
    return c.Status(404).JSON(fiber.Map{
        "error":   "পৃষ্ঠা পাওয়া যায়নি",
        "path":    c.Path(),
        "method":  c.Method(),
    })
})
```

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    // হোম
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("🏠 হোম পেজ")
    })

    // পণ্য তালিকা
    app.Get("/products", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "products": []string{"মোবাইল", "ল্যাপটপ", "ট্যাবলেট"},
        })
    })

    // নির্দিষ্ট পণ্য
    app.Get("/products/:id", func(c fiber.Ctx) error {
        id := c.Params("id")
        return c.JSON(fiber.Map{
            "id":   id,
            "name": "পণ্য " + id,
        })
    })

    // পণ্য তৈরি
    app.Post("/products", func(c fiber.Ctx) error {
        return c.Status(201).JSON(fiber.Map{
            "message": "পণ্য তৈরি হয়েছে",
        })
    })

    // 404 হ্যান্ডলার
    app.Use(func(c fiber.Ctx) error {
        return c.Status(404).SendString("পৃষ্ঠা পাওয়া যায়নি 😢")
    })

    log.Fatal(app.Listen(":3000"))
}
```

## টেস্ট করুন

```bash
# GET রিকোয়েস্ট
curl http://localhost:3000/
curl http://localhost:3000/products
curl http://localhost:3000/products/123

# POST রিকোয়েস্ট
curl -X POST http://localhost:3000/products

# 404 টেস্ট
curl http://localhost:3000/unknown
```

---

[← আগের: প্রজেক্ট স্ট্রাকচার](../01-basics/04-project-structure.md) | [পরবর্তী: রাউট প্যারামিটার →](02-route-parameters.md)
