# কনকারেন্সি বেসিক (Concurrency Basics)

## কনকারেন্সি vs প্যারালেলিজম

```
কনকারেন্সি (Concurrency):
- একসাথে অনেক কাজ ম্যানেজ করা
- একটি CPU-তেও সম্ভব
- Go-এর ডিফল্ট মডেল

প্যারালেলিজম (Parallelism):
- একসাথে অনেক কাজ সম্পাদন করা
- একাধিক CPU প্রয়োজন
- GOMAXPROCS দিয়ে নিয়ন্ত্রণ
```

```go
import "runtime"

func main() {
    // CPU কোর সংখ্যা দেখুন
    fmt.Println("CPUs:", runtime.NumCPU())
    
    // সক্রিয় গোরুটিন সংখ্যা
    fmt.Println("Goroutines:", runtime.NumGoroutine())
    
    // সর্বোচ্চ প্রসেসর সেট করুন
    runtime.GOMAXPROCS(4) // ৪টি CPU ব্যবহার করুন
}
```

## Go-এর কনকারেন্সি মডেল: CSP

CSP (Communicating Sequential Processes):
- "শেয়ার্ড মেমরি দিয়ে কমিউনিকেশন নয়, কমিউনিকেশন দিয়ে মেমরি শেয়ার করুন"
- চ্যানেল দিয়ে গোরুটিনগুলো কথা বলে

```go
// ❌ ভুল - শেয়ার্ড মেমরি (Race Condition)
var counter int
go func() { counter++ }()
go func() { counter++ }()

// ✅ সঠিক - চ্যানেল দিয়ে কমিউনিকেশন
counter := 0
ch := make(chan int)
go func() { ch <- 1 }()
go func() { ch <- 1 }()
counter += <-ch
counter += <-ch
```

## Race Condition

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    counter := 0
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter++ // Race Condition!
        }()
    }
    
    wg.Wait()
    fmt.Println("Counter:", counter) // ১০০০ নাও হতে পারে!
}
```

### Race Detector

```bash
# রেস কন্ডিশন সনাক্ত করুন
go run -race main.go
go test -race ./...
```

## Atomic Operations

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    var counter int64
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            atomic.AddInt64(&counter, 1) // নিরাপদ!
        }()
    }
    
    wg.Wait()
    fmt.Println("Counter:", counter) // সবসময় ১০০০
}
```

### Atomic অপারেশনস

```go
import "sync/atomic"

var counter int64

// যোগ
atomic.AddInt64(&counter, 1)

// বিয়োগ
atomic.AddInt64(&counter, -1)

// পড়ুন
value := atomic.LoadInt64(&counter)

// লিখুন
atomic.StoreInt64(&counter, 100)

// Compare and Swap
atomic.CompareAndSwapInt64(&counter, 100, 200)

// Swap
old := atomic.SwapInt64(&counter, 0)
```

## Fiber-এ Atomic Counter

```go
package main

import (
    "sync/atomic"
    "github.com/gofiber/fiber/v3"
)

var requestCount int64

func main() {
    app := fiber.New()

    app.Use(func(c fiber.Ctx) error {
        atomic.AddInt64(&requestCount, 1)
        return c.Next()
    })

    app.Get("/stats", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "total_requests": atomic.LoadInt64(&requestCount),
        })
    })

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello!")
    })

    app.Listen(":3000")
}
```

## Sync Package

### sync.Once

```go
package main

import (
    "fmt"
    "sync"
)

var once sync.Once
var config map[string]string

func loadConfig() {
    fmt.Println("কনফিগ লোড হচ্ছে...")
    config = map[string]string{
        "db": "localhost:5432",
    }
}

func getConfig() map[string]string {
    once.Do(loadConfig) // শুধু একবার চলবে
    return config
}

func main() {
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            getConfig()
        }()
    }
    
    wg.Wait()
    // আউটপুট: কনফিগ লোড হচ্ছে... (শুধু একবার)
}
```

### sync.Map

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var m sync.Map
    
    // স্টোর
    m.Store("key1", "value1")
    m.Store("key2", 42)
    
    // লোড
    if value, ok := m.Load("key1"); ok {
        fmt.Println("key1:", value)
    }
    
    // লোড বা স্টোর
    actual, loaded := m.LoadOrStore("key3", "default")
    fmt.Printf("key3: %v (loaded: %v)\n", actual, loaded)
    
    // ডিলিট
    m.Delete("key1")
    
    // Range
    m.Range(func(key, value interface{}) bool {
        fmt.Printf("%v: %v\n", key, value)
        return true // true = চালিয়ে যান
    })
}
```

## Fiber-এ Session Store

```go
package main

import (
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/google/uuid"
)

type Session struct {
    ID        string
    UserID    int
    Data      map[string]interface{}
    CreatedAt time.Time
}

var sessions sync.Map

func main() {
    app := fiber.New()

    // সেশন তৈরি
    app.Post("/login", func(c fiber.Ctx) error {
        sessionID := uuid.New().String()
        
        session := &Session{
            ID:        sessionID,
            UserID:    1,
            Data:      make(map[string]interface{}),
            CreatedAt: time.Now(),
        }
        
        sessions.Store(sessionID, session)
        
        c.Cookie(&fiber.Cookie{
            Name:  "session_id",
            Value: sessionID,
        })
        
        return c.JSON(fiber.Map{"message": "লগইন সফল"})
    })

    // সেশন চেক মিডলওয়্যার
    authMiddleware := func(c fiber.Ctx) error {
        sessionID := c.Cookies("session_id")
        if sessionID == "" {
            return c.Status(401).JSON(fiber.Map{"error": "Unauthorized"})
        }
        
        if session, ok := sessions.Load(sessionID); ok {
            c.Locals("session", session)
            return c.Next()
        }
        
        return c.Status(401).JSON(fiber.Map{"error": "Invalid session"})
    }

    app.Get("/profile", authMiddleware, func(c fiber.Ctx) error {
        session := c.Locals("session").(*Session)
        return c.JSON(fiber.Map{
            "user_id":    session.UserID,
            "created_at": session.CreatedAt,
        })
    })

    app.Listen(":3000")
}
```

## sync.Pool

```go
package main

import (
    "bytes"
    "sync"
)

var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func processData(data string) string {
    // Pool থেকে বাফার নিন
    buf := bufferPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufferPool.Put(buf) // ফেরত দিন
    }()
    
    buf.WriteString("Processed: ")
    buf.WriteString(data)
    
    return buf.String()
}
```

## Fiber-এ Buffer Pool

```go
package main

import (
    "bytes"
    "sync"
    "github.com/gofiber/fiber/v3"
)

var jsonBufferPool = sync.Pool{
    New: func() interface{} {
        return bytes.NewBuffer(make([]byte, 0, 1024))
    },
}

func main() {
    app := fiber.New()

    app.Get("/data", func(c fiber.Ctx) error {
        buf := jsonBufferPool.Get().(*bytes.Buffer)
        defer func() {
            buf.Reset()
            jsonBufferPool.Put(buf)
        }()
        
        // JSON তৈরি করুন
        buf.WriteString(`{"message":"Hello","timestamp":`)
        buf.WriteString(`"2024-01-01"}`)
        
        c.Set("Content-Type", "application/json")
        return c.Send(buf.Bytes())
    })

    app.Listen(":3000")
}
```

## Concurrency Pattern Summary

```go
// ১. Goroutine - হালকা থ্রেড
go func() { ... }()

// ২. Channel - কমিউনিকেশন
ch := make(chan T)

// ৩. WaitGroup - সিঙ্ক্রোনাইজেশন
var wg sync.WaitGroup

// ৪. Mutex - এক্সক্লুসিভ অ্যাক্সেস
var mu sync.Mutex

// ৫. Atomic - লক-ফ্রি অপারেশন
atomic.AddInt64(&counter, 1)

// ৬. Once - একবার ইনিশিয়ালাইজেশন
once.Do(func)

// ৭. Pool - অবজেক্ট রিইউজ
pool.Get() / pool.Put()

// ৮. Map - থ্রেড-সেফ ম্যাপ
sync.Map
```

## কনকারেন্সি চেকলিস্ট

- [ ] Race condition চেক করুন (`-race`)
- [ ] শেয়ার্ড ডেটায় Mutex/Atomic ব্যবহার করুন
- [ ] চ্যানেল বন্ধ করতে ভুলবেন না
- [ ] WaitGroup দিয়ে গোরুটিন ট্র্যাক করুন
- [ ] Context দিয়ে বাতিল করার ব্যবস্থা রাখুন
- [ ] sync.Pool দিয়ে মেমরি অপটিমাইজ করুন

---

[← আগের: সিলেক্ট স্টেটমেন্ট](../06-channels/03-select-statement.md) | [পরবর্তী: Mutex এবং Sync →](02-mutex-sync.md)
