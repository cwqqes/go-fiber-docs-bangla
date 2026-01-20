# সিলেক্ট স্টেটমেন্ট (Select Statement)

## সিলেক্ট কি?

`select` হলো Go-এর একটি কন্ট্রোল স্ট্রাকচার যা একাধিক চ্যানেল অপারেশনের মধ্যে অপেক্ষা করতে দেয়।

```go
select {
case msg := <-ch1:
    fmt.Println("ch1 থেকে:", msg)
case msg := <-ch2:
    fmt.Println("ch2 থেকে:", msg)
}
```

## বেসিক সিলেক্ট

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    go func() {
        time.Sleep(100 * time.Millisecond)
        ch1 <- "এক"
    }()
    
    go func() {
        time.Sleep(200 * time.Millisecond)
        ch2 <- "দুই"
    }()
    
    // যেটি আগে রেডি হয় সেটি গ্রহণ করে
    select {
    case msg := <-ch1:
        fmt.Println("প্রাপ্ত:", msg)
    case msg := <-ch2:
        fmt.Println("প্রাপ্ত:", msg)
    }
}
// আউটপুট: প্রাপ্ত: এক
```

## Default Case

```go
package main

import "fmt"

func main() {
    ch := make(chan int)
    
    select {
    case value := <-ch:
        fmt.Println("মান:", value)
    default:
        // কোনো চ্যানেল রেডি না থাকলে
        fmt.Println("কোনো ডেটা নেই")
    }
}
// আউটপুট: কোনো ডেটা নেই
```

## Timeout Pattern

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan string)
    
    go func() {
        time.Sleep(3 * time.Second)
        ch <- "ফলাফল"
    }()
    
    select {
    case result := <-ch:
        fmt.Println("পাওয়া গেছে:", result)
    case <-time.After(2 * time.Second):
        fmt.Println("টাইমআউট!")
    }
}
// আউটপুট: টাইমআউট!
```

## Fiber-এ Timeout

```go
package main

import (
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Get("/api/slow", func(c fiber.Ctx) error {
        resultCh := make(chan string, 1)
        
        go func() {
            result := callSlowExternalAPI()
            resultCh <- result
        }()
        
        select {
        case result := <-resultCh:
            return c.JSON(fiber.Map{"result": result})
            
        case <-time.After(5 * time.Second):
            return c.Status(504).JSON(fiber.Map{
                "error": "Gateway timeout",
            })
        }
    })

    app.Listen(":3000")
}

func callSlowExternalAPI() string {
    time.Sleep(10 * time.Second) // সিমুলেটেড স্লো API
    return "data"
}
```

## Multiple Channel Handling

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    orders := make(chan string)
    payments := make(chan string)
    done := make(chan struct{})
    
    // অর্ডার প্রসেসর
    go func() {
        for i := 1; i <= 3; i++ {
            time.Sleep(100 * time.Millisecond)
            orders <- fmt.Sprintf("Order-%d", i)
        }
    }()
    
    // পেমেন্ট প্রসেসর
    go func() {
        for i := 1; i <= 2; i++ {
            time.Sleep(150 * time.Millisecond)
            payments <- fmt.Sprintf("Payment-%d", i)
        }
    }()
    
    // টাইমার
    go func() {
        time.Sleep(1 * time.Second)
        close(done)
    }()
    
    // ইভেন্ট লুপ
    for {
        select {
        case order := <-orders:
            fmt.Println("নতুন অর্ডার:", order)
            
        case payment := <-payments:
            fmt.Println("নতুন পেমেন্ট:", payment)
            
        case <-done:
            fmt.Println("সময় শেষ!")
            return
        }
    }
}
```

## Non-Blocking Operations

```go
package main

import "fmt"

func main() {
    ch := make(chan int, 1)
    
    // Non-blocking send
    select {
    case ch <- 42:
        fmt.Println("পাঠানো হয়েছে")
    default:
        fmt.Println("চ্যানেল পূর্ণ")
    }
    
    // Non-blocking receive
    select {
    case value := <-ch:
        fmt.Println("প্রাপ্ত:", value)
    default:
        fmt.Println("কোনো মান নেই")
    }
}
```

## Ticker এবং Timer

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ticker := time.NewTicker(500 * time.Millisecond)
    timeout := time.After(3 * time.Second)
    
    for {
        select {
        case <-ticker.C:
            fmt.Println("টিক!")
            
        case <-timeout:
            fmt.Println("শেষ!")
            ticker.Stop()
            return
        }
    }
}
```

## Fiber-এ Server-Sent Events

```go
package main

import (
    "bufio"
    "fmt"
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Get("/events", func(c fiber.Ctx) error {
        c.Set("Content-Type", "text/event-stream")
        c.Set("Cache-Control", "no-cache")
        c.Set("Connection", "keep-alive")
        
        // Response writer
        c.Context().SetBodyStreamWriter(func(w *bufio.Writer) {
            ticker := time.NewTicker(1 * time.Second)
            defer ticker.Stop()
            
            timeout := time.After(30 * time.Second)
            count := 0
            
            for {
                select {
                case <-ticker.C:
                    count++
                    fmt.Fprintf(w, "data: Event #%d at %s\n\n", 
                        count, time.Now().Format(time.RFC3339))
                    w.Flush()
                    
                case <-timeout:
                    fmt.Fprintf(w, "data: Stream ended\n\n")
                    w.Flush()
                    return
                }
            }
        })
        
        return nil
    })

    app.Listen(":3000")
}
```

## Worker Shutdown Pattern

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, jobs <-chan int, done <-chan struct{}, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for {
        select {
        case job, ok := <-jobs:
            if !ok {
                fmt.Printf("Worker %d: jobs চ্যানেল বন্ধ\n", id)
                return
            }
            fmt.Printf("Worker %d: Job %d প্রসেসিং\n", id, job)
            time.Sleep(100 * time.Millisecond)
            
        case <-done:
            fmt.Printf("Worker %d: শাটডাউন সিগন্যাল পাওয়া\n", id)
            return
        }
    }
}

func main() {
    jobs := make(chan int, 10)
    done := make(chan struct{})
    var wg sync.WaitGroup
    
    // ৩টি worker শুরু করুন
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go worker(i, jobs, done, &wg)
    }
    
    // কিছু জব পাঠান
    for i := 1; i <= 5; i++ {
        jobs <- i
    }
    
    // শাটডাউন সিগন্যাল
    time.Sleep(500 * time.Millisecond)
    close(done)
    
    wg.Wait()
    fmt.Println("সব worker বন্ধ")
}
```

## Priority Select

```go
package main

import "fmt"

func main() {
    highPriority := make(chan string, 5)
    lowPriority := make(chan string, 5)
    
    highPriority <- "HIGH-1"
    highPriority <- "HIGH-2"
    lowPriority <- "LOW-1"
    lowPriority <- "LOW-2"
    
    // প্রায়োরিটি চেক
    for i := 0; i < 4; i++ {
        select {
        case msg := <-highPriority:
            fmt.Println("উচ্চ প্রায়োরিটি:", msg)
        default:
            select {
            case msg := <-highPriority:
                fmt.Println("উচ্চ প্রায়োরিটি:", msg)
            case msg := <-lowPriority:
                fmt.Println("নিম্ন প্রায়োরিটি:", msg)
            }
        }
    }
}
```

## Fiber-এ Health Check

```go
package main

import (
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
)

type HealthStatus struct {
    mu       sync.RWMutex
    database bool
    cache    bool
    external bool
}

var health = &HealthStatus{}

func checkHealth(status *HealthStatus) {
    dbCh := make(chan bool)
    cacheCh := make(chan bool)
    extCh := make(chan bool)
    
    go func() { dbCh <- pingDatabase() }()
    go func() { cacheCh <- pingCache() }()
    go func() { extCh <- pingExternal() }()
    
    timeout := time.After(2 * time.Second)
    
    for i := 0; i < 3; i++ {
        select {
        case db := <-dbCh:
            status.mu.Lock()
            status.database = db
            status.mu.Unlock()
            
        case cache := <-cacheCh:
            status.mu.Lock()
            status.cache = cache
            status.mu.Unlock()
            
        case ext := <-extCh:
            status.mu.Lock()
            status.external = ext
            status.mu.Unlock()
            
        case <-timeout:
            return
        }
    }
}

func main() {
    // পর্যায়ক্রমিক হেলথ চেক
    go func() {
        ticker := time.NewTicker(10 * time.Second)
        for range ticker.C {
            checkHealth(health)
        }
    }()

    app := fiber.New()

    app.Get("/health", func(c fiber.Ctx) error {
        health.mu.RLock()
        defer health.mu.RUnlock()
        
        return c.JSON(fiber.Map{
            "database": health.database,
            "cache":    health.cache,
            "external": health.external,
        })
    })

    app.Listen(":3000")
}

func pingDatabase() bool { return true }
func pingCache() bool    { return true }
func pingExternal() bool { return true }
```

## সিলেক্ট বেস্ট প্র্যাকটিসেস

```go
// ✅ সঠিক - টাইমআউট সহ
select {
case result := <-ch:
    process(result)
case <-time.After(5 * time.Second):
    handleTimeout()
}

// ✅ সঠিক - default দিয়ে non-blocking
select {
case ch <- value:
    // পাঠানো হয়েছে
default:
    // বাফার পূর্ণ
}

// ❌ ভুল - অসীম ব্লক
result := <-ch // টাইমআউট নেই

// ❌ ভুল - busy loop
for {
    select {
    case <-ch:
    default:
        // CPU 100% ব্যবহার করবে
    }
}
```

## সারসংক্ষেপ

| প্যাটার্ন | ব্যবহার |
|---------|--------|
| `select { case...: }` | একাধিক চ্যানেল হ্যান্ডলিং |
| `case <-time.After()` | টাইমআউট |
| `default:` | Non-blocking অপারেশন |
| `ticker.C` | পর্যায়ক্রমিক কাজ |
| `done` চ্যানেল | শাটডাউন সিগন্যাল |

---

[← আগের: বাফার্ড চ্যানেল](02-buffered-channels.md) | [পরবর্তী: কনকারেন্সি বেসিক →](../07-concurrency/01-concurrency-basics.md)
