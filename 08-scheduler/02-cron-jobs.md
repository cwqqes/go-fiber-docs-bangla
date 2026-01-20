# ক্রন জবস (Cron Jobs)

## Cron কি?

Cron হলো একটি সময়-ভিত্তিক জব স্কেডিউলার। Cron expression দিয়ে নির্দিষ্ট সময়ে কাজ চালানো যায়।

```
┌───────────── মিনিট (0 - 59)
│ ┌───────────── ঘণ্টা (0 - 23)
│ │ ┌───────────── দিন (1 - 31)
│ │ │ ┌───────────── মাস (1 - 12)
│ │ │ │ ┌───────────── সপ্তাহের দিন (0 - 6) (0 = রবিবার)
│ │ │ │ │
* * * * *
```

## Cron Expression Examples

| Expression | বর্ণনা |
|-----------|--------|
| `* * * * *` | প্রতি মিনিটে |
| `0 * * * *` | প্রতি ঘণ্টায় |
| `0 0 * * *` | প্রতিদিন মধ্যরাতে |
| `0 0 * * 0` | প্রতি রবিবার মধ্যরাতে |
| `0 9 * * 1-5` | সোম-শুক্র সকাল ৯টায় |
| `*/5 * * * *` | প্রতি ৫ মিনিটে |
| `0 0 1 * *` | প্রতি মাসের ১ তারিখে |

## robfig/cron Package

```bash
go get github.com/robfig/cron/v3
```

### বেসিক ব্যবহার

```go
package main

import (
    "log"
    "github.com/robfig/cron/v3"
)

func main() {
    c := cron.New()
    
    // প্রতি মিনিটে
    c.AddFunc("* * * * *", func() {
        log.Println("প্রতি মিনিটে চলছে")
    })
    
    // প্রতি ঘণ্টায়
    c.AddFunc("0 * * * *", func() {
        log.Println("প্রতি ঘণ্টায় চলছে")
    })
    
    // প্রতিদিন সকাল ৯টায়
    c.AddFunc("0 9 * * *", func() {
        log.Println("সকাল ৯টার জব")
    })
    
    c.Start()
    
    // ব্লক করুন
    select {}
}
```

### With Seconds

```go
package main

import (
    "log"
    "github.com/robfig/cron/v3"
)

func main() {
    // সেকেন্ড সাপোর্ট সহ
    c := cron.New(cron.WithSeconds())
    
    // প্রতি ৫ সেকেন্ডে
    c.AddFunc("*/5 * * * * *", func() {
        log.Println("প্রতি ৫ সেকেন্ডে")
    })
    
    c.Start()
    select {}
}
```

## Fiber-এ Cron Integration

```go
package main

import (
    "log"
    "sync"
    "github.com/gofiber/fiber/v3"
    "github.com/robfig/cron/v3"
)

type CronManager struct {
    cron *cron.Cron
    jobs map[string]cron.EntryID
    mu   sync.RWMutex
}

func NewCronManager() *CronManager {
    return &CronManager{
        cron: cron.New(cron.WithSeconds()),
        jobs: make(map[string]cron.EntryID),
    }
}

func (cm *CronManager) AddJob(name, schedule string, job func()) error {
    cm.mu.Lock()
    defer cm.mu.Unlock()
    
    entryID, err := cm.cron.AddFunc(schedule, job)
    if err != nil {
        return err
    }
    
    cm.jobs[name] = entryID
    log.Printf("Cron job '%s' যোগ করা হয়েছে: %s\n", name, schedule)
    return nil
}

func (cm *CronManager) RemoveJob(name string) bool {
    cm.mu.Lock()
    defer cm.mu.Unlock()
    
    if entryID, ok := cm.jobs[name]; ok {
        cm.cron.Remove(entryID)
        delete(cm.jobs, name)
        log.Printf("Cron job '%s' সরানো হয়েছে\n", name)
        return true
    }
    return false
}

func (cm *CronManager) Start() {
    cm.cron.Start()
}

func (cm *CronManager) Stop() {
    cm.cron.Stop()
}

func (cm *CronManager) ListJobs() []map[string]interface{} {
    cm.mu.RLock()
    defer cm.mu.RUnlock()
    
    jobs := make([]map[string]interface{}, 0)
    for name, entryID := range cm.jobs {
        entry := cm.cron.Entry(entryID)
        jobs = append(jobs, map[string]interface{}{
            "name":     name,
            "next_run": entry.Next,
            "prev_run": entry.Prev,
        })
    }
    return jobs
}

func main() {
    cm := NewCronManager()
    
    // নিয়মিত জবস যোগ করুন
    cm.AddJob("cleanup", "0 0 * * * *", func() {
        log.Println("ক্লিনআপ চলছে...")
    })
    
    cm.AddJob("report", "0 0 9 * * 1-5", func() {
        log.Println("দৈনিক রিপোর্ট তৈরি হচ্ছে...")
    })
    
    cm.AddJob("health-check", "*/30 * * * * *", func() {
        log.Println("হেলথ চেক চলছে...")
    })
    
    cm.Start()

    app := fiber.New()

    // সব জব দেখুন
    app.Get("/cron/jobs", func(c fiber.Ctx) error {
        return c.JSON(cm.ListJobs())
    })

    // নতুন জব যোগ করুন
    app.Post("/cron/jobs", func(c fiber.Ctx) error {
        var req struct {
            Name     string `json:"name"`
            Schedule string `json:"schedule"`
        }
        
        if err := c.Bind().JSON(&req); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
        }
        
        err := cm.AddJob(req.Name, req.Schedule, func() {
            log.Printf("Custom job '%s' চলছে...\n", req.Name)
        })
        
        if err != nil {
            return c.Status(400).JSON(fiber.Map{"error": err.Error()})
        }
        
        return c.JSON(fiber.Map{"message": "Job added"})
    })

    // জব সরান
    app.Delete("/cron/jobs/:name", func(c fiber.Ctx) error {
        name := c.Params("name")
        
        if cm.RemoveJob(name) {
            return c.JSON(fiber.Map{"message": "Job removed"})
        }
        
        return c.Status(404).JSON(fiber.Map{"error": "Job not found"})
    })

    log.Fatal(app.Listen(":3000"))
}
```

## Database Cleanup Job

```go
package main

import (
    "database/sql"
    "log"
    "time"
    "github.com/robfig/cron/v3"
)

func setupCleanupJobs(db *sql.DB) *cron.Cron {
    c := cron.New()
    
    // প্রতি ঘণ্টায় expired sessions ক্লিনআপ
    c.AddFunc("0 * * * *", func() {
        result, err := db.Exec(`
            DELETE FROM sessions 
            WHERE expires_at < NOW()
        `)
        if err != nil {
            log.Println("Session cleanup error:", err)
            return
        }
        
        affected, _ := result.RowsAffected()
        log.Printf("Cleaned up %d expired sessions\n", affected)
    })
    
    // প্রতিদিন মধ্যরাতে পুরাতন logs ক্লিনআপ
    c.AddFunc("0 0 * * *", func() {
        cutoff := time.Now().AddDate(0, 0, -30) // ৩০ দিনের পুরাতন
        
        result, err := db.Exec(`
            DELETE FROM logs 
            WHERE created_at < $1
        `, cutoff)
        if err != nil {
            log.Println("Log cleanup error:", err)
            return
        }
        
        affected, _ := result.RowsAffected()
        log.Printf("Cleaned up %d old logs\n", affected)
    })
    
    // প্রতি সপ্তাহে backup
    c.AddFunc("0 0 * * 0", func() {
        log.Println("Weekly backup শুরু...")
        // backup লজিক
    })
    
    return c
}
```

## Email Scheduler

```go
package main

import (
    "log"
    "sync"
    "time"
    "github.com/robfig/cron/v3"
)

type EmailJob struct {
    To      string
    Subject string
    Body    string
    SendAt  time.Time
}

type EmailScheduler struct {
    cron     *cron.Cron
    pending  []EmailJob
    mu       sync.Mutex
}

func NewEmailScheduler() *EmailScheduler {
    es := &EmailScheduler{
        cron:    cron.New(),
        pending: make([]EmailJob, 0),
    }
    
    // প্রতি মিনিটে pending emails চেক করুন
    es.cron.AddFunc("* * * * *", es.processPending)
    
    return es
}

func (es *EmailScheduler) Schedule(job EmailJob) {
    es.mu.Lock()
    defer es.mu.Unlock()
    
    es.pending = append(es.pending, job)
    log.Printf("Email scheduled for %s at %v\n", job.To, job.SendAt)
}

func (es *EmailScheduler) processPending() {
    es.mu.Lock()
    defer es.mu.Unlock()
    
    now := time.Now()
    remaining := make([]EmailJob, 0)
    
    for _, job := range es.pending {
        if now.After(job.SendAt) || now.Equal(job.SendAt) {
            // ইমেইল পাঠান
            log.Printf("Sending email to %s: %s\n", job.To, job.Subject)
            es.sendEmail(job)
        } else {
            remaining = append(remaining, job)
        }
    }
    
    es.pending = remaining
}

func (es *EmailScheduler) sendEmail(job EmailJob) {
    // ইমেইল পাঠানোর লজিক
}

func (es *EmailScheduler) Start() {
    es.cron.Start()
}

func (es *EmailScheduler) Stop() {
    es.cron.Stop()
}
```

## Distributed Cron (with Lock)

```go
package main

import (
    "context"
    "log"
    "time"
    "github.com/go-redis/redis/v8"
    "github.com/robfig/cron/v3"
)

type DistributedCron struct {
    cron   *cron.Cron
    redis  *redis.Client
    nodeID string
}

func NewDistributedCron(redisClient *redis.Client, nodeID string) *DistributedCron {
    return &DistributedCron{
        cron:   cron.New(),
        redis:  redisClient,
        nodeID: nodeID,
    }
}

func (dc *DistributedCron) AddJob(name, schedule string, job func()) {
    dc.cron.AddFunc(schedule, func() {
        ctx := context.Background()
        lockKey := "cron_lock:" + name
        
        // Lock অর্জন করার চেষ্টা করুন
        ok, err := dc.redis.SetNX(ctx, lockKey, dc.nodeID, 5*time.Minute).Result()
        if err != nil || !ok {
            log.Printf("Node %s: Could not acquire lock for %s\n", dc.nodeID, name)
            return
        }
        
        defer dc.redis.Del(ctx, lockKey)
        
        log.Printf("Node %s: Running job %s\n", dc.nodeID, name)
        job()
    })
}

func (dc *DistributedCron) Start() {
    dc.cron.Start()
}
```

## Fiber Integration with Monitoring

```go
package main

import (
    "log"
    "sync/atomic"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/robfig/cron/v3"
)

type JobStats struct {
    Name       string
    RunCount   int64
    LastRun    time.Time
    LastError  string
    AvgRunTime time.Duration
}

type MonitoredCron struct {
    cron  *cron.Cron
    stats map[string]*JobStats
}

func NewMonitoredCron() *MonitoredCron {
    return &MonitoredCron{
        cron:  cron.New(cron.WithSeconds()),
        stats: make(map[string]*JobStats),
    }
}

func (mc *MonitoredCron) AddJob(name, schedule string, job func() error) error {
    mc.stats[name] = &JobStats{Name: name}
    
    _, err := mc.cron.AddFunc(schedule, func() {
        stats := mc.stats[name]
        start := time.Now()
        
        atomic.AddInt64(&stats.RunCount, 1)
        
        if err := job(); err != nil {
            stats.LastError = err.Error()
            log.Printf("Job '%s' error: %v\n", name, err)
        } else {
            stats.LastError = ""
        }
        
        stats.LastRun = time.Now()
        duration := time.Since(start)
        
        // রানিং অ্যাভারেজ
        if stats.AvgRunTime == 0 {
            stats.AvgRunTime = duration
        } else {
            stats.AvgRunTime = (stats.AvgRunTime + duration) / 2
        }
    })
    
    return err
}

func (mc *MonitoredCron) GetStats() map[string]*JobStats {
    return mc.stats
}

func (mc *MonitoredCron) Start() {
    mc.cron.Start()
}

func main() {
    mc := NewMonitoredCron()
    
    mc.AddJob("process-orders", "*/10 * * * * *", func() error {
        log.Println("Orders প্রসেস হচ্ছে...")
        time.Sleep(2 * time.Second)
        return nil
    })
    
    mc.AddJob("send-notifications", "0 * * * * *", func() error {
        log.Println("Notifications পাঠানো হচ্ছে...")
        return nil
    })
    
    mc.Start()

    app := fiber.New()

    app.Get("/cron/stats", func(c fiber.Ctx) error {
        stats := mc.GetStats()
        
        result := make([]map[string]interface{}, 0)
        for name, s := range stats {
            result = append(result, map[string]interface{}{
                "name":         name,
                "run_count":    s.RunCount,
                "last_run":     s.LastRun,
                "last_error":   s.LastError,
                "avg_run_time": s.AvgRunTime.String(),
            })
        }
        
        return c.JSON(result)
    })

    log.Fatal(app.Listen(":3000"))
}
```

## সারসংক্ষেপ

| Feature | Description |
|---------|-------------|
| `cron.New()` | নতুন cron scheduler |
| `cron.WithSeconds()` | সেকেন্ড সাপোর্ট |
| `AddFunc()` | জব যোগ করুন |
| `Remove()` | জব সরান |
| `Start()` | স্কেডিউলার শুরু |
| `Stop()` | স্কেডিউলার বন্ধ |

---

[← আগের: স্কেডিউলার বেসিক](01-scheduler-basics.md) | [পরবর্তী: ব্যাকগ্রাউন্ড টাস্ক →](03-background-tasks.md)
