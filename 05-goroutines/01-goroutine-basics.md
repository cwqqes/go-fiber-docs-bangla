# গোরুটিন বেসিক (Goroutine Basics)

## গোরুটিন কী?

গোরুটিন হলো Go-তে লাইটওয়েট থ্রেড। এটি একই সময়ে একাধিক কাজ চালাতে সাহায্য করে (Concurrency)।

```go
// সাধারণ ফাংশন কল
doWork()

// গোরুটিন হিসেবে কল (go কীওয়ার্ড)
go doWork()
```

## বেসিক উদাহরণ

```go
package main

import (
    "fmt"
    "time"
)

func sayHello(name string) {
    for i := 0; i < 3; i++ {
        fmt.Println("হ্যালো", name)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    // গোরুটিন শুরু করুন
    go sayHello("রহিম")
    go sayHello("করিম")
    
    // মেইন থ্রেড অপেক্ষা করুক
    time.Sleep(500 * time.Millisecond)
    
    fmt.Println("মেইন শেষ")
}
```

## Anonymous ফাংশন গোরুটিন

```go
go func() {
    fmt.Println("Anonymous গোরুটিন")
}()

// প্যারামিটার সহ
name := "রহিম"
go func(n string) {
    fmt.Println("হ্যালো", n)
}(name)
```

## গোরুটিন vs থ্রেড

| বৈশিষ্ট্য | গোরুটিন | OS থ্রেড |
|----------|---------|----------|
| মেমোরি | ~2KB | ~1MB |
| স্টার্টআপ | দ্রুত | ধীর |
| ম্যানেজমেন্ট | Go রানটাইম | OS |
| সংখ্যা | লক্ষাধিক সম্ভব | শত/হাজার |

## WaitGroup দিয়ে অপেক্ষা

`time.Sleep()` এর বদলে `WaitGroup` ব্যবহার করুন:

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1) // কাউন্টার বাড়ান
        
        go func(num int) {
            defer wg.Done() // কাউন্টার কমান
            fmt.Println("গোরুটিন", num)
        }(i)
    }

    wg.Wait() // সব শেষ হওয়া পর্যন্ত অপেক্ষা
    fmt.Println("সব শেষ")
}
```

## গোরুটিনে ডেটা পাস

### ক্লোজার সমস্যা

```go
// ভুল - সব গোরুটিন একই i দেখবে
for i := 0; i < 5; i++ {
    go func() {
        fmt.Println(i) // সমস্যা!
    }()
}

// সঠিক - প্যারামিটার হিসেবে পাস করুন
for i := 0; i < 5; i++ {
    go func(num int) {
        fmt.Println(num)
    }(i)
}
```

## গোরুটিন লিক প্রতিরোধ

```go
// সমস্যা - এই গোরুটিন কখনো শেষ হবে না
go func() {
    for {
        // অসীম লুপ
    }
}()

// সমাধান - বাতিল করার উপায় রাখুন
done := make(chan bool)

go func() {
    for {
        select {
        case <-done:
            return // বাতিল হলে বের হয়ে যান
        default:
            // কাজ করুন
        }
    }
}()

// বাতিল করতে
close(done)
```

## Context দিয়ে বাতিল করা

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context, id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d বাতিল হয়েছে\n", id)
            return
        default:
            fmt.Printf("Worker %d কাজ করছে\n", id)
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    // ২ সেকেন্ড পর বাতিল হবে
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()

    go worker(ctx, 1)
    go worker(ctx, 2)

    <-ctx.Done()
    fmt.Println("সব শেষ")
}
```

## Error Handling

```go
package main

import (
    "errors"
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    errChan := make(chan error, 3)

    tasks := []string{"task1", "task2", "task3"}

    for _, task := range tasks {
        wg.Add(1)
        go func(t string) {
            defer wg.Done()
            
            if err := doTask(t); err != nil {
                errChan <- err
            }
        }(task)
    }

    // সব শেষ হলে চ্যানেল বন্ধ করুন
    go func() {
        wg.Wait()
        close(errChan)
    }()

    // এরর পড়ুন
    for err := range errChan {
        fmt.Println("Error:", err)
    }
}

func doTask(name string) error {
    if name == "task2" {
        return errors.New("task2 ব্যর্থ")
    }
    fmt.Println(name, "সফল")
    return nil
}
```

## Race Condition

```go
package main

import (
    "fmt"
    "sync"
)

// সমস্যা - Race condition
var counter int

func increment() {
    counter++ // একাধিক গোরুটিন একসাথে এটি পরিবর্তন করতে পারে
}

// সমাধান - Mutex ব্যবহার করুন
var (
    safeCounter int
    mutex       sync.Mutex
)

func safeIncrement() {
    mutex.Lock()
    safeCounter++
    mutex.Unlock()
}

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            safeIncrement()
        }()
    }

    wg.Wait()
    fmt.Println("Counter:", safeCounter)
}
```

## Race Detector

```bash
# Race condition খুঁজতে
go run -race main.go

# অথবা টেস্টে
go test -race ./...
```

## গোরুটিন পরিসংখ্যান

```go
import "runtime"

// বর্তমান গোরুটিন সংখ্যা
count := runtime.NumGoroutine()
fmt.Println("গোরুটিন সংখ্যা:", count)

// GOMAXPROCS (CPU কোর ব্যবহার)
cpus := runtime.GOMAXPROCS(0)
fmt.Println("CPU ব্যবহার:", cpus)
```

## বেস্ট প্র্যাকটিস

1. **WaitGroup ব্যবহার করুন** - `time.Sleep()` নয়
2. **গোরুটিন লিক এড়িয়ে চলুন** - বাতিল করার উপায় রাখুন
3. **ভেরিয়েবল কপি করুন** - ক্লোজার সমস্যা এড়াতে
4. **Mutex/Channel ব্যবহার করুন** - শেয়ার্ড ডেটায়
5. **Race detector চালান** - টেস্টে `-race` ফ্ল্যাগ

## সম্পূর্ণ উদাহরণ

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()

    var wg sync.WaitGroup
    results := make(chan string, 5)

    // ৫টি worker শুরু করুন
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            select {
            case <-ctx.Done():
                return
            case results <- fmt.Sprintf("Worker %d সম্পন্ন", id):
                time.Sleep(500 * time.Millisecond)
            }
        }(i)
    }

    // রেজাল্ট সংগ্রহ করুন
    go func() {
        wg.Wait()
        close(results)
    }()

    // রেজাল্ট প্রিন্ট করুন
    for result := range results {
        fmt.Println(result)
    }

    fmt.Println("প্রোগ্রাম শেষ")
}
```

---

[← আগের: লোকালস](../04-context/04-locals.md) | [পরবর্তী: Fiber-এ গোরুটিন →](02-fiber-goroutines.md)
