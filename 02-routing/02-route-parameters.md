# রাউট প্যারামিটার

## প্যারামিটার কী?

প্যারামিটার হলো URL-এর ডায়নামিক অংশ যা রান টাইমে পরিবর্তন হতে পারে। উদাহরণ: `/users/123` এ `123` হলো প্যারামিটার।

## বেসিক প্যারামিটার

```go
// :name সিনট্যাক্স ব্যবহার করুন
app.Get("/user/:name", func(c fiber.Ctx) error {
    name := c.Params("name")
    return c.SendString("হ্যালো " + name)
})

// http://localhost:3000/user/rahim
// আউটপুট: হ্যালো rahim
```

## একাধিক প্যারামিটার

```go
app.Get("/user/:name/book/:title", func(c fiber.Ctx) error {
    name := c.Params("name")
    title := c.Params("title")
    return c.JSON(fiber.Map{
        "user":  name,
        "book":  title,
    })
})

// http://localhost:3000/user/rahim/book/golang
// আউটপুট: {"user":"rahim","book":"golang"}
```

## অপশনাল প্যারামিটার

`?` চিহ্ন দিয়ে প্যারামিটার অপশনাল করুন:

```go
app.Get("/user/:name?", func(c fiber.Ctx) error {
    name := c.Params("name")
    if name == "" {
        return c.SendString("অতিথি ইউজার")
    }
    return c.SendString("হ্যালো " + name)
})

// http://localhost:3000/user → অতিথি ইউজার
// http://localhost:3000/user/rahim → হ্যালো rahim
```

## ওয়াইল্ডকার্ড প্যারামিটার

`*` চিহ্ন দিয়ে বাকি সব পাথ ক্যাপচার করুন:

```go
app.Get("/api/*", func(c fiber.Ctx) error {
    wildcard := c.Params("*")
    return c.SendString("API পাথ: " + wildcard)
})

// http://localhost:3000/api/user/123/orders
// আউটপুট: API পাথ: user/123/orders
```

## প্লাস (+) প্যারামিটার

`+` চিহ্ন বাধ্যতামূলক ওয়াইল্ডকার্ড:

```go
app.Get("/files/+", func(c fiber.Ctx) error {
    filePath := c.Params("+")
    return c.SendString("ফাইল পাথ: " + filePath)
})

// http://localhost:3000/files/images/photo.jpg
// আউটপুট: ফাইল পাথ: images/photo.jpg

// http://localhost:3000/files → 404 (+ বাধ্যতামূলক)
```

## প্যারামিটার ভ্যালিডেশন

### টাইপ কনভার্সন

```go
import "strconv"

app.Get("/user/:id", func(c fiber.Ctx) error {
    idStr := c.Params("id")
    
    // স্ট্রিং থেকে int
    id, err := strconv.Atoi(idStr)
    if err != nil {
        return c.Status(400).JSON(fiber.Map{
            "error": "ID অবশ্যই সংখ্যা হতে হবে",
        })
    }
    
    return c.JSON(fiber.Map{
        "id": id,
    })
})
```

### ফাইবারের বিল্ট-ইন পার্সার

```go
// ParamsInt সরাসরি int রিটার্ন করে
app.Get("/user/:id", func(c fiber.Ctx) error {
    id := c.Params("id")
    // ডিফল্ট ভ্যালু সহ
    // id, err := c.ParamsInt("id", 0)
    
    return c.JSON(fiber.Map{
        "id": id,
    })
})
```

## ডট এবং হাইফেন

ডট (.) এবং হাইফেন (-) লিটারেলি ম্যাচ হয়:

```go
// ডট সহ রাউট
app.Get("/plant/:genus.:species", func(c fiber.Ctx) error {
    genus := c.Params("genus")
    species := c.Params("species")
    return c.SendString(genus + "." + species)
})

// http://localhost:3000/plant/rosa.indica
// আউটপুট: rosa.indica

// হাইফেন সহ রাউট
app.Get("/flight/:from-:to", func(c fiber.Ctx) error {
    from := c.Params("from")
    to := c.Params("to")
    return c.SendString(from + " থেকে " + to)
})

// http://localhost:3000/flight/dhaka-chittagong
// আউটপুট: dhaka থেকে chittagong
```

## এস্কেপ ক্যারেক্টার

বিশেষ ক্যারেক্টার এস্কেপ করতে `\` ব্যবহার করুন:

```go
app.Get(`/api/resource/name\:customVerb`, func(c fiber.Ctx) error {
    return c.SendString("কাস্টম মেথড")
})

// http://localhost:3000/api/resource/name:customVerb
```

## Query Parameters

URL এ `?` এর পরের অংশ:

```go
app.Get("/search", func(c fiber.Ctx) error {
    // সিঙ্গেল ক্যোয়ারি প্যারামিটার
    q := c.Query("q")
    page := c.Query("page", "1")      // ডিফল্ট ভ্যালু সহ
    limit := c.Query("limit", "10")

    return c.JSON(fiber.Map{
        "query": q,
        "page":  page,
        "limit": limit,
    })
})

// http://localhost:3000/search?q=golang&page=2&limit=20
```

### সব ক্যোয়ারি প্যারামিটার

```go
app.Get("/search", func(c fiber.Ctx) error {
    // সব ক্যোয়ারি প্যারামিটার ম্যাপ হিসেবে
    queries := c.Queries()
    return c.JSON(queries)
})
```

### স্ট্রাক্টে পার্স করা

```go
app.Get("/search", func(c fiber.Ctx) error {
    type SearchQuery struct {
        Q     string `query:"q"`
        Page  int    `query:"page"`
        Limit int    `query:"limit"`
    }

    var search SearchQuery
    if err := c.Bind().Query(&search); err != nil {
        return c.Status(400).SendString("Invalid query")
    }

    return c.JSON(search)
})
```

## Path vs Query প্যারামিটার

| বৈশিষ্ট্য | Path Parameter | Query Parameter |
|----------|----------------|-----------------|
| সিনট্যাক্স | `/user/:id` | `/user?id=123` |
| অ্যাক্সেস | `c.Params("id")` | `c.Query("id")` |
| ব্যবহার | রিসোর্স আইডেন্টিফাই | ফিল্টার/অপশন |
| উদাহরণ | `/users/123` | `/users?page=2` |

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    // বেসিক প্যারামিটার
    app.Get("/user/:id", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "user_id": c.Params("id"),
        })
    })

    // একাধিক প্যারামিটার
    app.Get("/post/:year/:month/:day/:slug", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "year":  c.Params("year"),
            "month": c.Params("month"),
            "day":   c.Params("day"),
            "slug":  c.Params("slug"),
        })
    })

    // অপশনাল প্যারামিটার
    app.Get("/greet/:name?", func(c fiber.Ctx) error {
        name := c.Params("name", "অতিথি")
        return c.SendString("স্বাগতম, " + name + "!")
    })

    // ওয়াইল্ডকার্ড
    app.Get("/static/*", func(c fiber.Ctx) error {
        return c.SendString("ফাইল: " + c.Params("*"))
    })

    // ক্যোয়ারি প্যারামিটার
    app.Get("/products", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "category": c.Query("category", "all"),
            "sort":     c.Query("sort", "newest"),
            "page":     c.Query("page", "1"),
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

## টেস্ট করুন

```bash
# বেসিক প্যারামিটার
curl http://localhost:3000/user/123

# একাধিক প্যারামিটার
curl http://localhost:3000/post/2024/01/15/hello-world

# অপশনাল প্যারামিটার
curl http://localhost:3000/greet
curl http://localhost:3000/greet/Rahim

# ওয়াইল্ডকার্ড
curl http://localhost:3000/static/images/logo.png

# ক্যোয়ারি প্যারামিটার
curl "http://localhost:3000/products?category=phones&sort=price"
```

---

[← আগের: বেসিক রাউটিং](01-basic-routing.md) | [পরবর্তী: রাউট গ্রুপিং →](03-route-groups.md)
