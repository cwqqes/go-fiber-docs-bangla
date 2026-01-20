# WaitGroups

## WaitGroup কি?

`sync.WaitGroup` গোরুটিনগুলোর সমাপ্তির জন্য অপেক্ষা করতে ব্যবহৃত হয়।

```go
import "sync"

var wg sync.WaitGroup

wg.Add(1)    // কাউন্টার বাড়ান
wg.Done()    // কাউন্টার কমান
wg.Wait()    // কাউন্টার ০ হওয়া পর্যন্ত অপেক্ষা
```

## বেসিক ব্যবহার

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done() // শেষে কাউন্টার কমান
    
    fmt.Printf("Worker %d শুরু\n", id)
    time.Sleep(time.Second)
    fmt.Printf("Worker %d শেষ\n", id)
}

func main() {
    var wg sync.WaitGroup
    
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go worker(i, &wg) // পয়েন্টার পাস করুন!
    }
    
    wg.Wait()
    fmt.Println("সব worker শেষ")
}
```

## Common Mistake: Value Copy

```go
// ❌ ভুল - WaitGroup কপি হয়ে যাচ্ছে
func worker(id int, wg sync.WaitGroup) {
    defer wg.Done() // এটি আলাদা কপিতে কাজ করছে!
    // ...
}

// ✅ সঠিক - পয়েন্টার ব্যবহার করুন
func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    // ...
}
```

## Add() কল করার সঠিক সময়

```go
// ❌ ভুল - Race condition
for i := 0; i < 5; i++ {
    go func() {
        wg.Add(1) // গোরুটিনের ভিতরে - সমস্যা!
        defer wg.Done()
        // ...
    }()
}
wg.Wait() // কিছু গোরুটিন শুরু হওয়ার আগেই Wait হতে পারে!

// ✅ সঠিক - গোরুটিন শুরুর আগে Add করুন
for i := 0; i < 5; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        // ...
    }()
}
wg.Wait()
```

## Fiber-এ Parallel Data Fetching

```go
package main

import (
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
)

type DashboardData struct {
    mu            sync.Mutex
    User          map[string]interface{} `json:"user"`
    Orders        []map[string]interface{} `json:"orders"`
    Notifications []string `json:"notifications"`
    Stats         map[string]int `json:"stats"`
}

func main() {
    app := fiber.New()

    app.Get("/dashboard/:userId", func(c fiber.Ctx) error {
        userId := c.Params("userId")
        
        var wg sync.WaitGroup
        data := &DashboardData{}
        
        // ৪টি প্যারালেল ফেচ
        wg.Add(4)
        
        go func() {
            defer wg.Done()
            user := fetchUser(userId)
            data.mu.Lock()
            data.User = user
            data.mu.Unlock()
        }()
        
        go func() {
            defer wg.Done()
            orders := fetchOrders(userId)
            data.mu.Lock()
            data.Orders = orders
            data.mu.Unlock()
        }()
        
        go func() {
            defer wg.Done()
            notifications := fetchNotifications(userId)
            data.mu.Lock()
            data.Notifications = notifications
            data.mu.Unlock()
        }()
        
        go func() {
            defer wg.Done()
            stats := fetchStats(userId)
            data.mu.Lock()
            data.Stats = stats
            data.mu.Unlock()
        }()
        
        wg.Wait()
        
        return c.JSON(data)
    })

    app.Listen(":3000")
}

func fetchUser(id string) map[string]interface{} {
    time.Sleep(100 * time.Millisecond)
    return map[string]interface{}{"id": id, "name": "User"}
}

func fetchOrders(userId string) []map[string]interface{} {
    time.Sleep(150 * time.Millisecond)
    return []map[string]interface{}{{"id": 1}, {"id": 2}}
}

func fetchNotifications(userId string) []string {
    time.Sleep(80 * time.Millisecond)
    return []string{"Notification 1", "Notification 2"}
}

func fetchStats(userId string) map[string]int {
    time.Sleep(120 * time.Millisecond)
    return map[string]int{"orders": 10, "reviews": 5}
}
```

## Error Handling with WaitGroup

```go
package main

import (
    "errors"
    "fmt"
    "sync"
)

type Result struct {
    Data  interface{}
    Error error
}

func main() {
    var wg sync.WaitGroup
    results := make(chan Result, 3)
    
    tasks := []func() (interface{}, error){
        func() (interface{}, error) { return "data1", nil },
        func() (interface{}, error) { return nil, errors.New("error") },
        func() (interface{}, error) { return "data3", nil },
    }
    
    for _, task := range tasks {
        wg.Add(1)
        go func(t func() (interface{}, error)) {
            defer wg.Done()
            data, err := t()
            results <- Result{Data: data, Error: err}
        }(task)
    }
    
    // রেজাল্ট সংগ্রহ করতে আলাদা গোরুটিন
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // রেজাল্ট প্রসেস করুন
    for result := range results {
        if result.Error != nil {
            fmt.Println("Error:", result.Error)
        } else {
            fmt.Println("Data:", result.Data)
        }
    }
}
```

## Fiber-এ Batch Processing

```go
package main

import (
    "sync"
    "github.com/gofiber/fiber/v3"
)

type BatchRequest struct {
    Items []string `json:"items"`
}

type BatchResult struct {
    Item   string `json:"item"`
    Result string `json:"result"`
    Error  string `json:"error,omitempty"`
}

func main() {
    app := fiber.New()

    app.Post("/batch", func(c fiber.Ctx) error {
        var req BatchRequest
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
        }
        
        var wg sync.WaitGroup
        var mu sync.Mutex
        results := make([]BatchResult, 0, len(req.Items))
        
        for _, item := range req.Items {
            wg.Add(1)
            go func(itm string) {
                defer wg.Done()
                
                result := processItem(itm)
                
                mu.Lock()
                results = append(results, result)
                mu.Unlock()
            }(item)
        }
        
        wg.Wait()
        
        return c.JSON(fiber.Map{
            "results": results,
            "total":   len(results),
        })
    })

    app.Listen(":3000")
}

func processItem(item string) BatchResult {
    // সিমুলেটেড প্রসেসিং
    return BatchResult{
        Item:   item,
        Result: "processed",
    }
}
```

## WaitGroup with Context

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

func workerWithContext(ctx context.Context, id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    select {
    case <-time.After(5 * time.Second):
        fmt.Printf("Worker %d সম্পন্ন\n", id)
    case <-ctx.Done():
        fmt.Printf("Worker %d বাতিল\n", id)
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    var wg sync.WaitGroup
    
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go workerWithContext(ctx, i, &wg)
    }
    
    wg.Wait()
    fmt.Println("সব worker শেষ বা বাতিল")
}
```

## Limited Concurrency

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup
    sem := make(chan struct{}, 3) // সর্বোচ্চ ৩টি concurrent
    
    for i := 1; i <= 10; i++ {
        wg.Add(1)
        
        go func(id int) {
            defer wg.Done()
            
            sem <- struct{}{} // slot নিন
            defer func() { <-sem }() // slot ফেরত দিন
            
            fmt.Printf("Worker %d শুরু\n", id)
            time.Sleep(time.Second)
            fmt.Printf("Worker %d শেষ\n", id)
        }(i)
    }
    
    wg.Wait()
    fmt.Println("সব শেষ")
}
```

## errgroup Package

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "golang.org/x/sync/errgroup"
)

func main() {
    g, ctx := errgroup.WithContext(context.Background())
    
    // প্রথম টাস্ক
    g.Go(func() error {
        return nil
    })
    
    // দ্বিতীয় টাস্ক - ত্রুটি ফেরত দেয়
    g.Go(func() error {
        return errors.New("something went wrong")
    })
    
    // তৃতীয় টাস্ক - context চেক করে
    g.Go(func() error {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            return nil
        }
    })
    
    // সব টাস্কের জন্য অপেক্ষা করুন
    if err := g.Wait(); err != nil {
        fmt.Println("Error:", err)
    }
}
```

## Fiber-এ errgroup ব্যবহার

```go
package main

import (
    "context"
    "golang.org/x/sync/errgroup"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Get("/aggregate", func(c fiber.Ctx) error {
        g, ctx := errgroup.WithContext(context.Background())
        
        var user, orders, stats interface{}
        
        g.Go(func() error {
            var err error
            user, err = fetchUserData(ctx)
            return err
        })
        
        g.Go(func() error {
            var err error
            orders, err = fetchOrdersData(ctx)
            return err
        })
        
        g.Go(func() error {
            var err error
            stats, err = fetchStatsData(ctx)
            return err
        })
        
        if err := g.Wait(); err != nil {
            return c.Status(500).JSON(fiber.Map{
                "error": err.Error(),
            })
        }
        
        return c.JSON(fiber.Map{
            "user":   user,
            "orders": orders,
            "stats":  stats,
        })
    })

    app.Listen(":3000")
}

func fetchUserData(ctx context.Context) (interface{}, error) {
    return map[string]string{"name": "User"}, nil
}

func fetchOrdersData(ctx context.Context) (interface{}, error) {
    return []string{"order1", "order2"}, nil
}

func fetchStatsData(ctx context.Context) (interface{}, error) {
    return map[string]int{"count": 10}, nil
}
```

## WaitGroup Pattern Summary

```go
// বেসিক প্যাটার্ন
var wg sync.WaitGroup
for i := 0; i < n; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        // কাজ
    }()
}
wg.Wait()

// রেজাল্ট সংগ্রহ
results := make(chan T, n)
// ...গোরুটিন...
go func() {
    wg.Wait()
    close(results)
}()
for r := range results { ... }

// Limited concurrency
sem := make(chan struct{}, limit)
// ...
sem <- struct{}{}
defer func() { <-sem }()
```

## চেকলিস্ট

- [ ] `wg.Add()` গোরুটিন শুরুর আগে কল করুন
- [ ] `defer wg.Done()` গোরুটিনের শুরুতে রাখুন
- [ ] পয়েন্টার পাস করুন, value না
- [ ] Context দিয়ে বাতিল করার ব্যবস্থা রাখুন
- [ ] Error handling যুক্ত করুন (errgroup ব্যবহার করুন)

---

[← আগের: Mutex এবং Sync](02-mutex-sync.md) | [পরবর্তী: Worker Pools →](04-worker-pools.md)
