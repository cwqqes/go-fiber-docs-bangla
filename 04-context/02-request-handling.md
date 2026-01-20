# রিকোয়েস্ট হ্যান্ডলিং

## JSON বডি পার্স করা

```go
type CreateUserRequest struct {
    Name     string `json:"name"`
    Email    string `json:"email"`
    Password string `json:"password"`
}

app.Post("/users", func(c fiber.Ctx) error {
    var req CreateUserRequest
    
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{
            "error": "অবৈধ JSON",
        })
    }
    
    return c.JSON(fiber.Map{
        "message": "ইউজার তৈরি হয়েছে",
        "user":    req,
    })
})
```

## ফর্ম ডেটা পার্স করা

### URL-encoded ফর্ম

```go
app.Post("/login", func(c fiber.Ctx) error {
    // সিঙ্গেল ভ্যালু
    username := c.FormValue("username")
    password := c.FormValue("password")
    
    return c.JSON(fiber.Map{
        "username": username,
        "password": "***",
    })
})

// স্ট্রাক্টে পার্স
type LoginForm struct {
    Username string `form:"username"`
    Password string `form:"password"`
}

app.Post("/login", func(c fiber.Ctx) error {
    var form LoginForm
    
    if err := c.Bind().Form(&form); err != nil {
        return c.Status(400).SendString("ফর্ম পার্স ব্যর্থ")
    }
    
    return c.JSON(form)
})
```

### Multipart ফর্ম

```go
app.Post("/upload", func(c fiber.Ctx) error {
    // ফাইল আপলোড
    file, err := c.FormFile("avatar")
    if err != nil {
        return c.Status(400).SendString("ফাইল পাওয়া যায়নি")
    }
    
    // অন্যান্য ফিল্ড
    name := c.FormValue("name")
    
    // ফাইল সেভ করুন
    c.SaveFile(file, "./uploads/"+file.Filename)
    
    return c.JSON(fiber.Map{
        "name":     name,
        "filename": file.Filename,
        "size":     file.Size,
    })
})
```

## ক্যোয়ারি প্যারামিটার পার্স

```go
type SearchQuery struct {
    Q        string `query:"q"`
    Page     int    `query:"page"`
    Limit    int    `query:"limit"`
    SortBy   string `query:"sort_by"`
    Order    string `query:"order"`
}

app.Get("/search", func(c fiber.Ctx) error {
    var query SearchQuery
    
    if err := c.Bind().Query(&query); err != nil {
        return c.Status(400).SendString("অবৈধ ক্যোয়ারি")
    }
    
    // ডিফল্ট ভ্যালু
    if query.Page == 0 {
        query.Page = 1
    }
    if query.Limit == 0 {
        query.Limit = 10
    }
    
    return c.JSON(query)
})
```

## ভ্যালিডেশন

```go
import "github.com/go-playground/validator/v10"

var validate = validator.New()

type CreateUserRequest struct {
    Name     string `json:"name" validate:"required,min=3,max=50"`
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=6"`
    Age      int    `json:"age" validate:"gte=18,lte=100"`
}

app.Post("/users", func(c fiber.Ctx) error {
    var req CreateUserRequest
    
    // JSON পার্স
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{
            "error": "অবৈধ JSON",
        })
    }
    
    // ভ্যালিডেশন
    if err := validate.Struct(req); err != nil {
        var errors []string
        for _, err := range err.(validator.ValidationErrors) {
            errors = append(errors, err.Field()+" "+err.Tag())
        }
        return c.Status(400).JSON(fiber.Map{
            "error":   "ভ্যালিডেশন ব্যর্থ",
            "details": errors,
        })
    }
    
    return c.Status(201).JSON(fiber.Map{
        "message": "ইউজার তৈরি হয়েছে",
    })
})
```

## Raw Body

```go
app.Post("/webhook", func(c fiber.Ctx) error {
    // Raw বডি পান
    body := c.Body()
    
    // স্ট্রিং হিসেবে
    bodyString := string(body)
    
    return c.JSON(fiber.Map{
        "received": len(body),
        "preview":  bodyString[:min(100, len(bodyString))],
    })
})
```

## মাল্টিপার্ট ফর্ম (একাধিক ফাইল)

```go
app.Post("/upload-multiple", func(c fiber.Ctx) error {
    form, err := c.MultipartForm()
    if err != nil {
        return c.Status(400).SendString("ফর্ম পার্স ব্যর্থ")
    }
    
    // ফাইল সমূহ
    files := form.File["files"]
    
    var uploaded []string
    for _, file := range files {
        dst := "./uploads/" + file.Filename
        c.SaveFile(file, dst)
        uploaded = append(uploaded, file.Filename)
    }
    
    // অন্যান্য ফিল্ড
    values := form.Value
    
    return c.JSON(fiber.Map{
        "files":  uploaded,
        "values": values,
    })
})
```

## Content-Type অনুযায়ী পার্স

```go
app.Post("/data", func(c fiber.Ctx) error {
    contentType := c.Get("Content-Type")
    
    switch {
    case strings.Contains(contentType, "application/json"):
        var data map[string]interface{}
        c.Bind().JSON(&data)
        return c.JSON(fiber.Map{"type": "json", "data": data})
        
    case strings.Contains(contentType, "application/x-www-form-urlencoded"):
        name := c.FormValue("name")
        return c.JSON(fiber.Map{"type": "form", "name": name})
        
    case strings.Contains(contentType, "multipart/form-data"):
        file, _ := c.FormFile("file")
        return c.JSON(fiber.Map{"type": "multipart", "file": file.Filename})
        
    default:
        return c.Status(415).SendString("Unsupported Media Type")
    }
})
```

## রিকোয়েস্ট বডি লিমিট

```go
// গ্লোবাল লিমিট
app := fiber.New(fiber.Config{
    BodyLimit: 10 * 1024 * 1024, // 10MB
})

// নির্দিষ্ট রাউটে লিমিট
import "github.com/gofiber/fiber/v3/middleware/limiter"

app.Post("/upload", limiter.New(limiter.Config{
    Max: 1,
}), func(c fiber.Ctx) error {
    // ...
})
```

## XML পার্স করা

```go
type XMLData struct {
    Name  string `xml:"name"`
    Value int    `xml:"value"`
}

app.Post("/xml", func(c fiber.Ctx) error {
    var data XMLData
    
    if err := c.Bind().XML(&data); err != nil {
        return c.Status(400).SendString("অবৈধ XML")
    }
    
    return c.JSON(data)
})
```

## কাস্টম বডি পার্সার

```go
import "encoding/csv"

app.Post("/csv", func(c fiber.Ctx) error {
    body := c.Body()
    reader := csv.NewReader(strings.NewReader(string(body)))
    
    records, err := reader.ReadAll()
    if err != nil {
        return c.Status(400).SendString("অবৈধ CSV")
    }
    
    return c.JSON(fiber.Map{
        "rows":  len(records),
        "data":  records,
    })
})
```

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
    "github.com/go-playground/validator/v10"
)

var validate = validator.New()

type Product struct {
    Name        string  `json:"name" validate:"required,min=3"`
    Description string  `json:"description"`
    Price       float64 `json:"price" validate:"required,gt=0"`
    Category    string  `json:"category" validate:"required"`
}

func main() {
    app := fiber.New(fiber.Config{
        BodyLimit: 5 * 1024 * 1024, // 5MB
    })

    // JSON বডি
    app.Post("/api/products", func(c fiber.Ctx) error {
        var product Product
        
        if err := c.Bind().JSON(&product); err != nil {
            return c.Status(400).JSON(fiber.Map{
                "error": "অবৈধ রিকোয়েস্ট বডি",
            })
        }
        
        if err := validate.Struct(product); err != nil {
            return c.Status(400).JSON(fiber.Map{
                "error": "ভ্যালিডেশন ব্যর্থ",
            })
        }
        
        return c.Status(201).JSON(fiber.Map{
            "message": "পণ্য তৈরি হয়েছে",
            "product": product,
        })
    })

    // ফর্ম ডেটা
    app.Post("/api/feedback", func(c fiber.Ctx) error {
        name := c.FormValue("name")
        email := c.FormValue("email")
        message := c.FormValue("message")
        
        return c.JSON(fiber.Map{
            "message": "ফিডব্যাক গ্রহণ করা হয়েছে",
            "from":    name,
        })
    })

    // ফাইল আপলোড
    app.Post("/api/upload", func(c fiber.Ctx) error {
        file, err := c.FormFile("file")
        if err != nil {
            return c.Status(400).JSON(fiber.Map{
                "error": "ফাইল প্রয়োজন",
            })
        }
        
        // ফাইল টাইপ চেক
        contentType := file.Header.Get("Content-Type")
        if contentType != "image/jpeg" && contentType != "image/png" {
            return c.Status(400).JSON(fiber.Map{
                "error": "শুধু JPEG এবং PNG অনুমোদিত",
            })
        }
        
        dst := "./uploads/" + file.Filename
        c.SaveFile(file, dst)
        
        return c.JSON(fiber.Map{
            "message":  "আপলোড সফল",
            "filename": file.Filename,
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

---

[← আগের: কনটেক্সট বেসিক](01-context-basics.md) | [পরবর্তী: রেসপন্স মেথড →](03-response-methods.md)
