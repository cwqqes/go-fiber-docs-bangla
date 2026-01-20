# Fiber-এ গোরুটিন

## Fiber Context এবং গোরুটিন

Fiber-এ গোরুটিন ব্যবহার করার সময় সতর্ক থাকতে হবে, কারণ `fiber.Ctx` রিকোয়েস্ট শেষ হলে পুনঃব্যবহৃত হয়।

## সমস্যা: Context পুনঃব্যবহার

```go
// ভুল - বিপজ্জনক!
app.Get("/", func(c fiber.Ctx) error {
    go func() {
        // c এখানে অন্য রিকোয়েস্টের জন্য ব্যবহৃত হতে পারে
        fmt.Println(c.Path()) // বিপজ্জনক!
    }()
    
    return c.SendString("OK")
})
```

## সঠিক উপায়: ডেটা কপি করুন

```go
app.Get("/", func(c fiber.Ctx) error {
    // প্রয়োজনীয় ডেটা কপি করুন
    path := c.Path()
    userId := c.Query("user_id")
    
    go func() {
        // কপি করা ডেটা ব্যবহার করুন
        fmt.Println("Path:", path)
        fmt.Println("User ID:", userId)
    }()
    
    return c.SendString("OK")
})
```

## UserContext ব্যবহার করুন

Fiber-এর `UserContext()` মেথড গোরুটিনে ব্যবহার করতে পারেন:

```go
app.Get("/", func(c fiber.Ctx) error {
    // UserContext পান
    ctx := c.UserContext()
    
    go func() {
        // ctx ব্যবহার করুন (cancellation এর জন্য)
        select {
        case <-ctx.Done():
            fmt.Println("Context বাতিল হয়েছে")
            return
        default:
            // কাজ করুন
        }
    }()
    
    return c.SendString("OK")
})
```

## ব্যাকগ্রাউন্ড টাস্ক

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Post("/send-email", func(c fiber.Ctx) error {
        // রিকোয়েস্ট ডেটা পান
        type EmailRequest struct {
            To      string `json:"to"`
            Subject string `json:"subject"`
            Body    string `json:"body"`
        }
        
        var req EmailRequest
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{
                "error": "অবৈধ রিকোয়েস্ট",
            })
        }
        
        // ব্যাকগ্রাউন্ডে ইমেইল পাঠান
        go func(email EmailRequest) {
            // ইমেইল পাঠানোর লজিক
            time.Sleep(2 * time.Second) // সিমুলেশন
            log.Printf("ইমেইল পাঠানো হয়েছে: %s", email.To)
        }(req)
        
        // তাৎক্ষণিক রেসপন্স
        return c.JSON(fiber.Map{
            "message": "ইমেইল পাঠানো হচ্ছে",
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

## Worker Pool প্যাটার্ন

```go
package main

import (
    "log"
    "sync"
    "github.com/gofiber/fiber/v3"
)

type Job struct {
    ID   int
    Data string
}

type Result struct {
    JobID   int
    Success bool
    Error   error
}

var (
    jobQueue    = make(chan Job, 100)
    resultQueue = make(chan Result, 100)
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for job := range jobQueue {
        log.Printf("Worker %d processing job %d", id, job.ID)
        
        // কাজ প্রসেস করুন
        // ...
        
        resultQueue <- Result{
            JobID:   job.ID,
            Success: true,
        }
    }
}

func main() {
    // Worker pool শুরু করুন
    var wg sync.WaitGroup
    workerCount := 5
    
    for i := 1; i <= workerCount; i++ {
        wg.Add(1)
        go worker(i, &wg)
    }

    app := fiber.New()

    app.Post("/job", func(c fiber.Ctx) error {
        job := Job{
            ID:   int(time.Now().UnixNano()),
            Data: c.FormValue("data"),
        }
        
        select {
        case jobQueue <- job:
            return c.JSON(fiber.Map{
                "message": "জব সাবমিট হয়েছে",
                "jobId":   job.ID,
            })
        default:
            return c.Status(503).JSON(fiber.Map{
                "error": "সার্ভার ব্যস্ত",
            })
        }
    })

    log.Fatal(app.Listen(":3000"))
}
```

## Async/Await প্যাটার্ন

```go
package main

import (
    "context"
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
)

// Future টাইপ
type Future[T any] struct {
    result chan T
    err    chan error
}

func NewFuture[T any](fn func() (T, error)) *Future[T] {
    f := &Future[T]{
        result: make(chan T, 1),
        err:    make(chan error, 1),
    }
    
    go func() {
        result, err := fn()
        if err != nil {
            f.err <- err
        } else {
            f.result <- result
        }
    }()
    
    return f
}

func (f *Future[T]) Await() (T, error) {
    select {
    case result := <-f.result:
        return result, nil
    case err := <-f.err:
        var zero T
        return zero, err
    }
}

func main() {
    app := fiber.New()

    app.Get("/data", func(c fiber.Ctx) error {
        // প্যারালেল ফেচ
        userFuture := NewFuture(func() (map[string]interface{}, error) {
            time.Sleep(100 * time.Millisecond)
            return map[string]interface{}{"name": "রহিম"}, nil
        })
        
        ordersFuture := NewFuture(func() ([]string, error) {
            time.Sleep(150 * time.Millisecond)
            return []string{"order1", "order2"}, nil
        })
        
        // রেজাল্ট পান
        user, err1 := userFuture.Await()
        orders, err2 := ordersFuture.Await()
        
        if err1 != nil || err2 != nil {
            return c.Status(500).JSON(fiber.Map{"error": "ডেটা ফেচ ব্যর্থ"})
        }
        
        return c.JSON(fiber.Map{
            "user":   user,
            "orders": orders,
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

## টাইমআউট সহ গোরুটিন

```go
app.Get("/slow", func(c fiber.Ctx) error {
    ctx, cancel := context.WithTimeout(c.UserContext(), 3*time.Second)
    defer cancel()
    
    resultChan := make(chan string, 1)
    errChan := make(chan error, 1)
    
    go func() {
        // সময় সাপেক্ষ কাজ
        result, err := doSlowWork()
        if err != nil {
            errChan <- err
            return
        }
        resultChan <- result
    }()
    
    select {
    case result := <-resultChan:
        return c.JSON(fiber.Map{"result": result})
    case err := <-errChan:
        return c.Status(500).JSON(fiber.Map{"error": err.Error()})
    case <-ctx.Done():
        return c.Status(408).JSON(fiber.Map{"error": "টাইমআউট"})
    }
})
```

## Graceful Shutdown

```go
package main

import (
    "context"
    "log"
    "os"
    "os/signal"
    "sync"
    "syscall"
    "time"
    
    "github.com/gofiber/fiber/v3"
)

var (
    activeJobs sync.WaitGroup
)

func main() {
    app := fiber.New()

    app.Post("/job", func(c fiber.Ctx) error {
        activeJobs.Add(1)
        
        go func() {
            defer activeJobs.Done()
            
            // কাজ করুন
            time.Sleep(5 * time.Second)
            log.Println("জব সম্পন্ন")
        }()
        
        return c.JSON(fiber.Map{"status": "started"})
    })

    // Graceful shutdown
    go func() {
        sigChan := make(chan os.Signal, 1)
        signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
        
        <-sigChan
        log.Println("Shutting down...")
        
        // নতুন রিকোয়েস্ট বন্ধ করুন
        app.Shutdown()
        
        // চলমান জব শেষ হওয়ার অপেক্ষা করুন
        log.Println("চলমান জব শেষ হওয়ার অপেক্ষায়...")
        activeJobs.Wait()
        
        log.Println("সব জব শেষ, বন্ধ হচ্ছে")
        os.Exit(0)
    }()

    log.Fatal(app.Listen(":3000"))
}
```

## সতর্কতা সারসংক্ষেপ

| করণীয় | বর্জনীয় |
|--------|---------|
| ডেটা কপি করুন | সরাসরি `c` পাস করবেন না |
| `UserContext()` ব্যবহার করুন | Context মডিফাই করবেন না |
| WaitGroup ব্যবহার করুন | গোরুটিন লিক করবেন না |
| এরর হ্যান্ডল করুন | প্যানিক ইগনোর করবেন না |

---

[← আগের: গোরুটিন বেসিক](01-goroutine-basics.md) | [পরবর্তী: সেফ প্র্যাকটিস →](03-safe-practices.md)
