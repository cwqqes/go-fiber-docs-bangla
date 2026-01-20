# রেসপন্স মেথড

## টেক্সট রেসপন্স

```go
// সাধারণ স্ট্রিং
app.Get("/text", func(c fiber.Ctx) error {
    return c.SendString("সাধারণ টেক্সট রেসপন্স")
})

// ফরম্যাটেড স্ট্রিং
app.Get("/hello/:name", func(c fiber.Ctx) error {
    name := c.Params("name")
    return c.SendString(fmt.Sprintf("হ্যালো, %s!", name))
})
```

## JSON রেসপন্স

```go
// fiber.Map ব্যবহার করে
app.Get("/json", func(c fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "message": "সফল",
        "code":    200,
        "data":    nil,
    })
})

// Struct থেকে
type User struct {
    ID        int       `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

app.Get("/user/:id", func(c fiber.Ctx) error {
    user := User{
        ID:        1,
        Name:      "রহিম",
        Email:     "rahim@example.com",
        CreatedAt: time.Now(),
    }
    return c.JSON(user)
})

// Array রেসপন্স
app.Get("/users", func(c fiber.Ctx) error {
    users := []User{
        {ID: 1, Name: "রহিম"},
        {ID: 2, Name: "করিম"},
    }
    return c.JSON(users)
})
```

## JSONP রেসপন্স

```go
app.Get("/jsonp", func(c fiber.Ctx) error {
    callback := c.Query("callback", "callback")
    return c.JSONP(fiber.Map{
        "data": "value",
    }, callback)
})
// আউটপুট: callback({"data":"value"})
```

## XML রেসপন্স

```go
type Product struct {
    XMLName xml.Name `xml:"product"`
    ID      int      `xml:"id"`
    Name    string   `xml:"name"`
    Price   float64  `xml:"price"`
}

app.Get("/xml", func(c fiber.Ctx) error {
    product := Product{
        ID:    1,
        Name:  "মোবাইল",
        Price: 15000,
    }
    return c.XML(product)
})
```

## HTML রেসপন্স

```go
app.Get("/html", func(c fiber.Ctx) error {
    html := `
    <!DOCTYPE html>
    <html>
    <head><title>পৃষ্ঠা</title></head>
    <body>
        <h1>স্বাগতম!</h1>
    </body>
    </html>
    `
    c.Set("Content-Type", "text/html")
    return c.SendString(html)
})
```

## টেমপ্লেট রেন্ডার

```go
import "github.com/gofiber/template/html/v2"

// সেটআপ
engine := html.New("./views", ".html")
app := fiber.New(fiber.Config{
    Views: engine,
})

app.Get("/", func(c fiber.Ctx) error {
    return c.Render("index", fiber.Map{
        "Title":   "হোম পেজ",
        "Message": "স্বাগতম!",
    })
})
```

## স্ট্যাটাস কোড

```go
// স্ট্যাটাস সহ রেসপন্স
app.Get("/created", func(c fiber.Ctx) error {
    return c.Status(201).JSON(fiber.Map{
        "message": "রিসোর্স তৈরি হয়েছে",
    })
})

app.Get("/not-found", func(c fiber.Ctx) error {
    return c.Status(404).JSON(fiber.Map{
        "error": "পাওয়া যায়নি",
    })
})

// শুধু স্ট্যাটাস কোড
app.Get("/no-content", func(c fiber.Ctx) error {
    return c.SendStatus(204) // No Content
})

// স্ট্যাটাস কনস্ট্যান্ট
app.Get("/constants", func(c fiber.Ctx) error {
    return c.Status(fiber.StatusOK).SendString("OK")
    // fiber.StatusCreated (201)
    // fiber.StatusBadRequest (400)
    // fiber.StatusUnauthorized (401)
    // fiber.StatusForbidden (403)
    // fiber.StatusNotFound (404)
    // fiber.StatusInternalServerError (500)
})
```

## ফাইল রেসপন্স

```go
// ফাইল সার্ভ
app.Get("/file", func(c fiber.Ctx) error {
    return c.SendFile("./files/document.pdf")
})

// ডাউনলোড হিসেবে
app.Get("/download", func(c fiber.Ctx) error {
    return c.Download("./files/report.pdf", "monthly-report.pdf")
})

// Attachment হিসেবে
app.Get("/attachment", func(c fiber.Ctx) error {
    c.Attachment("invoice.pdf")
    return c.SendFile("./files/invoice.pdf")
})
```

## রিডাইরেক্ট

```go
// সাধারণ রিডাইরেক্ট (302)
app.Get("/old", func(c fiber.Ctx) error {
    return c.Redirect("/new")
})

// পার্মানেন্ট রিডাইরেক্ট (301)
app.Get("/moved", func(c fiber.Ctx) error {
    return c.Redirect("/new-location", 301)
})

// এক্সটার্নাল রিডাইরেক্ট
app.Get("/external", func(c fiber.Ctx) error {
    return c.Redirect("https://google.com")
})

// ব্যাক রিডাইরেক্ট
app.Get("/back", func(c fiber.Ctx) error {
    return c.RedirectBack("/fallback")
})

// রাউট নাম দিয়ে
app.Get("/profile", func(c fiber.Ctx) error {
    url, _ := c.GetRouteURL("user.show", fiber.Map{"id": "123"})
    return c.Redirect(url)
})
```

## হেডার সেট করা

```go
app.Get("/headers", func(c fiber.Ctx) error {
    // একটি হেডার
    c.Set("X-Custom-Header", "MyValue")
    
    // Content-Type
    c.Set("Content-Type", "application/json; charset=utf-8")
    
    // Cache-Control
    c.Set("Cache-Control", "public, max-age=3600")
    
    // CORS হেডার
    c.Set("Access-Control-Allow-Origin", "*")
    
    return c.JSON(fiber.Map{"status": "ok"})
})

// Append হেডার
app.Get("/multi", func(c fiber.Ctx) error {
    c.Append("X-Values", "value1")
    c.Append("X-Values", "value2")
    // X-Values: value1, value2
    
    return c.SendString("OK")
})
```

## কুকি রেসপন্স

```go
app.Get("/set-cookie", func(c fiber.Ctx) error {
    cookie := &fiber.Cookie{
        Name:     "session_id",
        Value:    "abc123xyz",
        Expires:  time.Now().Add(24 * time.Hour),
        HTTPOnly: true,
        Secure:   true,
        SameSite: "Strict",
        Path:     "/",
    }
    
    c.Cookie(cookie)
    return c.SendString("Cookie set!")
})

// কুকি ডিলিট
app.Get("/logout", func(c fiber.Ctx) error {
    c.ClearCookie("session_id")
    // অথবা
    c.Cookie(&fiber.Cookie{
        Name:    "session_id",
        Value:   "",
        Expires: time.Now().Add(-time.Hour),
    })
    
    return c.Redirect("/login")
})
```

## স্ট্রিমিং রেসপন্স

```go
app.Get("/stream", func(c fiber.Ctx) error {
    c.Set("Content-Type", "text/event-stream")
    c.Set("Cache-Control", "no-cache")
    c.Set("Connection", "keep-alive")
    
    c.Context().SetBodyStreamWriter(func(w *bufio.Writer) {
        for i := 0; i < 10; i++ {
            fmt.Fprintf(w, "data: Message %d\n\n", i)
            w.Flush()
            time.Sleep(time.Second)
        }
    })
    
    return nil
})
```

## রেসপন্স সারসংক্ষেপ

| মেথড | বর্ণনা |
|------|--------|
| `c.SendString(s)` | টেক্সট রেসপন্স |
| `c.JSON(v)` | JSON রেসপন্স |
| `c.XML(v)` | XML রেসপন্স |
| `c.JSONP(v, callback)` | JSONP রেসপন্স |
| `c.SendFile(path)` | ফাইল সার্ভ |
| `c.Download(path, name)` | ফাইল ডাউনলোড |
| `c.Status(code)` | স্ট্যাটাস কোড |
| `c.SendStatus(code)` | শুধু স্ট্যাটাস |
| `c.Redirect(url)` | রিডাইরেক্ট |
| `c.Set(key, value)` | হেডার সেট |
| `c.Cookie(cookie)` | কুকি সেট |
| `c.Render(template, data)` | টেমপ্লেট রেন্ডার |

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

    // JSON API
    app.Get("/api/users", func(c fiber.Ctx) error {
        users := []fiber.Map{
            {"id": 1, "name": "রহিম"},
            {"id": 2, "name": "করিম"},
        }
        
        c.Set("X-Total-Count", "2")
        return c.JSON(fiber.Map{
            "success": true,
            "data":    users,
        })
    })

    // এরর রেসপন্স
    app.Get("/api/error", func(c fiber.Ctx) error {
        return c.Status(500).JSON(fiber.Map{
            "success": false,
            "error": fiber.Map{
                "code":    "INTERNAL_ERROR",
                "message": "কিছু একটা ভুল হয়েছে",
            },
        })
    })

    // ফাইল ডাউনলোড
    app.Get("/download/:file", func(c fiber.Ctx) error {
        filename := c.Params("file")
        return c.Download("./files/"+filename, filename)
    })

    // রিডাইরেক্ট
    app.Get("/old-api/*", func(c fiber.Ctx) error {
        newPath := "/api/v2/" + c.Params("*")
        return c.Redirect(newPath, 301)
    })

    log.Fatal(app.Listen(":3000"))
}
```

---

[← আগের: রিকোয়েস্ট হ্যান্ডলিং](02-request-handling.md) | [পরবর্তী: লোকালস →](04-locals.md)
