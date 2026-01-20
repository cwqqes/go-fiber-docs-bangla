# চ্যানেল বেসিক (Channel Basics)

## চ্যানেল কি?

চ্যানেল হলো Go-এর বিল্ট-ইন ডেটা স্ট্রাকচার যা গোরুটিনদের মধ্যে নিরাপদে ডেটা আদান-প্রদান করতে দেয়।

```go
// চ্যানেল তৈরি
ch := make(chan int)      // int টাইপের চ্যানেল
strCh := make(chan string) // string টাইপের চ্যানেল
```

## চ্যানেল অপারেশন

```go
package main

import "fmt"

func main() {
    ch := make(chan string)
    
    // ডেটা পাঠান (গোরুটিনে)
    go func() {
        ch <- "Hello, Fiber!" // চ্যানেলে পাঠান
    }()
    
    // ডেটা গ্রহণ করুন
    msg := <-ch // চ্যানেল থেকে পড়ুন
    fmt.Println(msg)
}
```

## Unbuffered Channel

Unbuffered চ্যানেল সিঙ্ক্রোনাস - sender অপেক্ষা করে যতক্ষণ না receiver রেডি হয়।

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int) // unbuffered
    
    go func() {
        fmt.Println("পাঠানোর আগে...")
        ch <- 42 // receiver রেডি না হওয়া পর্যন্ত ব্লক
        fmt.Println("পাঠানোর পর")
    }()
    
    time.Sleep(2 * time.Second)
    
    fmt.Println("গ্রহণের আগে...")
    value := <-ch
    fmt.Println("প্রাপ্ত মান:", value)
}

// আউটপুট:
// পাঠানোর আগে...
// (২ সেকেন্ড অপেক্ষা)
// গ্রহণের আগে...
// পাঠানোর পর
// প্রাপ্ত মান: 42
```

## চ্যানেল বন্ধ করা

```go
package main

import "fmt"

func main() {
    ch := make(chan int)
    
    go func() {
        for i := 1; i <= 5; i++ {
            ch <- i
        }
        close(ch) // চ্যানেল বন্ধ করুন
    }()
    
    // range দিয়ে পড়ুন - চ্যানেল বন্ধ হলে লুপ শেষ
    for value := range ch {
        fmt.Println(value)
    }
}
```

## চ্যানেল স্ট্যাটাস চেক

```go
// চ্যানেল খোলা আছে কিনা চেক করুন
value, ok := <-ch
if !ok {
    fmt.Println("চ্যানেল বন্ধ")
}
```

## Fiber-এ চ্যানেল ব্যবহার

```go
package main

import (
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Get("/async", func(c fiber.Ctx) error {
        resultChan := make(chan string)
        
        go func() {
            time.Sleep(100 * time.Millisecond)
            resultChan <- "ফলাফল প্রস্তুত!"
        }()
        
        result := <-resultChan
        
        return c.JSON(fiber.Map{
            "result": result,
        })
    })

    app.Listen(":3000")
}
```

## Multi-Source Data Fetching

```go
package main

import (
    "time"
    "github.com/gofiber/fiber/v3"
)

type APIResponse struct {
    Source string
    Data   string
    Error  error
}

func main() {
    app := fiber.New()

    app.Get("/aggregate", func(c fiber.Ctx) error {
        ch := make(chan APIResponse, 3) // ৩টি সোর্স
        
        // সমান্তরালে ডেটা ফেচ করুন
        go func() {
            ch <- APIResponse{Source: "API-1", Data: fetchAPI1()}
        }()
        go func() {
            ch <- APIResponse{Source: "API-2", Data: fetchAPI2()}
        }()
        go func() {
            ch <- APIResponse{Source: "API-3", Data: fetchAPI3()}
        }()
        
        // সব রেসপন্স সংগ্রহ করুন
        results := make(map[string]string)
        for i := 0; i < 3; i++ {
            resp := <-ch
            if resp.Error == nil {
                results[resp.Source] = resp.Data
            }
        }
        
        return c.JSON(results)
    })

    app.Listen(":3000")
}

func fetchAPI1() string {
    time.Sleep(100 * time.Millisecond)
    return "Data from API 1"
}

func fetchAPI2() string {
    time.Sleep(150 * time.Millisecond)
    return "Data from API 2"
}

func fetchAPI3() string {
    time.Sleep(200 * time.Millisecond)
    return "Data from API 3"
}
```

## Done Channel Pattern

```go
package main

import (
    "fmt"
    "time"
)

func worker(done <-chan struct{}) {
    for {
        select {
        case <-done:
            fmt.Println("ওয়ার্কার বন্ধ হচ্ছে...")
            return
        default:
            fmt.Println("কাজ করছি...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    done := make(chan struct{})
    
    go worker(done)
    
    time.Sleep(2 * time.Second)
    
    close(done) // ওয়ার্কার বন্ধ করুন
    
    time.Sleep(time.Second) // বন্ধ হওয়ার জন্য অপেক্ষা
}
```

## Directional Channels

```go
// শুধু পাঠানোর চ্যানেল
func producer(ch chan<- int) {
    for i := 1; i <= 10; i++ {
        ch <- i
    }
    close(ch)
}

// শুধু পড়ার চ্যানেল
func consumer(ch <-chan int) {
    for value := range ch {
        fmt.Println("প্রাপ্ত:", value)
    }
}

func main() {
    ch := make(chan int)
    
    go producer(ch)
    consumer(ch)
}
```

## চ্যানেল টাইপস টেবিল

| টাইপ | সিনট্যাক্স | ব্যবহার |
|-----|----------|--------|
| Bidirectional | `chan T` | পড়া ও লেখা দুইই |
| Send-only | `chan<- T` | শুধু পাঠানো |
| Receive-only | `<-chan T` | শুধু পড়া |
| Buffered | `make(chan T, n)` | n সাইজের বাফার |

## চ্যানেল nil চেক

```go
var ch chan int // nil channel

// nil চ্যানেলে পড়া/লেখা চিরকাল ব্লক করে
// select এ ব্যবহার করে এড়ানো যায়

select {
case <-ch:
    // এই কেস কখনো চলবে না (nil channel)
default:
    fmt.Println("চ্যানেল nil")
}
```

## সারসংক্ষেপ

```go
// তৈরি
ch := make(chan int)       // unbuffered
ch := make(chan int, 10)   // buffered (10)

// পাঠান
ch <- value

// গ্রহণ
value := <-ch
value, ok := <-ch  // ok=false মানে চ্যানেল বন্ধ

// বন্ধ
close(ch)

// range
for v := range ch { ... }
```

---

[← আগের: সেফ প্র্যাকটিস](../05-goroutines/03-safe-practices.md) | [পরবর্তী: বাফার্ড চ্যানেল →](02-buffered-channels.md)
