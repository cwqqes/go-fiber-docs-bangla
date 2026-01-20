# Worker Pools

## Worker Pool কি?

Worker Pool হলো একটি প্যাটার্ন যেখানে নির্দিষ্ট সংখ্যক গোরুটিন (workers) একটি কাজের সারি (job queue) থেকে কাজ নিয়ে প্রসেস করে।

```
[Jobs Queue] --> [Worker 1]
             --> [Worker 2]
             --> [Worker 3]
             --> [Results Queue]
```

## বেসিক Worker Pool

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Job struct {
    ID   int
    Data string
}

type Result struct {
    JobID  int
    Output string
}

func worker(id int, jobs <-chan Job, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for job := range jobs {
        fmt.Printf("Worker %d: Job %d প্রসেসিং\n", id, job.ID)
        time.Sleep(100 * time.Millisecond) // সিমুলেটেড কাজ
        
        results <- Result{
            JobID:  job.ID,
            Output: fmt.Sprintf("Processed: %s", job.Data),
        }
    }
    
    fmt.Printf("Worker %d: শেষ\n", id)
}

func main() {
    numWorkers := 3
    numJobs := 10
    
    jobs := make(chan Job, numJobs)
    results := make(chan Result, numJobs)
    
    var wg sync.WaitGroup
    
    // Workers শুরু করুন
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go worker(i, jobs, results, &wg)
    }
    
    // Jobs পাঠান
    for i := 1; i <= numJobs; i++ {
        jobs <- Job{ID: i, Data: fmt.Sprintf("data-%d", i)}
    }
    close(jobs)
    
    // Workers শেষ হওয়ার জন্য অপেক্ষা
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Results সংগ্রহ করুন
    for result := range results {
        fmt.Printf("Result: Job %d -> %s\n", result.JobID, result.Output)
    }
}
```

## Fiber-এ Worker Pool

```go
package main

import (
    "fmt"
    "log"
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/google/uuid"
)

type Task struct {
    ID     string
    Type   string
    Data   map[string]interface{}
    Status string
}

var (
    taskQueue  = make(chan Task, 100)
    taskStatus = sync.Map{}
)

func worker(id int) {
    for task := range taskQueue {
        log.Printf("Worker %d: Task %s শুরু\n", id, task.ID)
        
        // স্ট্যাটাস আপডেট করুন
        task.Status = "processing"
        taskStatus.Store(task.ID, task)
        
        // কাজ করুন
        time.Sleep(2 * time.Second)
        
        // সম্পন্ন
        task.Status = "completed"
        taskStatus.Store(task.ID, task)
        
        log.Printf("Worker %d: Task %s সম্পন্ন\n", id, task.ID)
    }
}

func main() {
    // ৫টি worker শুরু করুন
    for i := 1; i <= 5; i++ {
        go worker(i)
    }

    app := fiber.New()

    // নতুন টাস্ক তৈরি করুন
    app.Post("/tasks", func(c fiber.Ctx) error {
        var data map[string]interface{}
        if err := c.Bind().JSON(&data); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid data"})
        }
        
        task := Task{
            ID:     uuid.New().String(),
            Type:   "process",
            Data:   data,
            Status: "queued",
        }
        
        // Non-blocking queue
        select {
        case taskQueue <- task:
            taskStatus.Store(task.ID, task)
            return c.Status(202).JSON(fiber.Map{
                "task_id": task.ID,
                "status":  "queued",
            })
        default:
            return c.Status(503).JSON(fiber.Map{
                "error": "Queue full",
            })
        }
    })

    // টাস্ক স্ট্যাটাস চেক করুন
    app.Get("/tasks/:id", func(c fiber.Ctx) error {
        taskID := c.Params("id")
        
        if task, ok := taskStatus.Load(taskID); ok {
            return c.JSON(task)
        }
        
        return c.Status(404).JSON(fiber.Map{
            "error": "Task not found",
        })
    })

    // Queue স্ট্যাটাস
    app.Get("/queue/status", func(c fiber.Ctx) error {
        return c.JSON(fiber.Map{
            "queued":   len(taskQueue),
            "capacity": cap(taskQueue),
        })
    })

    log.Fatal(app.Listen(":3000"))
}
```

## Dynamic Worker Pool

```go
package main

import (
    "context"
    "log"
    "sync"
    "sync/atomic"
    "time"
)

type DynamicPool struct {
    jobs         chan func()
    workerCount  int32
    minWorkers   int32
    maxWorkers   int32
    wg           sync.WaitGroup
    ctx          context.Context
    cancel       context.CancelFunc
}

func NewDynamicPool(min, max int32) *DynamicPool {
    ctx, cancel := context.WithCancel(context.Background())
    
    p := &DynamicPool{
        jobs:       make(chan func(), 100),
        minWorkers: min,
        maxWorkers: max,
        ctx:        ctx,
        cancel:     cancel,
    }
    
    // ন্যূনতম workers শুরু করুন
    for i := int32(0); i < min; i++ {
        p.addWorker()
    }
    
    // মনিটরিং গোরুটিন
    go p.monitor()
    
    return p
}

func (p *DynamicPool) addWorker() {
    atomic.AddInt32(&p.workerCount, 1)
    p.wg.Add(1)
    
    go func() {
        defer p.wg.Done()
        defer atomic.AddInt32(&p.workerCount, -1)
        
        idleTimeout := time.NewTimer(30 * time.Second)
        defer idleTimeout.Stop()
        
        for {
            select {
            case job := <-p.jobs:
                idleTimeout.Reset(30 * time.Second)
                job()
                
            case <-idleTimeout.C:
                // Idle timeout - extra worker বন্ধ করুন
                if atomic.LoadInt32(&p.workerCount) > p.minWorkers {
                    return
                }
                idleTimeout.Reset(30 * time.Second)
                
            case <-p.ctx.Done():
                return
            }
        }
    }()
}

func (p *DynamicPool) monitor() {
    ticker := time.NewTicker(time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            queueLen := len(p.jobs)
            currentWorkers := atomic.LoadInt32(&p.workerCount)
            
            // Queue বেশি হলে worker বাড়ান
            if queueLen > int(currentWorkers)*2 && currentWorkers < p.maxWorkers {
                log.Println("Worker বাড়ানো হচ্ছে...")
                p.addWorker()
            }
            
        case <-p.ctx.Done():
            return
        }
    }
}

func (p *DynamicPool) Submit(job func()) {
    select {
    case p.jobs <- job:
    case <-p.ctx.Done():
    }
}

func (p *DynamicPool) Stop() {
    p.cancel()
    p.wg.Wait()
}
```

## Priority Worker Pool

```go
package main

import (
    "container/heap"
    "sync"
)

type PriorityJob struct {
    Priority int
    Job      func()
    Index    int
}

type PriorityQueue []*PriorityJob

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
    job := x.(*PriorityJob)
    job.Index = n
    *pq = append(*pq, job)
}

func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    job := old[n-1]
    old[n-1] = nil
    job.Index = -1
    *pq = old[0 : n-1]
    return job
}

type PriorityPool struct {
    mu     sync.Mutex
    cond   *sync.Cond
    queue  PriorityQueue
    closed bool
}

func NewPriorityPool(numWorkers int) *PriorityPool {
    p := &PriorityPool{
        queue: make(PriorityQueue, 0),
    }
    p.cond = sync.NewCond(&p.mu)
    heap.Init(&p.queue)
    
    for i := 0; i < numWorkers; i++ {
        go p.worker()
    }
    
    return p
}

func (p *PriorityPool) worker() {
    for {
        p.mu.Lock()
        
        for p.queue.Len() == 0 && !p.closed {
            p.cond.Wait()
        }
        
        if p.closed && p.queue.Len() == 0 {
            p.mu.Unlock()
            return
        }
        
        job := heap.Pop(&p.queue).(*PriorityJob)
        p.mu.Unlock()
        
        job.Job()
    }
}

func (p *PriorityPool) Submit(priority int, job func()) {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    heap.Push(&p.queue, &PriorityJob{
        Priority: priority,
        Job:      job,
    })
    
    p.cond.Signal()
}

func (p *PriorityPool) Close() {
    p.mu.Lock()
    p.closed = true
    p.mu.Unlock()
    p.cond.Broadcast()
}
```

## Fiber-এ Image Processing Pool

```go
package main

import (
    "fmt"
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/google/uuid"
)

type ImageJob struct {
    ID       string
    Filename string
    Status   string
    Result   string
}

var (
    imageJobs    = make(chan *ImageJob, 50)
    imageResults = sync.Map{}
)

func imageProcessor(id int) {
    for job := range imageJobs {
        fmt.Printf("Processor %d: %s প্রসেসিং\n", id, job.Filename)
        
        job.Status = "processing"
        imageResults.Store(job.ID, job)
        
        // সিমুলেটেড ইমেজ প্রসেসিং
        time.Sleep(3 * time.Second)
        
        job.Status = "completed"
        job.Result = fmt.Sprintf("processed_%s", job.Filename)
        imageResults.Store(job.ID, job)
        
        fmt.Printf("Processor %d: %s সম্পন্ন\n", id, job.Filename)
    }
}

func main() {
    // ইমেজ প্রসেসর pool
    for i := 1; i <= 3; i++ {
        go imageProcessor(i)
    }

    app := fiber.New()

    app.Post("/images/process", func(c fiber.Ctx) error {
        filename := c.FormValue("filename")
        if filename == "" {
            return c.Status(400).JSON(fiber.Map{"error": "Filename required"})
        }
        
        job := &ImageJob{
            ID:       uuid.New().String(),
            Filename: filename,
            Status:   "queued",
        }
        
        select {
        case imageJobs <- job:
            imageResults.Store(job.ID, job)
            return c.Status(202).JSON(fiber.Map{
                "job_id": job.ID,
                "status": "queued",
            })
        default:
            return c.Status(503).JSON(fiber.Map{
                "error": "Processing queue full",
            })
        }
    })

    app.Get("/images/status/:id", func(c fiber.Ctx) error {
        jobID := c.Params("id")
        
        if job, ok := imageResults.Load(jobID); ok {
            return c.JSON(job)
        }
        
        return c.Status(404).JSON(fiber.Map{"error": "Job not found"})
    })

    app.Listen(":3000")
}
```

## Rate-Limited Worker Pool

```go
package main

import (
    "sync"
    "time"
)

type RateLimitedPool struct {
    jobs    chan func()
    limiter <-chan time.Time
    wg      sync.WaitGroup
}

func NewRateLimitedPool(workers int, ratePerSecond int) *RateLimitedPool {
    p := &RateLimitedPool{
        jobs:    make(chan func(), 100),
        limiter: time.Tick(time.Second / time.Duration(ratePerSecond)),
    }
    
    for i := 0; i < workers; i++ {
        p.wg.Add(1)
        go p.worker()
    }
    
    return p
}

func (p *RateLimitedPool) worker() {
    defer p.wg.Done()
    
    for job := range p.jobs {
        <-p.limiter // Rate limit
        job()
    }
}

func (p *RateLimitedPool) Submit(job func()) {
    p.jobs <- job
}

func (p *RateLimitedPool) Close() {
    close(p.jobs)
    p.wg.Wait()
}
```

## Worker Pool Best Practices

```go
// ✅ সঠিক প্যাটার্ন
type Pool struct {
    jobs    chan Job
    results chan Result
    wg      sync.WaitGroup
    done    chan struct{}
}

func NewPool(workers, bufferSize int) *Pool {
    p := &Pool{
        jobs:    make(chan Job, bufferSize),
        results: make(chan Result, bufferSize),
        done:    make(chan struct{}),
    }
    
    for i := 0; i < workers; i++ {
        p.wg.Add(1)
        go p.worker(i)
    }
    
    return p
}

func (p *Pool) worker(id int) {
    defer p.wg.Done()
    
    for {
        select {
        case job, ok := <-p.jobs:
            if !ok {
                return
            }
            p.results <- p.process(job)
            
        case <-p.done:
            return
        }
    }
}

func (p *Pool) Submit(job Job) {
    p.jobs <- job
}

func (p *Pool) Results() <-chan Result {
    return p.results
}

func (p *Pool) Close() {
    close(p.done)
    close(p.jobs)
    p.wg.Wait()
    close(p.results)
}
```

## সারসংক্ষেপ

| প্যাটার্ন | ব্যবহার |
|---------|--------|
| Fixed Pool | নির্দিষ্ট লোড |
| Dynamic Pool | পরিবর্তনশীল লোড |
| Priority Pool | বিভিন্ন গুরুত্বের কাজ |
| Rate-Limited | API rate limiting |

---

[← আগের: WaitGroups](03-waitgroups.md) | [পরবর্তী: স্কেডিউলার বেসিক →](../08-scheduler/01-scheduler-basics.md)
