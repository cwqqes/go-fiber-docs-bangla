# ব্যাকগ্রাউন্ড টাস্ক (Background Tasks)

## ব্যাকগ্রাউন্ড টাস্ক কি?

ব্যাকগ্রাউন্ড টাস্ক হলো এমন কাজ যা মূল রিকোয়েস্ট-রেসপন্স সাইকেলের বাইরে অ্যাসিঙ্ক্রোনাসভাবে চালানো হয়।

## সিম্পল Background Task

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Post("/send-email", func(c fiber.Ctx) error {
        var req struct {
            To      string `json:"to"`
            Subject string `json:"subject"`
            Body    string `json:"body"`
        }
        
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
        }
        
        // ব্যাকগ্রাউন্ডে ইমেইল পাঠান
        go func(to, subject, body string) {
            log.Printf("Sending email to %s...\n", to)
            time.Sleep(5 * time.Second) // সিমুলেটেড ইমেইল
            log.Printf("Email sent to %s\n", to)
        }(req.To, req.Subject, req.Body)
        
        // অবিলম্বে রেসপন্স
        return c.JSON(fiber.Map{
            "message": "Email queued for sending",
        })
    })

    app.Listen(":3000")
}
```

## Task Manager

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

type TaskStatus string

const (
    StatusPending    TaskStatus = "pending"
    StatusRunning    TaskStatus = "running"
    StatusCompleted  TaskStatus = "completed"
    StatusFailed     TaskStatus = "failed"
    StatusCanceled   TaskStatus = "canceled"
)

type Task struct {
    ID        string        `json:"id"`
    Type      string        `json:"type"`
    Status    TaskStatus    `json:"status"`
    Progress  int           `json:"progress"`
    Result    interface{}   `json:"result,omitempty"`
    Error     string        `json:"error,omitempty"`
    CreatedAt time.Time     `json:"created_at"`
    StartedAt *time.Time    `json:"started_at,omitempty"`
    EndedAt   *time.Time    `json:"ended_at,omitempty"`
    cancel    context.CancelFunc
}

type TaskManager struct {
    tasks map[string]*Task
    mu    sync.RWMutex
    queue chan *Task
}

func NewTaskManager(workers int) *TaskManager {
    tm := &TaskManager{
        tasks: make(map[string]*Task),
        queue: make(chan *Task, 100),
    }
    
    // Workers শুরু করুন
    for i := 0; i < workers; i++ {
        go tm.worker(i)
    }
    
    return tm
}

func (tm *TaskManager) worker(id int) {
    for task := range tm.queue {
        log.Printf("Worker %d: Task %s শুরু\n", id, task.ID)
        tm.runTask(task)
    }
}

func (tm *TaskManager) runTask(task *Task) {
    ctx, cancel := context.WithCancel(context.Background())
    task.cancel = cancel
    
    tm.mu.Lock()
    task.Status = StatusRunning
    now := time.Now()
    task.StartedAt = &now
    tm.mu.Unlock()
    
    // টাস্ক টাইপ অনুযায়ী চালান
    var err error
    switch task.Type {
    case "email":
        err = tm.processEmail(ctx, task)
    case "export":
        err = tm.processExport(ctx, task)
    case "import":
        err = tm.processImport(ctx, task)
    default:
        err = tm.processGeneric(ctx, task)
    }
    
    tm.mu.Lock()
    endTime := time.Now()
    task.EndedAt = &endTime
    
    if ctx.Err() == context.Canceled {
        task.Status = StatusCanceled
    } else if err != nil {
        task.Status = StatusFailed
        task.Error = err.Error()
    } else {
        task.Status = StatusCompleted
        task.Progress = 100
    }
    tm.mu.Unlock()
}

func (tm *TaskManager) processEmail(ctx context.Context, task *Task) error {
    for i := 1; i <= 10; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            time.Sleep(500 * time.Millisecond)
            tm.updateProgress(task.ID, i*10)
        }
    }
    task.Result = "Email sent successfully"
    return nil
}

func (tm *TaskManager) processExport(ctx context.Context, task *Task) error {
    for i := 1; i <= 20; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            time.Sleep(200 * time.Millisecond)
            tm.updateProgress(task.ID, i*5)
        }
    }
    task.Result = map[string]string{"file": "export_12345.csv"}
    return nil
}

func (tm *TaskManager) processImport(ctx context.Context, task *Task) error {
    for i := 1; i <= 50; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            time.Sleep(100 * time.Millisecond)
            tm.updateProgress(task.ID, i*2)
        }
    }
    task.Result = map[string]int{"imported": 1000}
    return nil
}

func (tm *TaskManager) processGeneric(ctx context.Context, task *Task) error {
    time.Sleep(3 * time.Second)
    task.Result = "Done"
    return nil
}

func (tm *TaskManager) updateProgress(taskID string, progress int) {
    tm.mu.Lock()
    defer tm.mu.Unlock()
    
    if task, ok := tm.tasks[taskID]; ok {
        task.Progress = progress
    }
}

func (tm *TaskManager) CreateTask(taskType string) *Task {
    task := &Task{
        ID:        uuid.New().String(),
        Type:      taskType,
        Status:    StatusPending,
        Progress:  0,
        CreatedAt: time.Now(),
    }
    
    tm.mu.Lock()
    tm.tasks[task.ID] = task
    tm.mu.Unlock()
    
    tm.queue <- task
    return task
}

func (tm *TaskManager) GetTask(id string) (*Task, bool) {
    tm.mu.RLock()
    defer tm.mu.RUnlock()
    
    task, ok := tm.tasks[id]
    return task, ok
}

func (tm *TaskManager) CancelTask(id string) bool {
    tm.mu.Lock()
    defer tm.mu.Unlock()
    
    if task, ok := tm.tasks[id]; ok && task.Status == StatusRunning {
        if task.cancel != nil {
            task.cancel()
            return true
        }
    }
    return false
}

func (tm *TaskManager) ListTasks() []*Task {
    tm.mu.RLock()
    defer tm.mu.RUnlock()
    
    tasks := make([]*Task, 0, len(tm.tasks))
    for _, task := range tm.tasks {
        tasks = append(tasks, task)
    }
    return tasks
}

func main() {
    tm := NewTaskManager(5)
    app := fiber.New()

    // নতুন টাস্ক তৈরি
    app.Post("/tasks", func(c fiber.Ctx) error {
        var req struct {
            Type string `json:"type"`
        }
        
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
        }
        
        task := tm.CreateTask(req.Type)
        
        return c.Status(202).JSON(fiber.Map{
            "task_id": task.ID,
            "status":  task.Status,
        })
    })

    // টাস্ক স্ট্যাটাস
    app.Get("/tasks/:id", func(c fiber.Ctx) error {
        task, ok := tm.GetTask(c.Params("id"))
        if !ok {
            return c.Status(404).JSON(fiber.Map{"error": "Task not found"})
        }
        return c.JSON(task)
    })

    // টাস্ক বাতিল
    app.Delete("/tasks/:id", func(c fiber.Ctx) error {
        if tm.CancelTask(c.Params("id")) {
            return c.JSON(fiber.Map{"message": "Task canceled"})
        }
        return c.Status(400).JSON(fiber.Map{"error": "Cannot cancel task"})
    })

    // সব টাস্ক
    app.Get("/tasks", func(c fiber.Ctx) error {
        return c.JSON(tm.ListTasks())
    })

    log.Fatal(app.Listen(":3000"))
}
```

## Long-Running Task with SSE

```go
package main

import (
    "bufio"
    "fmt"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/google/uuid"
)

func main() {
    app := fiber.New()

    app.Get("/tasks/:id/stream", func(c fiber.Ctx) error {
        taskID := c.Params("id")
        
        c.Set("Content-Type", "text/event-stream")
        c.Set("Cache-Control", "no-cache")
        c.Set("Connection", "keep-alive")
        
        c.Context().SetBodyStreamWriter(func(w *bufio.Writer) {
            for progress := 0; progress <= 100; progress += 10 {
                fmt.Fprintf(w, "data: {\"task_id\":\"%s\",\"progress\":%d}\n\n", taskID, progress)
                w.Flush()
                time.Sleep(500 * time.Millisecond)
            }
            
            fmt.Fprintf(w, "data: {\"task_id\":\"%s\",\"status\":\"completed\"}\n\n", taskID)
            w.Flush()
        })
        
        return nil
    })

    app.Listen(":3000")
}
```

## Webhook Notification

```go
package main

import (
    "bytes"
    "encoding/json"
    "log"
    "net/http"
    "time"
)

type TaskNotifier struct {
    webhookURL string
    client     *http.Client
}

func NewTaskNotifier(webhookURL string) *TaskNotifier {
    return &TaskNotifier{
        webhookURL: webhookURL,
        client: &http.Client{
            Timeout: 10 * time.Second,
        },
    }
}

func (tn *TaskNotifier) NotifyCompletion(taskID, status string, result interface{}) {
    go func() {
        payload := map[string]interface{}{
            "task_id":      taskID,
            "status":       status,
            "result":       result,
            "completed_at": time.Now(),
        }
        
        body, err := json.Marshal(payload)
        if err != nil {
            log.Printf("Webhook marshal error: %v\n", err)
            return
        }
        
        resp, err := tn.client.Post(tn.webhookURL, "application/json", bytes.NewBuffer(body))
        if err != nil {
            log.Printf("Webhook error: %v\n", err)
            return
        }
        defer resp.Body.Close()
        
        log.Printf("Webhook sent for task %s, status: %d\n", taskID, resp.StatusCode)
    }()
}
```

## Retry Background Task

```go
package main

import (
    "context"
    "errors"
    "log"
    "time"
)

type RetryableTask struct {
    MaxRetries int
    Delay      time.Duration
    Task       func(ctx context.Context) error
}

func (rt *RetryableTask) Run(ctx context.Context) error {
    var lastErr error
    
    for attempt := 0; attempt <= rt.MaxRetries; attempt++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }
        
        err := rt.Task(ctx)
        if err == nil {
            return nil
        }
        
        lastErr = err
        log.Printf("Attempt %d failed: %v\n", attempt+1, err)
        
        if attempt < rt.MaxRetries {
            delay := rt.Delay * time.Duration(1<<attempt) // exponential backoff
            time.Sleep(delay)
        }
    }
    
    return lastErr
}

// ব্যবহার
func main() {
    task := &RetryableTask{
        MaxRetries: 3,
        Delay:      time.Second,
        Task: func(ctx context.Context) error {
            // সিমুলেটেড ব্যর্থতা
            return errors.New("temporary failure")
        },
    }
    
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := task.Run(ctx); err != nil {
        log.Println("Task finally failed:", err)
    }
}
```

## Batch Processing

```go
package main

import (
    "log"
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
)

type BatchProcessor struct {
    items     []interface{}
    mu        sync.Mutex
    batchSize int
    interval  time.Duration
    process   func([]interface{}) error
    done      chan struct{}
}

func NewBatchProcessor(batchSize int, interval time.Duration, processor func([]interface{}) error) *BatchProcessor {
    bp := &BatchProcessor{
        items:     make([]interface{}, 0),
        batchSize: batchSize,
        interval:  interval,
        process:   processor,
        done:      make(chan struct{}),
    }
    
    go bp.run()
    return bp
}

func (bp *BatchProcessor) Add(item interface{}) {
    bp.mu.Lock()
    bp.items = append(bp.items, item)
    
    if len(bp.items) >= bp.batchSize {
        items := bp.items
        bp.items = make([]interface{}, 0)
        bp.mu.Unlock()
        
        go bp.process(items)
        return
    }
    
    bp.mu.Unlock()
}

func (bp *BatchProcessor) run() {
    ticker := time.NewTicker(bp.interval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            bp.flush()
        case <-bp.done:
            bp.flush()
            return
        }
    }
}

func (bp *BatchProcessor) flush() {
    bp.mu.Lock()
    if len(bp.items) == 0 {
        bp.mu.Unlock()
        return
    }
    
    items := bp.items
    bp.items = make([]interface{}, 0)
    bp.mu.Unlock()
    
    go bp.process(items)
}

func (bp *BatchProcessor) Stop() {
    close(bp.done)
}

func main() {
    processor := NewBatchProcessor(10, 5*time.Second, func(items []interface{}) error {
        log.Printf("Processing batch of %d items\n", len(items))
        // batch প্রসেস করুন
        return nil
    })
    defer processor.Stop()

    app := fiber.New()

    app.Post("/events", func(c fiber.Ctx) error {
        var event map[string]interface{}
        if err := c.Bind().JSON(&event); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid event"})
        }
        
        processor.Add(event)
        
        return c.JSON(fiber.Map{"message": "Event queued"})
    })

    app.Listen(":3000")
}
```

## সারসংক্ষেপ

| প্যাটার্ন | ব্যবহার |
|---------|--------|
| Simple `go func()` | সিম্পল fire-and-forget |
| Task Manager | স্ট্যাটাস ট্র্যাকিং সহ |
| Worker Pool | সীমিত concurrency |
| SSE | রিয়েল-টাইম আপডেট |
| Batch Processing | বাল্ক অপারেশন |
| Retry | অস্থায়ী ত্রুটি হ্যান্ডলিং |

---

[← আগের: ক্রন জবস](02-cron-jobs.md) | [পরবর্তী: টাস্ক কিউ →](04-task-queues.md)
