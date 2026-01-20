# Mutex এবং Sync (Mutex and Synchronization)

## Mutex কি?

Mutex (Mutual Exclusion) হলো একটি লক মেকানিজম যা একসাথে একটি মাত্র গোরুটিনকে কোড ব্লক এক্সিকিউট করতে দেয়।

```go
import "sync"

var mu sync.Mutex

mu.Lock()   // লক অর্জন করুন
// critical section
mu.Unlock() // লক মুক্ত করুন
```

## বেসিক Mutex ব্যবহার

```go
package main

import (
    "fmt"
    "sync"
)

type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

func main() {
    counter := &Counter{}
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter.Increment()
        }()
    }
    
    wg.Wait()
    fmt.Println("Count:", counter.Value()) // সবসময় ১০০০
}
```

## RWMutex (Read-Write Mutex)

RWMutex একাধিক reader-কে একসাথে পড়তে দেয়, কিন্তু writer একা থাকে।

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

func NewCache() *Cache {
    return &Cache{
        data: make(map[string]string),
    }
}

// Read - একাধিক গোরুটিন একসাথে পড়তে পারে
func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    value, ok := c.data[key]
    return value, ok
}

// Write - শুধু একটি গোরুটিন
func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}

func main() {
    cache := NewCache()
    cache.Set("name", "Fiber")
    
    var wg sync.WaitGroup
    
    // অনেক reader
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            if value, ok := cache.Get("name"); ok {
                fmt.Println("Read:", value)
            }
        }()
    }
    
    // একটি writer
    wg.Add(1)
    go func() {
        defer wg.Done()
        cache.Set("name", "Go Fiber")
    }()
    
    wg.Wait()
}
```

## Fiber-এ Thread-Safe Cache

```go
package main

import (
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
)

type CacheItem struct {
    Value      interface{}
    Expiration time.Time
}

type MemoryCache struct {
    mu    sync.RWMutex
    items map[string]CacheItem
}

func NewMemoryCache() *MemoryCache {
    cache := &MemoryCache{
        items: make(map[string]CacheItem),
    }
    
    // ক্লিনআপ গোরুটিন
    go cache.cleanupLoop()
    
    return cache
}

func (c *MemoryCache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    item, ok := c.items[key]
    if !ok || time.Now().After(item.Expiration) {
        return nil, false
    }
    return item.Value, true
}

func (c *MemoryCache) Set(key string, value interface{}, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.items[key] = CacheItem{
        Value:      value,
        Expiration: time.Now().Add(ttl),
    }
}

func (c *MemoryCache) Delete(key string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    delete(c.items, key)
}

func (c *MemoryCache) cleanupLoop() {
    ticker := time.NewTicker(time.Minute)
    for range ticker.C {
        c.mu.Lock()
        for key, item := range c.items {
            if time.Now().After(item.Expiration) {
                delete(c.items, key)
            }
        }
        c.mu.Unlock()
    }
}

func main() {
    cache := NewMemoryCache()
    
    app := fiber.New()

    app.Get("/cache/:key", func(c fiber.Ctx) error {
        key := c.Params("key")
        
        if value, ok := cache.Get(key); ok {
            return c.JSON(fiber.Map{
                "key":   key,
                "value": value,
                "hit":   true,
            })
        }
        
        return c.Status(404).JSON(fiber.Map{
            "error": "Key not found",
            "hit":   false,
        })
    })

    app.Post("/cache/:key", func(c fiber.Ctx) error {
        key := c.Params("key")
        
        var body struct {
            Value string `json:"value"`
            TTL   int    `json:"ttl"` // সেকেন্ডে
        }
        
        if err := c.Bind().JSON(&body); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid body"})
        }
        
        ttl := time.Duration(body.TTL) * time.Second
        if ttl == 0 {
            ttl = 5 * time.Minute
        }
        
        cache.Set(key, body.Value, ttl)
        
        return c.JSON(fiber.Map{
            "message": "Cached successfully",
        })
    })

    app.Listen(":3000")
}
```

## Deadlock এড়ানো

```go
// ❌ ভুল - Deadlock!
func (c *Counter) BadMethod() {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    // এই ফাংশনও Lock করে - Deadlock!
    c.AnotherMethod()
}

func (c *Counter) AnotherMethod() {
    c.mu.Lock() // আগে থেকেই locked!
    defer c.mu.Unlock()
    // ...
}

// ✅ সঠিক - Internal method ব্যবহার করুন
func (c *Counter) GoodMethod() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.anotherMethodLocked() // লক ছাড়া
}

func (c *Counter) anotherMethodLocked() {
    // লক ধরা আছে ধরে নেয়
    // ...
}
```

## sync.Cond (Condition Variable)

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Queue struct {
    mu    sync.Mutex
    cond  *sync.Cond
    items []string
}

func NewQueue() *Queue {
    q := &Queue{
        items: make([]string, 0),
    }
    q.cond = sync.NewCond(&q.mu)
    return q
}

func (q *Queue) Enqueue(item string) {
    q.mu.Lock()
    defer q.mu.Unlock()
    
    q.items = append(q.items, item)
    q.cond.Signal() // একটি waiting গোরুটিনকে জাগান
}

func (q *Queue) Dequeue() string {
    q.mu.Lock()
    defer q.mu.Unlock()
    
    // খালি থাকলে অপেক্ষা করুন
    for len(q.items) == 0 {
        q.cond.Wait() // লক ছেড়ে অপেক্ষা করে
    }
    
    item := q.items[0]
    q.items = q.items[1:]
    return item
}

func main() {
    queue := NewQueue()
    
    // Consumer
    go func() {
        for {
            item := queue.Dequeue()
            fmt.Println("Dequeued:", item)
        }
    }()
    
    // Producer
    for i := 1; i <= 5; i++ {
        queue.Enqueue(fmt.Sprintf("Item-%d", i))
        time.Sleep(500 * time.Millisecond)
    }
    
    time.Sleep(time.Second)
}
```

## Connection Pool

```go
package main

import (
    "errors"
    "sync"
    "time"
)

type Connection struct {
    ID int
}

func (c *Connection) Close() {
    // connection বন্ধ করুন
}

type ConnectionPool struct {
    mu          sync.Mutex
    cond        *sync.Cond
    connections []*Connection
    maxSize     int
    created     int
}

func NewConnectionPool(maxSize int) *ConnectionPool {
    pool := &ConnectionPool{
        connections: make([]*Connection, 0),
        maxSize:     maxSize,
    }
    pool.cond = sync.NewCond(&pool.mu)
    return pool
}

func (p *ConnectionPool) Get(timeout time.Duration) (*Connection, error) {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    deadline := time.Now().Add(timeout)
    
    for {
        // পুলে connection আছে?
        if len(p.connections) > 0 {
            conn := p.connections[0]
            p.connections = p.connections[1:]
            return conn, nil
        }
        
        // নতুন তৈরি করতে পারি?
        if p.created < p.maxSize {
            p.created++
            return &Connection{ID: p.created}, nil
        }
        
        // টাইমআউট চেক
        if time.Now().After(deadline) {
            return nil, errors.New("connection timeout")
        }
        
        // অপেক্ষা করুন
        p.cond.Wait()
    }
}

func (p *ConnectionPool) Put(conn *Connection) {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    p.connections = append(p.connections, conn)
    p.cond.Signal()
}

func (p *ConnectionPool) Close() {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    for _, conn := range p.connections {
        conn.Close()
    }
    p.connections = nil
}
```

## Fiber-এ DB Pool

```go
package main

import (
    "database/sql"
    "github.com/gofiber/fiber/v3"
    _ "github.com/lib/pq"
)

func main() {
    // sql.DB নিজেই connection pool!
    db, err := sql.Open("postgres", "postgres://...")
    if err != nil {
        panic(err)
    }
    
    // Pool কনফিগারেশন
    db.SetMaxOpenConns(25)              // সর্বোচ্চ open connections
    db.SetMaxIdleConns(5)               // সর্বোচ্চ idle connections
    db.SetConnMaxLifetime(5 * time.Minute) // connection lifetime

    app := fiber.New()

    app.Get("/users/:id", func(c fiber.Ctx) error {
        id := c.Params("id")
        
        var name string
        err := db.QueryRow("SELECT name FROM users WHERE id = $1", id).Scan(&name)
        if err != nil {
            return c.Status(500).JSON(fiber.Map{"error": err.Error()})
        }
        
        return c.JSON(fiber.Map{"name": name})
    })

    app.Listen(":3000")
}
```

## TryLock Pattern

```go
package main

import (
    "sync"
    "time"
)

type TryMutex struct {
    ch chan struct{}
}

func NewTryMutex() *TryMutex {
    return &TryMutex{
        ch: make(chan struct{}, 1),
    }
}

func (m *TryMutex) Lock() {
    m.ch <- struct{}{}
}

func (m *TryMutex) Unlock() {
    <-m.ch
}

func (m *TryMutex) TryLock() bool {
    select {
    case m.ch <- struct{}{}:
        return true
    default:
        return false
    }
}

func (m *TryMutex) TryLockTimeout(timeout time.Duration) bool {
    select {
    case m.ch <- struct{}{}:
        return true
    case <-time.After(timeout):
        return false
    }
}

// ব্যবহার
func main() {
    mu := NewTryMutex()
    
    if mu.TryLock() {
        defer mu.Unlock()
        // critical section
    } else {
        // lock পাওয়া যায়নি
    }
}
```

## Mutex বেস্ট প্র্যাকটিসেস

```go
// ✅ defer ব্যবহার করুন
mu.Lock()
defer mu.Unlock()

// ✅ ছোট critical section
mu.Lock()
value := sharedData
mu.Unlock()
// value প্রসেস করুন (লক ছাড়া)

// ✅ RWMutex ব্যবহার করুন বেশি read হলে
var rw sync.RWMutex
rw.RLock()   // একাধিক reader
rw.RUnlock()

// ❌ লক ধরে I/O করবেন না
mu.Lock()
http.Get("...") // ভুল!
mu.Unlock()

// ❌ নেস্টেড লক এড়িয়ে চলুন
mu1.Lock()
mu2.Lock() // Deadlock ঝুঁকি!
```

## সারসংক্ষেপ

| টাইপ | ব্যবহার | Read | Write |
|-----|--------|------|-------|
| `sync.Mutex` | এক্সক্লুসিভ | ১ | ১ |
| `sync.RWMutex` | R/W আলাদা | অনেক | ১ |
| `sync.Cond` | সিগন্যালিং | - | - |
| `sync/atomic` | সিম্পল ভ্যালু | ∞ | ∞ |

---

[← আগের: কনকারেন্সি বেসিক](01-concurrency-basics.md) | [পরবর্তী: WaitGroups →](03-waitgroups.md)
