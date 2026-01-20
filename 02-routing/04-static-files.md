# স্ট্যাটিক ফাইল সার্ভিং

## স্ট্যাটিক ফাইল কী?

স্ট্যাটিক ফাইল হলো এমন ফাইল যা সার্ভার সরাসরি ক্লায়েন্টকে পাঠায় - যেমন HTML, CSS, JavaScript, ছবি, ভিডিও ইত্যাদি।

## বেসিক স্ট্যাটিক সার্ভিং

```go
package main

import (
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/static"
)

func main() {
    app := fiber.New()

    // ./public ফোল্ডার থেকে ফাইল সার্ভ করুন
    app.Get("/*", static.New("./public"))

    app.Listen(":3000")
}
```

ফোল্ডার স্ট্রাকচার:
```
project/
├── main.go
└── public/
    ├── index.html
    ├── css/
    │   └── style.css
    ├── js/
    │   └── app.js
    └── images/
        └── logo.png
```

## Use দিয়ে স্ট্যাটিক সার্ভিং

```go
// রুট পাথে সার্ভ
app.Use("/", static.New("./public"))

// নির্দিষ্ট প্রিফিক্সে সার্ভ
app.Use("/static", static.New("./public"))
// http://localhost:3000/static/css/style.css

// একাধিক ফোল্ডার
app.Use("/uploads", static.New("./uploads"))
app.Use("/assets", static.New("./assets"))
```

## কনফিগারেশন অপশন

```go
app.Use("/static", static.New("./public", static.Config{
    // ইনডেক্স ফাইল
    Index: "index.html",
    
    // ফাইল না পেলে ব্রাউজ করতে দিন
    Browse: true,
    
    // গজিপ কম্প্রেশন
    Compress: true,
    
    // ক্যাশ কন্ট্রোল (সেকেন্ডে)
    MaxAge: 3600, // 1 ঘন্টা
    
    // ডট (.) দিয়ে শুরু ফাইল সার্ভ করুন
    // যেমন .htaccess, .env
    // সাবধান: সিকিউরিটি ইস্যু হতে পারে
}))
```

## সিঙ্গেল ফাইল সার্ভ

```go
// একটি নির্দিষ্ট ফাইল সার্ভ করুন
app.Use("/static", static.New("./public/hello.html"))

// http://localhost:3000/static → hello.html
// http://localhost:3000/static/anything → hello.html
```

## Embed করা ফাইল সার্ভ

Go 1.16+ এ `embed` প্যাকেজ ব্যবহার করে:

```go
package main

import (
    "embed"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/static"
)

//go:embed public/*
var publicFS embed.FS

func main() {
    app := fiber.New()

    // এমবেডেড ফাইল সার্ভ করুন
    app.Use("/", static.New("", static.Config{
        FS: publicFS,
        PathPrefix: "public",
    }))

    app.Listen(":3000")
}
```

## SPA (Single Page Application) সার্ভিং

React, Vue, Angular ইত্যাদির জন্য:

```go
package main

import (
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/static"
)

func main() {
    app := fiber.New()

    // API রাউট
    api := app.Group("/api")
    api.Get("/users", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{"users": []string{}})
    })

    // স্ট্যাটিক ফাইল সার্ভ (CSS, JS, ইমেজ)
    app.Use("/", static.New("./dist"))

    // SPA ফলব্যাক - সব রাউটে index.html
    app.Get("/*", func(c fiber.Ctx) error {
        return c.SendFile("./dist/index.html")
    })

    app.Listen(":3000")
}
```

## ফাইল ডাউনলোড

```go
app.Get("/download/:filename", func(c fiber.Ctx) error {
    filename := c.Params("filename")
    filepath := "./downloads/" + filename
    
    // ফাইল ডাউনলোড হিসেবে পাঠান
    return c.Download(filepath, filename)
})

// কাস্টম ডাউনলোড নাম
app.Get("/export/report", func(c fiber.Ctx) error {
    return c.Download("./data/report.pdf", "monthly-report.pdf")
})
```

## ফাইল পাঠানো (ডাউনলোড ছাড়া)

```go
app.Get("/image/:name", func(c fiber.Ctx) error {
    name := c.Params("name")
    return c.SendFile("./images/" + name)
})

// Content-Type সেট করে
app.Get("/pdf", func(c fiber.Ctx) error {
    c.Set("Content-Type", "application/pdf")
    return c.SendFile("./documents/manual.pdf")
})
```

## ফাইল আপলোড

```go
app.Post("/upload", func(c fiber.Ctx) error {
    // সিঙ্গেল ফাইল
    file, err := c.FormFile("file")
    if err != nil {
        return c.Status(400).JSON(fiber.Map{
            "error": "ফাইল পাওয়া যায়নি",
        })
    }

    // ফাইল সেভ করুন
    destination := "./uploads/" + file.Filename
    if err := c.SaveFile(file, destination); err != nil {
        return c.Status(500).JSON(fiber.Map{
            "error": "ফাইল সেভ ব্যর্থ",
        })
    }

    return c.JSON(fiber.Map{
        "message":  "ফাইল আপলোড সফল",
        "filename": file.Filename,
        "size":     file.Size,
    })
})
```

## মাল্টিপল ফাইল আপলোড

```go
app.Post("/upload-multiple", func(c fiber.Ctx) error {
    // ফর্ম পার্স করুন
    form, err := c.MultipartForm()
    if err != nil {
        return c.Status(400).SendString("ফর্ম পার্স ব্যর্থ")
    }

    files := form.File["files"]
    
    var uploadedFiles []string
    for _, file := range files {
        destination := "./uploads/" + file.Filename
        if err := c.SaveFile(file, destination); err != nil {
            continue
        }
        uploadedFiles = append(uploadedFiles, file.Filename)
    }

    return c.JSON(fiber.Map{
        "message": "আপলোড সম্পন্ন",
        "files":   uploadedFiles,
        "count":   len(uploadedFiles),
    })
})
```

## ক্যাশিং হেডার

```go
app.Use("/static", static.New("./public", static.Config{
    MaxAge: 86400, // 24 ঘন্টা
}))

// ম্যানুয়াল ক্যাশ কন্ট্রোল
app.Get("/assets/*", func(c fiber.Ctx) error {
    c.Set("Cache-Control", "public, max-age=31536000") // 1 বছর
    return c.Next()
}, static.New("./assets"))
```

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "log"
    "path/filepath"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/static"
)

func main() {
    app := fiber.New(fiber.Config{
        BodyLimit: 50 * 1024 * 1024, // 50MB লিমিট
    })

    // স্ট্যাটিক ফাইল
    app.Use("/static", static.New("./public", static.Config{
        Compress: true,
        MaxAge:   3600,
    }))

    // ইমেজ সার্ভ
    app.Get("/images/:name", func(c fiber.Ctx) error {
        name := c.Params("name")
        ext := filepath.Ext(name)
        
        // অনুমোদিত এক্সটেনশন চেক
        allowed := map[string]bool{".jpg": true, ".png": true, ".gif": true}
        if !allowed[ext] {
            return c.Status(400).SendString("অননুমোদিত ফাইল টাইপ")
        }
        
        return c.SendFile("./uploads/images/" + name)
    })

    // ফাইল আপলোড
    app.Post("/upload", func(c fiber.Ctx) error {
        file, err := c.FormFile("file")
        if err != nil {
            return c.Status(400).JSON(fiber.Map{"error": err.Error()})
        }

        // সাইজ চেক (10MB)
        if file.Size > 10*1024*1024 {
            return c.Status(400).JSON(fiber.Map{
                "error": "ফাইল অনেক বড় (সর্বোচ্চ 10MB)",
            })
        }

        destination := "./uploads/" + file.Filename
        c.SaveFile(file, destination)

        return c.JSON(fiber.Map{
            "message":  "আপলোড সফল",
            "filename": file.Filename,
        })
    })

    // ফাইল ডাউনলোড
    app.Get("/download/:filename", func(c fiber.Ctx) error {
        filename := c.Params("filename")
        filepath := "./uploads/" + filename
        return c.Download(filepath)
    })

    log.Fatal(app.Listen(":3000"))
}
```

## HTML ফর্ম উদাহরণ

```html
<!-- সিঙ্গেল ফাইল আপলোড -->
<form action="/upload" method="POST" enctype="multipart/form-data">
    <input type="file" name="file" />
    <button type="submit">আপলোড</button>
</form>

<!-- মাল্টিপল ফাইল আপলোড -->
<form action="/upload-multiple" method="POST" enctype="multipart/form-data">
    <input type="file" name="files" multiple />
    <button type="submit">আপলোড</button>
</form>
```

## টেস্ট করুন

```bash
# স্ট্যাটিক ফাইল
curl http://localhost:3000/static/css/style.css
curl http://localhost:3000/static/index.html

# ফাইল আপলোড
curl -F "file=@photo.jpg" http://localhost:3000/upload

# ফাইল ডাউনলোড
curl -O http://localhost:3000/download/photo.jpg
```

---

[← আগের: রাউট গ্রুপিং](03-route-groups.md) | [পরবর্তী: মিডলওয়্যার বেসিক →](../03-middleware/01-middleware-basics.md)
