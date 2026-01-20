# বাফার্ড চ্যানেল (Buffered Channels)

## বাফার্ড চ্যানেল কি?

বাফার্ড চ্যানেল হলো একটি নির্দিষ্ট ক্যাপাসিটি সহ চ্যানেল যা কিছু মান ধরে রাখতে পারে।

```go
// বাফার্ড চ্যানেল তৈরি
ch := make(chan int, 5) // ৫টি মান ধরে রাখতে পারবে
```

## Unbuffered vs Buffered

```go
package main

import "fmt"

func main() {
    // Unbuffered - সাথে সাথে receiver লাগে
    unbuffered := make(chan int)
    
    // Buffered - বাফার পূর্ণ না হওয়া পর্যন্ত ব্লক হয় না
    buffered := make(chan int, 3)
    
    // বাফার্ড চ্যানেলে ব্লক ছাড়া পাঠান
    buffered <- 1
    buffered <- 2
    buffered <- 3
    // buffered <- 4 // এটি ব্লক করবে কারণ বাফার পূর্ণ
    
    fmt.Println(<-buffered) // 1
    fmt.Println(<-buffered) // 2
    fmt.Println(<-buffered) // 3
}
```

## বাফার সাইজ এবং ক্যাপাসিটি

```go
package main

import "fmt"

func main() {
    ch := make(chan string, 5)
    
    ch <- "এক"
    ch <- "দুই"
    ch <- "তিন"
    
    fmt.Println("Length:", len(ch))  // 3 (বর্তমান উপাদান)
    fmt.Println("Capacity:", cap(ch)) // 5 (সর্বোচ্চ ধারণক্ষমতা)
}
```

## বাফার্ড চ্যানেলের সুবিধা

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // ১. ব্লকিং এড়ানো
    jobs := make(chan int, 100)
    
    // দ্রুত ১০০টি জব যোগ করুন
    for i := 1; i <= 100; i++ {
        jobs <- i // ব্লক হবে না
    }
    close(jobs)
    
    // ২. Producer-Consumer ডিকাপলিং
    results := make(chan int, 10)
    
    // প্রোডিউসার দ্রুত উৎপাদন করতে পারে
    go func() {
        for i := 1; i <= 20; i++ {
            results <- i * 2
            fmt.Println("উৎপাদিত:", i*2)
        }
        close(results)
    }()
    
    // কনজিউমার ধীরে প্রসেস করতে পারে
    for result := range results {
        time.Sleep(100 * time.Millisecond)
        fmt.Println("ব্যবহৃত:", result)
    }
}
```

## Fiber-এ Job Queue

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
)

type Job struct {
    ID   int    `json:"id"`
    Data string `json:"data"`
}

var jobQueue = make(chan Job, 100) // ১০০ জব ধারণক্ষমতা

func worker(id int) {
    for job := range jobQueue {
        log.Printf("Worker %d: Job %d প্রসেসিং", id, job.ID)
        time.Sleep(time.Second) // সিমুলেটেড কাজ
        log.Printf("Worker %d: Job %d সম্পন্ন", id, job.ID)
    }
}

func main() {
    // ৫টি worker শুরু করুন
    for i := 1; i <= 5; i++ {
        go worker(i)
    }

    app := fiber.New()

    app.Post("/jobs", func(c fiber.Ctx) error {
        var job Job
        if err := c.Bind().JSON(&job); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid job"})
        }
        
        // Non-blocking job submission
        select {
        case jobQueue <- job:
            return c.Status(202).JSON(fiber.Map{
                "message": "Job queued",
                "id":      job.ID,
            })
        default:
            return c.Status(503).JSON(fiber.Map{
                "error": "Queue full, try again later",
            })
        }
    })

    app.Get("/queue/status", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "queued":   len(jobQueue),
            "capacity": cap(jobQueue),
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

## Rate Limiter with Buffer

```go
package main

import (
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    // Token bucket rate limiter
    tokens := make(chan struct{}, 10) // ১০ টোকেন
    
    // টোকেন রিফিল গোরুটিন
    go func() {
        ticker := time.NewTicker(100 * time.Millisecond)
        for range ticker.C {
            select {
            case tokens <- struct{}{}:
                // টোকেন যোগ হয়েছে
            default:
                // বাফার পূর্ণ, কিছু করার নেই
            }
        }
    }()

    app := fiber.New()

    app.Use(func(c fiber.Ctx) error {
        select {
        case <-tokens:
            // টোকেন পাওয়া গেছে
            return c.Next()
        default:
            // টোকেন নেই
            return c.Status(429).JSON(fiber.Map{
                "error": "Rate limit exceeded",
            })
        }
    })

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello!")
    })

    app.Listen(":3000")
}
```

## Semaphore Pattern

```go
package main

import (
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    // সর্বোচ্চ ১০টি concurrent রিকোয়েস্ট
    semaphore := make(chan struct{}, 10)

    app := fiber.New()

    app.Post("/heavy", func(c fiber.Ctx) error {
        // Semaphore acquire
        select {
        case semaphore <- struct{}{}:
            // অনুমতি পাওয়া গেছে
        case <-time.After(5 * time.Second):
            return c.Status(503).JSON(fiber.Map{
                "error": "Server busy",
            })
        }
        defer func() { <-semaphore }() // Semaphore release
        
        // ভারী কাজ
        time.Sleep(2 * time.Second)
        
        return c.JSON(fiber.Map{"status": "done"})
    })

    app.Listen(":3000")
}
```

## Fan-Out/Fan-In Pattern

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    input := make(chan int, 10)
    output := make(chan int, 10)
    
    // ইনপুট প্রস্তুত করুন
    go func() {
        for i := 1; i <= 20; i++ {
            input <- i
        }
        close(input)
    }()
    
    // Fan-Out: একাধিক worker
    var wg sync.WaitGroup
    numWorkers := 4
    
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for n := range input {
                output <- n * n // স্কয়ার করুন
                fmt.Printf("Worker %d: %d -> %d\n", id, n, n*n)
            }
        }(i)
    }
    
    // Fan-In: রেজাল্ট সংগ্রহ
    go func() {
        wg.Wait()
        close(output)
    }()
    
    // আউটপুট প্রিন্ট করুন
    for result := range output {
        fmt.Println("Result:", result)
    }
}
```

## Fiber-এ Parallel Processing

```go
package main

import (
    "sync"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Post("/process-batch", func(c fiber.Ctx) error {
        type Item struct {
            ID int `json:"id"`
        }
        type Request struct {
            Items []Item `json:"items"`
        }
        
        var req Request
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
        }
        
        // বাফার্ড চ্যানেল দিয়ে প্যারালেল প্রসেসিং
        results := make(chan map[string]interface{}, len(req.Items))
        var wg sync.WaitGroup
        
        for _, item := range req.Items {
            wg.Add(1)
            go func(i Item) {
                defer wg.Done()
                // প্রতিটি আইটেম প্রসেস করুন
                results <- map[string]interface{}{
                    "id":     i.ID,
                    "result": i.ID * 2,
                }
            }(item)
        }
        
        go func() {
            wg.Wait()
            close(results)
        }()
        
        // রেজাল্ট সংগ্রহ করুন
        var allResults []map[string]interface{}
        for r := range results {
            allResults = append(allResults, r)
        }
        
        return c.JSON(fiber.Map{
            "results": allResults,
        })
    })

    app.Listen(":3000")
}
```

## বাফার সাইজ নির্ধারণ

```go
// ছোট বাফার (1-10)
// - কম মেমরি
// - টাইট সিঙ্ক্রোনাইজেশন
ch := make(chan int, 5)

// মাঝারি বাফার (10-100)
// - ভালো থ্রুপুট
// - মধ্যম মেমরি
ch := make(chan int, 50)

// বড় বাফার (100+)
// - উচ্চ থ্রুপুট
// - বেশি মেমরি
// - সতর্কতা: মেমরি লিক হতে পারে
ch := make(chan int, 1000)
```

## সারসংক্ষেপ টেবিল

| বৈশিষ্ট্য | Unbuffered | Buffered |
|---------|-----------|----------|
| ক্যাপাসিটি | 0 | > 0 |
| ব্লকিং (Send) | সবসময় | বাফার পূর্ণ হলে |
| ব্লকিং (Receive) | সবসময় | বাফার খালি হলে |
| সিঙ্ক্রোনাইজেশন | টাইট | লুজ |
| ব্যবহার | সিগন্যালিং | কিউ, ব্যাচ |

---

[← আগের: চ্যানেল বেসিক](01-channel-basics.md) | [পরবর্তী: সিলেক্ট স্টেটমেন্ট →](03-select-statement.md)
