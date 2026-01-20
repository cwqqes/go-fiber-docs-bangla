# স্কেডিউলার বেসিক (Scheduler Basics)

## Go Runtime Scheduler

Go-এর রানটাইম একটি বিল্ট-ইন স্কেডিউলার ব্যবহার করে গোরুটিনগুলো ম্যানেজ করে।

### G-M-P মডেল

```
G (Goroutine) - গোরুটিন
M (Machine)   - OS থ্রেড
P (Processor) - লজিক্যাল প্রসেসর

[G] [G] [G]  <- গোরুটিন Queue
    |
   [P]       <- প্রসেসর
    |
   [M]       <- OS থ্রেড
```

```go
import "runtime"

// প্রসেসর সংখ্যা দেখুন
fmt.Println("GOMAXPROCS:", runtime.GOMAXPROCS(0))

// CPU কোর সংখ্যা
fmt.Println("NumCPU:", runtime.NumCPU())

// সক্রিয় গোরুটিন
fmt.Println("NumGoroutine:", runtime.NumGoroutine())
```

## Task Scheduling Patterns

### সিম্পল টাইমার

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // একবার চালান
    timer := time.NewTimer(2 * time.Second)
    
    <-timer.C
    fmt.Println("টাইমার শেষ!")
}
```

### পর্যায়ক্রমিক টিকার

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ticker := time.NewTicker(1 * time.Second)
    done := make(chan bool)
    
    go func() {
        for {
            select {
            case <-done:
                return
            case t := <-ticker.C:
                fmt.Println("টিক at", t)
            }
        }
    }()
    
    time.Sleep(5 * time.Second)
    ticker.Stop()
    done <- true
    fmt.Println("টিকার বন্ধ")
}
```

## Fiber-এ Scheduled Tasks

```go
package main

import (
    "log"
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
)

type Scheduler struct {
    tasks  map[string]*ScheduledTask
    mu     sync.RWMutex
    done   chan struct{}
}

type ScheduledTask struct {
    Name     string
    Interval time.Duration
    Task     func()
    ticker   *time.Ticker
    done     chan struct{}
}

func NewScheduler() *Scheduler {
    return &Scheduler{
        tasks: make(map[string]*ScheduledTask),
        done:  make(chan struct{}),
    }
}

func (s *Scheduler) AddTask(name string, interval time.Duration, task func()) {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    t := &ScheduledTask{
        Name:     name,
        Interval: interval,
        Task:     task,
        ticker:   time.NewTicker(interval),
        done:     make(chan struct{}),
    }
    
    s.tasks[name] = t
    
    go func() {
        for {
            select {
            case <-t.ticker.C:
                log.Printf("Task '%s' চালানো হচ্ছে...\n", name)
                t.Task()
            case <-t.done:
                t.ticker.Stop()
                return
            case <-s.done:
                t.ticker.Stop()
                return
            }
        }
    }()
    
    log.Printf("Task '%s' যোগ করা হয়েছে (প্রতি %v)\n", name, interval)
}

func (s *Scheduler) RemoveTask(name string) {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    if task, ok := s.tasks[name]; ok {
        close(task.done)
        delete(s.tasks, name)
        log.Printf("Task '%s' সরানো হয়েছে\n", name)
    }
}

func (s *Scheduler) Stop() {
    close(s.done)
}

func main() {
    scheduler := NewScheduler()
    
    // নিয়মিত টাস্ক যোগ করুন
    scheduler.AddTask("cleanup", 1*time.Minute, func() {
        log.Println("ক্লিনআপ চলছে...")
    })
    
    scheduler.AddTask("health-check", 30*time.Second, func() {
        log.Println("হেলথ চেক চলছে...")
    })

    app := fiber.New()

    app.Get("/tasks", func(c fiber.Ctx) error {
        scheduler.mu.RLock()
        defer scheduler.mu.RUnlock()
        
        tasks := make([]map[string]interface{}, 0)
        for name, task := range scheduler.tasks {
            tasks = append(tasks, map[string]interface{}{
                "name":     name,
                "interval": task.Interval.String(),
            })
        }
        
        return c.JSON(tasks)
    })

    app.Listen(":3000")
}
```

## Delayed Execution

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
)

type DelayedTask struct {
    ID       string
    Delay    time.Duration
    Task     func()
    timer    *time.Timer
    canceled bool
}

var delayedTasks = make(map[string]*DelayedTask)

func scheduleDelayed(id string, delay time.Duration, task func()) *DelayedTask {
    dt := &DelayedTask{
        ID:    id,
        Delay: delay,
        Task:  task,
    }
    
    dt.timer = time.AfterFunc(delay, func() {
        if !dt.canceled {
            log.Printf("Delayed task '%s' চালানো হচ্ছে...\n", id)
            task()
        }
    })
    
    delayedTasks[id] = dt
    return dt
}

func cancelDelayed(id string) bool {
    if dt, ok := delayedTasks[id]; ok {
        dt.canceled = true
        dt.timer.Stop()
        delete(delayedTasks, id)
        return true
    }
    return false
}

func main() {
    app := fiber.New()

    app.Post("/schedule", func(c fiber.Ctx) error {
        var req struct {
            TaskID string `json:"task_id"`
            Delay  int    `json:"delay"` // সেকেন্ডে
        }
        
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
        }
        
        scheduleDelayed(req.TaskID, time.Duration(req.Delay)*time.Second, func() {
            log.Printf("Task %s সম্পন্ন!\n", req.TaskID)
        })
        
        return c.JSON(fiber.Map{
            "message": "Scheduled",
            "task_id": req.TaskID,
            "delay":   req.Delay,
        })
    })

    app.Delete("/schedule/:id", func(c fiber.Ctx) error {
        taskID := c.Params("id")
        
        if cancelDelayed(taskID) {
            return c.JSON(fiber.Map{"message": "Canceled"})
        }
        
        return c.Status(404).JSON(fiber.Map{"error": "Task not found"})
    })

    app.Listen(":3000")
}
```

## Retry Scheduler

```go
package main

import (
    "errors"
    "log"
    "time"
)

type RetryConfig struct {
    MaxRetries int
    InitialDelay time.Duration
    MaxDelay     time.Duration
    Multiplier   float64
}

func retryWithBackoff(config RetryConfig, task func() error) error {
    delay := config.InitialDelay
    
    for attempt := 0; attempt <= config.MaxRetries; attempt++ {
        err := task()
        if err == nil {
            return nil
        }
        
        if attempt == config.MaxRetries {
            return err
        }
        
        log.Printf("Attempt %d ব্যর্থ, %v পর আবার চেষ্টা করা হবে\n", attempt+1, delay)
        time.Sleep(delay)
        
        // Exponential backoff
        delay = time.Duration(float64(delay) * config.Multiplier)
        if delay > config.MaxDelay {
            delay = config.MaxDelay
        }
    }
    
    return errors.New("max retries exceeded")
}

func main() {
    config := RetryConfig{
        MaxRetries:   5,
        InitialDelay: 1 * time.Second,
        MaxDelay:     30 * time.Second,
        Multiplier:   2.0,
    }
    
    attempt := 0
    err := retryWithBackoff(config, func() error {
        attempt++
        if attempt < 3 {
            return errors.New("temporary error")
        }
        return nil
    })
    
    if err != nil {
        log.Println("সব চেষ্টা ব্যর্থ:", err)
    } else {
        log.Println("সফল!")
    }
}
```

## Fiber-এ Retry Middleware

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
)

func RetryMiddleware(maxRetries int, delay time.Duration) fiber.Handler {
    return func(c fiber.Ctx) error {
        var lastErr error
        
        for attempt := 0; attempt <= maxRetries; attempt++ {
            err := c.Next()
            
            // সফল হলে ফেরত দিন
            if err == nil && c.Response().StatusCode() < 500 {
                return nil
            }
            
            lastErr = err
            
            if attempt < maxRetries {
                log.Printf("Retry %d/%d after %v\n", attempt+1, maxRetries, delay)
                time.Sleep(delay)
                
                // Response রিসেট করুন
                c.Response().Reset()
            }
        }
        
        return lastErr
    }
}

func main() {
    app := fiber.New()

    // Retry middleware শুধু নির্দিষ্ট routes-এ
    app.Get("/unstable", RetryMiddleware(3, time.Second), func(c fiber.Ctx) error {
        // সিমুলেটেড অস্থির API
        if time.Now().Unix()%3 == 0 {
            return c.Status(500).JSON(fiber.Map{"error": "Temporary error"})
        }
        return c.JSON(fiber.Map{"status": "ok"})
    })

    app.Listen(":3000")
}
```

## Deadline Scheduler

```go
package main

import (
    "context"
    "log"
    "time"
)

func runWithDeadline(deadline time.Time, task func(ctx context.Context) error) error {
    ctx, cancel := context.WithDeadline(context.Background(), deadline)
    defer cancel()
    
    done := make(chan error, 1)
    
    go func() {
        done <- task(ctx)
    }()
    
    select {
    case err := <-done:
        return err
        
    case <-ctx.Done():
        return ctx.Err()
    }
}

func main() {
    deadline := time.Now().Add(5 * time.Second)
    
    err := runWithDeadline(deadline, func(ctx context.Context) error {
        // দীর্ঘ কাজ
        for i := 0; i < 10; i++ {
            select {
            case <-ctx.Done():
                log.Println("Task বাতিল করা হয়েছে")
                return ctx.Err()
            default:
                log.Printf("Step %d\n", i+1)
                time.Sleep(time.Second)
            }
        }
        return nil
    })
    
    if err != nil {
        log.Println("Error:", err)
    }
}
```

## Scheduler Patterns Summary

```go
// ১. One-time delay
time.AfterFunc(delay, func() { ... })

// ২. Periodic
ticker := time.NewTicker(interval)
for range ticker.C { ... }

// ৩. With context
ctx, cancel := context.WithTimeout(ctx, timeout)
defer cancel()

// ৪. Retry with backoff
for attempt := 0; attempt < maxRetries; attempt++ {
    if err := task(); err == nil {
        break
    }
    time.Sleep(delay * (1 << attempt)) // exponential
}

// ৫. At specific time
duration := targetTime.Sub(time.Now())
time.AfterFunc(duration, func() { ... })
```

## সারসংক্ষেপ

| প্যাটার্ন | ব্যবহার |
|---------|--------|
| `time.Timer` | একবার নির্দিষ্ট সময় পর |
| `time.Ticker` | পর্যায়ক্রমিক |
| `time.AfterFunc` | Delayed callback |
| `context.WithTimeout` | Timeout সহ |
| Retry + Backoff | অস্থায়ী ত্রুটি হ্যান্ডলিং |

---

[← আগের: Worker Pools](../07-concurrency/04-worker-pools.md) | [পরবর্তী: ক্রন জবস →](02-cron-jobs.md)
