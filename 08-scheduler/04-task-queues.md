# টাস্ক কিউ (Task Queues)

## টাস্ক কিউ কি?

টাস্ক কিউ হলো একটি সিস্টেম যেখানে কাজগুলো একটি সারিতে রাখা হয় এবং ব্যাকগ্রাউন্ডে প্রসেস করা হয়।

```
[Producer] --> [Queue] --> [Worker 1]
                       --> [Worker 2]
                       --> [Worker 3]
```

## In-Memory Queue

```go
package main

import (
    "log"
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/google/uuid"
)

type Task struct {
    ID      string
    Type    string
    Payload map[string]interface{}
}

type InMemoryQueue struct {
    tasks chan Task
    wg    sync.WaitGroup
}

func NewInMemoryQueue(bufferSize int) *InMemoryQueue {
    return &InMemoryQueue{
        tasks: make(chan Task, bufferSize),
    }
}

func (q *InMemoryQueue) Enqueue(task Task) {
    q.tasks <- task
}

func (q *InMemoryQueue) StartWorkers(count int, handler func(Task)) {
    for i := 0; i < count; i++ {
        q.wg.Add(1)
        go func(workerID int) {
            defer q.wg.Done()
            for task := range q.tasks {
                log.Printf("Worker %d: Processing task %s\n", workerID, task.ID)
                handler(task)
            }
        }(i)
    }
}

func (q *InMemoryQueue) Stop() {
    close(q.tasks)
    q.wg.Wait()
}

func main() {
    queue := NewInMemoryQueue(100)
    
    // Task handler
    queue.StartWorkers(5, func(task Task) {
        switch task.Type {
        case "email":
            log.Printf("Sending email: %v\n", task.Payload)
            time.Sleep(time.Second)
        case "sms":
            log.Printf("Sending SMS: %v\n", task.Payload)
            time.Sleep(500 * time.Millisecond)
        default:
            log.Printf("Unknown task type: %s\n", task.Type)
        }
    })

    app := fiber.New()

    app.Post("/tasks", func(c fiber.Ctx) error {
        var req struct {
            Type    string                 `json:"type"`
            Payload map[string]interface{} `json:"payload"`
        }
        
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
        }
        
        task := Task{
            ID:      uuid.New().String(),
            Type:    req.Type,
            Payload: req.Payload,
        }
        
        queue.Enqueue(task)
        
        return c.Status(202).JSON(fiber.Map{
            "task_id": task.ID,
            "message": "Task queued",
        })
    })

    app.Listen(":3000")
}
```

## Priority Queue

```go
package main

import (
    "container/heap"
    "sync"
)

type PriorityTask struct {
    Priority int
    Task     Task
    Index    int
}

type PriorityQueue []*PriorityTask

func (pq PriorityQueue) Len() int { return len(pq) }

func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].Priority > pq[j].Priority // Higher priority first
}

func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].Index = i
    pq[j].Index = j
}

func (pq *PriorityQueue) Push(x interface{}) {
    n := len(*pq)
    item := x.(*PriorityTask)
    item.Index = n
    *pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    old[n-1] = nil
    item.Index = -1
    *pq = old[0 : n-1]
    return item
}

type TaskQueue struct {
    pq     PriorityQueue
    mu     sync.Mutex
    cond   *sync.Cond
    closed bool
}

func NewTaskQueue() *TaskQueue {
    tq := &TaskQueue{
        pq: make(PriorityQueue, 0),
    }
    tq.cond = sync.NewCond(&tq.mu)
    heap.Init(&tq.pq)
    return tq
}

func (tq *TaskQueue) Enqueue(priority int, task Task) {
    tq.mu.Lock()
    defer tq.mu.Unlock()
    
    heap.Push(&tq.pq, &PriorityTask{
        Priority: priority,
        Task:     task,
    })
    tq.cond.Signal()
}

func (tq *TaskQueue) Dequeue() (Task, bool) {
    tq.mu.Lock()
    defer tq.mu.Unlock()
    
    for tq.pq.Len() == 0 && !tq.closed {
        tq.cond.Wait()
    }
    
    if tq.closed && tq.pq.Len() == 0 {
        return Task{}, false
    }
    
    item := heap.Pop(&tq.pq).(*PriorityTask)
    return item.Task, true
}

func (tq *TaskQueue) Close() {
    tq.mu.Lock()
    tq.closed = true
    tq.mu.Unlock()
    tq.cond.Broadcast()
}
```

## Redis Queue

```go
package main

import (
    "context"
    "encoding/json"
    "log"
    "time"
    "github.com/go-redis/redis/v8"
    "github.com/gofiber/fiber/v3"
    "github.com/google/uuid"
)

type RedisQueue struct {
    client *redis.Client
    name   string
}

func NewRedisQueue(addr, name string) *RedisQueue {
    return &RedisQueue{
        client: redis.NewClient(&redis.Options{
            Addr: addr,
        }),
        name: name,
    }
}

func (rq *RedisQueue) Enqueue(ctx context.Context, task Task) error {
    data, err := json.Marshal(task)
    if err != nil {
        return err
    }
    return rq.client.RPush(ctx, rq.name, data).Err()
}

func (rq *RedisQueue) Dequeue(ctx context.Context, timeout time.Duration) (*Task, error) {
    result, err := rq.client.BLPop(ctx, timeout, rq.name).Result()
    if err != nil {
        return nil, err
    }
    
    var task Task
    if err := json.Unmarshal([]byte(result[1]), &task); err != nil {
        return nil, err
    }
    
    return &task, nil
}

func (rq *RedisQueue) Len(ctx context.Context) (int64, error) {
    return rq.client.LLen(ctx, rq.name).Result()
}

func main() {
    queue := NewRedisQueue("localhost:6379", "tasks")
    ctx := context.Background()

    // Worker
    go func() {
        for {
            task, err := queue.Dequeue(ctx, 5*time.Second)
            if err != nil {
                if err != redis.Nil {
                    log.Println("Dequeue error:", err)
                }
                continue
            }
            
            log.Printf("Processing task: %s\n", task.ID)
            // Process task
        }
    }()

    app := fiber.New()

    app.Post("/tasks", func(c fiber.Ctx) error {
        var req struct {
            Type    string                 `json:"type"`
            Payload map[string]interface{} `json:"payload"`
        }
        
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
        }
        
        task := Task{
            ID:      uuid.New().String(),
            Type:    req.Type,
            Payload: req.Payload,
        }
        
        if err := queue.Enqueue(c.Context(), task); err != nil {
            return c.Status(500).JSON(fiber.Map{"error": "Queue error"})
        }
        
        return c.Status(202).JSON(fiber.Map{
            "task_id": task.ID,
        })
    })

    app.Get("/queue/length", func(c fiber.Ctx) error {
        length, err := queue.Len(c.Context())
        if err != nil {
            return c.Status(500).JSON(fiber.Map{"error": err.Error()})
        }
        return c.JSON(fiber.Map{"length": length})
    })

    app.Listen(":3000")
}
```

## Delayed Queue

```go
package main

import (
    "context"
    "encoding/json"
    "log"
    "time"
    "github.com/go-redis/redis/v8"
)

type DelayedTask struct {
    Task      Task
    ExecuteAt time.Time
}

type DelayedQueue struct {
    client *redis.Client
    name   string
}

func NewDelayedQueue(addr, name string) *DelayedQueue {
    return &DelayedQueue{
        client: redis.NewClient(&redis.Options{Addr: addr}),
        name:   name,
    }
}

func (dq *DelayedQueue) Schedule(ctx context.Context, task Task, delay time.Duration) error {
    data, err := json.Marshal(task)
    if err != nil {
        return err
    }
    
    score := float64(time.Now().Add(delay).Unix())
    return dq.client.ZAdd(ctx, dq.name, &redis.Z{
        Score:  score,
        Member: data,
    }).Err()
}

func (dq *DelayedQueue) Poll(ctx context.Context) (*Task, error) {
    now := float64(time.Now().Unix())
    
    // সময় হয়ে গেছে এমন টাস্ক নিন
    result, err := dq.client.ZRangeByScoreWithScores(ctx, dq.name, &redis.ZRangeBy{
        Min:    "-inf",
        Max:    fmt.Sprintf("%f", now),
        Offset: 0,
        Count:  1,
    }).Result()
    
    if err != nil || len(result) == 0 {
        return nil, err
    }
    
    // টাস্ক সরান
    dq.client.ZRem(ctx, dq.name, result[0].Member)
    
    var task Task
    if err := json.Unmarshal([]byte(result[0].Member.(string)), &task); err != nil {
        return nil, err
    }
    
    return &task, nil
}

func (dq *DelayedQueue) StartWorker(ctx context.Context, handler func(Task)) {
    ticker := time.NewTicker(time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            for {
                task, err := dq.Poll(ctx)
                if err != nil || task == nil {
                    break
                }
                handler(*task)
            }
        }
    }
}
```

## Dead Letter Queue

```go
package main

import (
    "context"
    "log"
    "time"
)

type DLQTask struct {
    Task       Task
    Error      string
    FailedAt   time.Time
    RetryCount int
}

type QueueWithDLQ struct {
    mainQueue *RedisQueue
    dlq       *RedisQueue
    maxRetry  int
}

func NewQueueWithDLQ(addr, name string, maxRetry int) *QueueWithDLQ {
    return &QueueWithDLQ{
        mainQueue: NewRedisQueue(addr, name),
        dlq:       NewRedisQueue(addr, name+"_dlq"),
        maxRetry:  maxRetry,
    }
}

func (q *QueueWithDLQ) Process(ctx context.Context, handler func(Task) error) {
    for {
        task, err := q.mainQueue.Dequeue(ctx, 5*time.Second)
        if err != nil {
            continue
        }
        
        err = handler(*task)
        if err != nil {
            // Retry count চেক করুন (payload এ store করা)
            retryCount := 0
            if rc, ok := task.Payload["_retry_count"].(float64); ok {
                retryCount = int(rc)
            }
            
            if retryCount >= q.maxRetry {
                // DLQ তে পাঠান
                log.Printf("Moving task %s to DLQ after %d retries\n", task.ID, retryCount)
                dlqTask := DLQTask{
                    Task:       *task,
                    Error:      err.Error(),
                    FailedAt:   time.Now(),
                    RetryCount: retryCount,
                }
                // DLQ তে স্টোর করুন
                q.moveToDLQ(ctx, dlqTask)
            } else {
                // রিকিউ করুন
                task.Payload["_retry_count"] = retryCount + 1
                q.mainQueue.Enqueue(ctx, *task)
            }
        }
    }
}

func (q *QueueWithDLQ) moveToDLQ(ctx context.Context, task DLQTask) {
    // DLQ তে টাস্ক স্টোর করুন
}
```

## Fiber Integration

```go
package main

import (
    "context"
    "log"
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/google/uuid"
)

type TaskHandler func(Task) error

type TaskQueue struct {
    queue    chan Task
    handlers map[string]TaskHandler
    wg       sync.WaitGroup
    mu       sync.RWMutex
}

func NewTaskQueue(bufferSize int) *TaskQueue {
    return &TaskQueue{
        queue:    make(chan Task, bufferSize),
        handlers: make(map[string]TaskHandler),
    }
}

func (tq *TaskQueue) RegisterHandler(taskType string, handler TaskHandler) {
    tq.mu.Lock()
    defer tq.mu.Unlock()
    tq.handlers[taskType] = handler
}

func (tq *TaskQueue) StartWorkers(count int) {
    for i := 0; i < count; i++ {
        tq.wg.Add(1)
        go tq.worker(i)
    }
}

func (tq *TaskQueue) worker(id int) {
    defer tq.wg.Done()
    
    for task := range tq.queue {
        tq.mu.RLock()
        handler, ok := tq.handlers[task.Type]
        tq.mu.RUnlock()
        
        if !ok {
            log.Printf("Worker %d: No handler for task type %s\n", id, task.Type)
            continue
        }
        
        if err := handler(task); err != nil {
            log.Printf("Worker %d: Task %s failed: %v\n", id, task.ID, err)
        } else {
            log.Printf("Worker %d: Task %s completed\n", id, task.ID)
        }
    }
}

func (tq *TaskQueue) Enqueue(task Task) {
    tq.queue <- task
}

func (tq *TaskQueue) Stop() {
    close(tq.queue)
    tq.wg.Wait()
}

func main() {
    tq := NewTaskQueue(100)
    
    // হ্যান্ডলার রেজিস্টার করুন
    tq.RegisterHandler("email", func(task Task) error {
        log.Printf("Sending email: %v\n", task.Payload)
        time.Sleep(time.Second)
        return nil
    })
    
    tq.RegisterHandler("sms", func(task Task) error {
        log.Printf("Sending SMS: %v\n", task.Payload)
        time.Sleep(500 * time.Millisecond)
        return nil
    })
    
    tq.RegisterHandler("notification", func(task Task) error {
        log.Printf("Sending notification: %v\n", task.Payload)
        return nil
    })
    
    tq.StartWorkers(5)

    app := fiber.New()

    app.Post("/queue/:type", func(c fiber.Ctx) error {
        taskType := c.Params("type")
        
        var payload map[string]interface{}
        if err := c.Bind().JSON(&payload); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid payload"})
        }
        
        task := Task{
            ID:      uuid.New().String(),
            Type:    taskType,
            Payload: payload,
        }
        
        tq.Enqueue(task)
        
        return c.Status(202).JSON(fiber.Map{
            "task_id": task.ID,
            "type":    taskType,
            "status":  "queued",
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

## সারসংক্ষেপ

| Queue Type | ব্যবহার |
|-----------|--------|
| In-Memory | সিম্পল, একক সার্ভার |
| Redis | ডিস্ট্রিবিউটেড, persistent |
| Priority | গুরুত্ব অনুযায়ী প্রসেসিং |
| Delayed | নির্দিষ্ট সময়ে এক্সিকিউশন |
| DLQ | ব্যর্থ টাস্ক ম্যানেজমেন্ট |

---

[← আগের: ব্যাকগ্রাউন্ড টাস্ক](03-background-tasks.md) | [পরবর্তী: এরর হ্যান্ডলিং →](../09-advanced/01-error-handling.md)
