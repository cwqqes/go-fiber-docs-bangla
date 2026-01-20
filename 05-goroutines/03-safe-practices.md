# গোরুটিন সেফ প্র্যাকটিস

## ১. Graceful Shutdown

সার্ভার বন্ধের সময় গোরুটিনগুলো সঠিকভাবে বন্ধ করা।

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
    shutdownChan = make(chan struct{})
    wg           sync.WaitGroup
)

func main() {
    app := fiber.New()

    // ব্যাকগ্রাউন্ড ওয়ার্কার শুরু করুন
    wg.Add(1)
    go backgroundWorker()

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello!")
    })

    // গ্রেসফুল শাটডাউন হ্যান্ডলিং
    go func() {
        sigChan := make(chan os.Signal, 1)
        signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
        <-sigChan

        log.Println("শাটডাউন সিগন্যাল পাওয়া গেছে...")

        // গোরুটিনগুলোকে বন্ধ হতে বলুন
        close(shutdownChan)

        // গোরুটিনগুলোর জন্য অপেক্ষা করুন
        wg.Wait()

        // সার্ভার বন্ধ করুন
        if err := app.Shutdown(); err != nil {
            log.Printf("শাটডাউন ত্রুটি: %v", err)
        }
    }()

    log.Fatal(app.Listen(":3000"))
}

func backgroundWorker() {
    defer wg.Done()
    
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            log.Println("ব্যাকগ্রাউন্ড কাজ চলছে...")
        case <-shutdownChan:
            log.Println("ওয়ার্কার বন্ধ হচ্ছে...")
            return
        }
    }
}
```

## ২. Resource Cleanup

```go
type ResourceManager struct {
    mu        sync.Mutex
    resources map[string]*Resource
    wg        sync.WaitGroup
    done      chan struct{}
}

func NewResourceManager() *ResourceManager {
    rm := &ResourceManager{
        resources: make(map[string]*Resource),
        done:      make(chan struct{}),
    }
    
    // ক্লিনআপ গোরুটিন
    rm.wg.Add(1)
    go rm.cleanupLoop()
    
    return rm
}

func (rm *ResourceManager) cleanupLoop() {
    defer rm.wg.Done()
    
    ticker := time.NewTicker(5 * time.Minute)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            rm.cleanup()
        case <-rm.done:
            rm.cleanup() // শেষ বার ক্লিনআপ
            return
        }
    }
}

func (rm *ResourceManager) cleanup() {
    rm.mu.Lock()
    defer rm.mu.Unlock()
    
    for id, res := range rm.resources {
        if res.IsExpired() {
            res.Close()
            delete(rm.resources, id)
        }
    }
}

func (rm *ResourceManager) Close() {
    close(rm.done)
    rm.wg.Wait()
}
```

## ৩. Goroutine Leak Prevention

```go
// ❌ ভুল - গোরুটিন লিক হতে পারে
func leakyGoroutine() {
    ch := make(chan int)
    go func() {
        result := doWork()
        ch <- result // কেউ না পড়লে আটকে থাকবে
    }()
    
    // ত্রুটি হলে ch পড়া হয় না
    if someError != nil {
        return // গোরুটিন আটকে থাকবে!
    }
    
    <-ch
}

// ✅ সঠিক - বাফার্ড চ্যানেল ব্যবহার করুন
func safeGoroutine() {
    ch := make(chan int, 1) // বাফার্ড
    go func() {
        result := doWork()
        ch <- result // ব্লক হবে না
    }()
    
    if someError != nil {
        return // গোরুটিন নিজে শেষ হয়ে যাবে
    }
    
    <-ch
}

// ✅ আরও ভালো - Context দিয়ে বাতিল করুন
func contextAwareGoroutine(ctx context.Context) {
    go func() {
        select {
        case <-ctx.Done():
            return // বাতিল হলে বের হন
        default:
            doWork()
        }
    }()
}
```

## ৪. Rate Limiting

```go
package main

import (
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
)

type RateLimiter struct {
    mu       sync.Mutex
    requests map[string][]time.Time
    limit    int
    window   time.Duration
}

func NewRateLimiter(limit int, window time.Duration) *RateLimiter {
    rl := &RateLimiter{
        requests: make(map[string][]time.Time),
        limit:    limit,
        window:   window,
    }
    
    // ক্লিনআপ গোরুটিন
    go func() {
        ticker := time.NewTicker(time.Minute)
        for range ticker.C {
            rl.cleanup()
        }
    }()
    
    return rl
}

func (rl *RateLimiter) Allow(key string) bool {
    rl.mu.Lock()
    defer rl.mu.Unlock()
    
    now := time.Now()
    cutoff := now.Add(-rl.window)
    
    // পুরাতন রিকোয়েস্ট ফিল্টার করুন
    valid := make([]time.Time, 0)
    for _, t := range rl.requests[key] {
        if t.After(cutoff) {
            valid = append(valid, t)
        }
    }
    
    if len(valid) >= rl.limit {
        return false
    }
    
    rl.requests[key] = append(valid, now)
    return true
}

func (rl *RateLimiter) cleanup() {
    rl.mu.Lock()
    defer rl.mu.Unlock()
    
    cutoff := time.Now().Add(-rl.window)
    for key, times := range rl.requests {
        valid := make([]time.Time, 0)
        for _, t := range times {
            if t.After(cutoff) {
                valid = append(valid, t)
            }
        }
        if len(valid) == 0 {
            delete(rl.requests, key)
        } else {
            rl.requests[key] = valid
        }
    }
}

func main() {
    limiter := NewRateLimiter(100, time.Minute)
    
    app := fiber.New()
    
    app.Use(func(c fiber.Ctx) error {
        ip := c.IP()
        if !limiter.Allow(ip) {
            return c.Status(429).JSON(fiber.Map{
                "error": "অনেক বেশি রিকোয়েস্ট",
            })
        }
        return c.Next()
    })
    
    app.Listen(":3000")
}
```

## ৫. Panic Recovery

```go
package main

import (
    "log"
    "runtime/debug"
)

func safeGo(fn func()) {
    go func() {
        defer func() {
            if r := recover(); r != nil {
                log.Printf("Panic recovered: %v\n%s", r, debug.Stack())
            }
        }()
        fn()
    }()
}

// ব্যবহার
func main() {
    safeGo(func() {
        // এই ফাংশন প্যানিক করলেও সার্ভার ক্র্যাশ করবে না
        riskyOperation()
    })
}
```

## ৬. Semaphore Pattern

```go
package main

import (
    "context"
    "golang.org/x/sync/semaphore"
)

func main() {
    // সর্বোচ্চ ১০টি কনকারেন্ট অপারেশন
    sem := semaphore.NewWeighted(10)
    
    app := fiber.New()
    
    app.Post("/heavy-task", func(c fiber.Ctx) error {
        ctx := context.Background()
        
        // সেমাফোর অ্যাকয়ার করুন
        if err := sem.Acquire(ctx, 1); err != nil {
            return c.Status(503).JSON(fiber.Map{
                "error": "সার্ভার ব্যস্ত",
            })
        }
        defer sem.Release(1)
        
        // ভারী কাজ করুন
        result := doHeavyWork()
        
        return c.JSON(fiber.Map{"result": result})
    })
    
    app.Listen(":3000")
}
```

## ৭. Circuit Breaker

```go
package main

import (
    "errors"
    "sync"
    "time"
)

type CircuitBreaker struct {
    mu           sync.Mutex
    failures     int
    successes    int
    state        string // closed, open, half-open
    threshold    int
    timeout      time.Duration
    lastFailure  time.Time
}

func NewCircuitBreaker(threshold int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        state:     "closed",
        threshold: threshold,
        timeout:   timeout,
    }
}

func (cb *CircuitBreaker) Execute(fn func() error) error {
    cb.mu.Lock()
    
    // Open state - সার্কিট খোলা
    if cb.state == "open" {
        if time.Since(cb.lastFailure) > cb.timeout {
            cb.state = "half-open"
        } else {
            cb.mu.Unlock()
            return errors.New("circuit breaker is open")
        }
    }
    cb.mu.Unlock()
    
    // ফাংশন চালান
    err := fn()
    
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    if err != nil {
        cb.failures++
        cb.lastFailure = time.Now()
        
        if cb.failures >= cb.threshold {
            cb.state = "open"
        }
        return err
    }
    
    // সফল
    if cb.state == "half-open" {
        cb.successes++
        if cb.successes >= 3 {
            cb.state = "closed"
            cb.failures = 0
            cb.successes = 0
        }
    }
    
    return nil
}

// ব্যবহার
var externalAPIBreaker = NewCircuitBreaker(5, 30*time.Second)

app.Get("/external", func(c fiber.Ctx) error {
    var result string
    
    err := externalAPIBreaker.Execute(func() error {
        var err error
        result, err = callExternalAPI()
        return err
    })
    
    if err != nil {
        return c.Status(503).JSON(fiber.Map{
            "error": "সার্ভিস অস্থায়ীভাবে অনুপলব্ধ",
        })
    }
    
    return c.JSON(fiber.Map{"result": result})
})
```

## ৮. Best Practices Summary

```go
// ✅ সঠিক প্যাটার্ন
type SafeApp struct {
    wg      sync.WaitGroup
    done    chan struct{}
    app     *fiber.App
}

func NewSafeApp() *SafeApp {
    return &SafeApp{
        done: make(chan struct{}),
        app:  fiber.New(),
    }
}

func (s *SafeApp) StartBackground(fn func(done <-chan struct{})) {
    s.wg.Add(1)
    go func() {
        defer s.wg.Done()
        fn(s.done)
    }()
}

func (s *SafeApp) Shutdown() {
    close(s.done)    // সব গোরুটিনকে বন্ধ হতে বলুন
    s.wg.Wait()      // অপেক্ষা করুন
    s.app.Shutdown() // সার্ভার বন্ধ করুন
}
```

## চেকলিস্ট

- [ ] সব গোরুটিনে `defer` এবং `recover` ব্যবহার করুন
- [ ] Context দিয়ে বাতিল করার ব্যবস্থা রাখুন
- [ ] `sync.WaitGroup` দিয়ে গোরুটিন ট্র্যাক করুন
- [ ] Graceful shutdown বাস্তবায়ন করুন
- [ ] Resource cleanup নিশ্চিত করুন
- [ ] Rate limiting প্রয়োগ করুন
- [ ] Goroutine leak পরীক্ষা করুন

---

[← আগের: Fiber-এ গোরুটিন](02-fiber-goroutines.md) | [পরবর্তী: চ্যানেল বেসিক →](../06-channels/01-channel-basics.md)
