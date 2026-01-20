# ডেটাবেস (Database)

## Database/SQL

Go-এর স্ট্যান্ডার্ড লাইব্রেরি `database/sql` ব্যবহার করে।

```bash
# PostgreSQL
go get github.com/lib/pq

# MySQL
go get github.com/go-sql-driver/mysql

# SQLite
go get github.com/mattn/go-sqlite3
```

## বেসিক Connection

```go
package main

import (
    "database/sql"
    "log"
    "github.com/gofiber/fiber/v3"
    _ "github.com/lib/pq"
)

var db *sql.DB

func initDB() {
    var err error
    db, err = sql.Open("postgres", "postgres://user:pass@localhost/dbname?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    
    // Connection pool settings
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(5)
    db.SetConnMaxLifetime(5 * time.Minute)
    
    // Test connection
    if err = db.Ping(); err != nil {
        log.Fatal(err)
    }
    
    log.Println("Database connected!")
}

func main() {
    initDB()
    defer db.Close()

    app := fiber.New()

    app.Get("/users", func(c fiber.Ctx) error {
        rows, err := db.Query("SELECT id, name, email FROM users")
        if err != nil {
            return c.Status(500).JSON(fiber.Map{"error": err.Error()})
        }
        defer rows.Close()
        
        var users []map[string]interface{}
        for rows.Next() {
            var id int
            var name, email string
            if err := rows.Scan(&id, &name, &email); err != nil {
                continue
            }
            users = append(users, map[string]interface{}{
                "id": id, "name": name, "email": email,
            })
        }
        
        return c.JSON(users)
    })

    app.Listen(":3000")
}
```

## GORM ORM

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/postgres
```

### Setup

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
)

var DB *gorm.DB

type User struct {
    ID        uint           `gorm:"primaryKey" json:"id"`
    Name      string         `gorm:"size:100;not null" json:"name"`
    Email     string         `gorm:"size:100;uniqueIndex;not null" json:"email"`
    Password  string         `gorm:"size:255;not null" json:"-"`
    IsActive  bool           `gorm:"default:true" json:"is_active"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`
}

func initDB() {
    dsn := "host=localhost user=postgres password=pass dbname=mydb port=5432"
    
    var err error
    DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
    })
    if err != nil {
        log.Fatal("Database connection failed:", err)
    }
    
    // Auto migrate
    DB.AutoMigrate(&User{})
    
    log.Println("Database connected and migrated!")
}

func main() {
    initDB()

    app := fiber.New()

    // CRUD routes
    app.Post("/users", createUser)
    app.Get("/users", getUsers)
    app.Get("/users/:id", getUser)
    app.Put("/users/:id", updateUser)
    app.Delete("/users/:id", deleteUser)

    app.Listen(":3000")
}
```

### CRUD Operations

```go
// Create
func createUser(c fiber.Ctx) error {
    user := new(User)
    
    if err := c.Bind().JSON(user); err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
    }
    
    // Password hash (use bcrypt in production)
    user.Password = hashPassword(user.Password)
    
    if err := DB.Create(user).Error; err != nil {
        return c.Status(500).JSON(fiber.Map{"error": err.Error()})
    }
    
    return c.Status(201).JSON(user)
}

// Read All
func getUsers(c fiber.Ctx) error {
    var users []User
    
    // Pagination
    page := c.QueryInt("page", 1)
    limit := c.QueryInt("limit", 10)
    offset := (page - 1) * limit
    
    result := DB.Offset(offset).Limit(limit).Find(&users)
    if result.Error != nil {
        return c.Status(500).JSON(fiber.Map{"error": result.Error.Error()})
    }
    
    var total int64
    DB.Model(&User{}).Count(&total)
    
    return c.JSON(fiber.Map{
        "users": users,
        "total": total,
        "page":  page,
        "limit": limit,
    })
}

// Read One
func getUser(c fiber.Ctx) error {
    id := c.Params("id")
    var user User
    
    if err := DB.First(&user, id).Error; err != nil {
        return c.Status(404).JSON(fiber.Map{"error": "User not found"})
    }
    
    return c.JSON(user)
}

// Update
func updateUser(c fiber.Ctx) error {
    id := c.Params("id")
    var user User
    
    if err := DB.First(&user, id).Error; err != nil {
        return c.Status(404).JSON(fiber.Map{"error": "User not found"})
    }
    
    var updates map[string]interface{}
    if err := c.Bind().JSON(&updates); err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
    }
    
    // Sensitive fields বাদ দিন
    delete(updates, "password")
    delete(updates, "id")
    
    if err := DB.Model(&user).Updates(updates).Error; err != nil {
        return c.Status(500).JSON(fiber.Map{"error": err.Error()})
    }
    
    return c.JSON(user)
}

// Delete (Soft)
func deleteUser(c fiber.Ctx) error {
    id := c.Params("id")
    
    result := DB.Delete(&User{}, id)
    if result.Error != nil {
        return c.Status(500).JSON(fiber.Map{"error": result.Error.Error()})
    }
    
    if result.RowsAffected == 0 {
        return c.Status(404).JSON(fiber.Map{"error": "User not found"})
    }
    
    return c.JSON(fiber.Map{"message": "User deleted"})
}
```

## Relationships

```go
// Models
type User struct {
    ID       uint      `gorm:"primaryKey" json:"id"`
    Name     string    `json:"name"`
    Email    string    `json:"email"`
    Posts    []Post    `json:"posts,omitempty"`
    Profile  *Profile  `json:"profile,omitempty"`
}

type Profile struct {
    ID     uint   `gorm:"primaryKey" json:"id"`
    UserID uint   `gorm:"uniqueIndex" json:"user_id"`
    Bio    string `json:"bio"`
    Avatar string `json:"avatar"`
}

type Post struct {
    ID        uint      `gorm:"primaryKey" json:"id"`
    UserID    uint      `json:"user_id"`
    Title     string    `json:"title"`
    Content   string    `json:"content"`
    Tags      []Tag     `gorm:"many2many:post_tags;" json:"tags,omitempty"`
    Comments  []Comment `json:"comments,omitempty"`
    CreatedAt time.Time `json:"created_at"`
}

type Tag struct {
    ID    uint   `gorm:"primaryKey" json:"id"`
    Name  string `gorm:"uniqueIndex" json:"name"`
    Posts []Post `gorm:"many2many:post_tags;" json:"-"`
}

type Comment struct {
    ID        uint      `gorm:"primaryKey" json:"id"`
    PostID    uint      `json:"post_id"`
    UserID    uint      `json:"user_id"`
    Content   string    `json:"content"`
    User      User      `json:"user,omitempty"`
    CreatedAt time.Time `json:"created_at"`
}

// Preload সহ Query
func getPostWithDetails(c fiber.Ctx) error {
    id := c.Params("id")
    var post Post
    
    err := DB.Preload("Tags").
        Preload("Comments").
        Preload("Comments.User").
        First(&post, id).Error
    
    if err != nil {
        return c.Status(404).JSON(fiber.Map{"error": "Post not found"})
    }
    
    return c.JSON(post)
}

func getUserWithPosts(c fiber.Ctx) error {
    id := c.Params("id")
    var user User
    
    err := DB.Preload("Profile").
        Preload("Posts", func(db *gorm.DB) *gorm.DB {
            return db.Order("posts.created_at DESC").Limit(5)
        }).
        First(&user, id).Error
    
    if err != nil {
        return c.Status(404).JSON(fiber.Map{"error": "User not found"})
    }
    
    return c.JSON(user)
}
```

## Transactions

```go
func transferMoney(c fiber.Ctx) error {
    var req struct {
        FromID uint    `json:"from_id"`
        ToID   uint    `json:"to_id"`
        Amount float64 `json:"amount"`
    }
    
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
    }
    
    // Transaction শুরু করুন
    tx := DB.Begin()
    defer func() {
        if r := recover(); r != nil {
            tx.Rollback()
        }
    }()
    
    // Sender থেকে টাকা কমান
    var sender Account
    if err := tx.First(&sender, req.FromID).Error; err != nil {
        tx.Rollback()
        return c.Status(404).JSON(fiber.Map{"error": "Sender not found"})
    }
    
    if sender.Balance < req.Amount {
        tx.Rollback()
        return c.Status(400).JSON(fiber.Map{"error": "Insufficient balance"})
    }
    
    sender.Balance -= req.Amount
    if err := tx.Save(&sender).Error; err != nil {
        tx.Rollback()
        return c.Status(500).JSON(fiber.Map{"error": err.Error()})
    }
    
    // Receiver এ টাকা যোগ করুন
    var receiver Account
    if err := tx.First(&receiver, req.ToID).Error; err != nil {
        tx.Rollback()
        return c.Status(404).JSON(fiber.Map{"error": "Receiver not found"})
    }
    
    receiver.Balance += req.Amount
    if err := tx.Save(&receiver).Error; err != nil {
        tx.Rollback()
        return c.Status(500).JSON(fiber.Map{"error": err.Error()})
    }
    
    // Transaction log
    transaction := Transaction{
        FromID:    req.FromID,
        ToID:      req.ToID,
        Amount:    req.Amount,
        CreatedAt: time.Now(),
    }
    if err := tx.Create(&transaction).Error; err != nil {
        tx.Rollback()
        return c.Status(500).JSON(fiber.Map{"error": err.Error()})
    }
    
    // Commit
    if err := tx.Commit().Error; err != nil {
        return c.Status(500).JSON(fiber.Map{"error": err.Error()})
    }
    
    return c.JSON(fiber.Map{"message": "Transfer successful"})
}
```

## Query Builder

```go
func searchUsers(c fiber.Ctx) error {
    query := DB.Model(&User{})
    
    // Filters
    if name := c.Query("name"); name != "" {
        query = query.Where("name ILIKE ?", "%"+name+"%")
    }
    
    if email := c.Query("email"); email != "" {
        query = query.Where("email ILIKE ?", "%"+email+"%")
    }
    
    if isActive := c.Query("active"); isActive != "" {
        query = query.Where("is_active = ?", isActive == "true")
    }
    
    // Date range
    if from := c.Query("from"); from != "" {
        query = query.Where("created_at >= ?", from)
    }
    
    if to := c.Query("to"); to != "" {
        query = query.Where("created_at <= ?", to)
    }
    
    // Sorting
    sortBy := c.Query("sort", "created_at")
    sortDir := c.Query("dir", "desc")
    query = query.Order(fmt.Sprintf("%s %s", sortBy, sortDir))
    
    // Pagination
    page := c.QueryInt("page", 1)
    limit := c.QueryInt("limit", 10)
    
    var total int64
    query.Count(&total)
    
    var users []User
    query.Offset((page - 1) * limit).Limit(limit).Find(&users)
    
    return c.JSON(fiber.Map{
        "data":       users,
        "total":      total,
        "page":       page,
        "limit":      limit,
        "total_pages": (total + int64(limit) - 1) / int64(limit),
    })
}
```

## Raw SQL

```go
func getStats(c fiber.Ctx) error {
    type Stats struct {
        TotalUsers     int64 `json:"total_users"`
        ActiveUsers    int64 `json:"active_users"`
        TotalPosts     int64 `json:"total_posts"`
        PostsThisMonth int64 `json:"posts_this_month"`
    }
    
    var stats Stats
    
    DB.Raw(`
        SELECT 
            (SELECT COUNT(*) FROM users WHERE deleted_at IS NULL) as total_users,
            (SELECT COUNT(*) FROM users WHERE is_active = true AND deleted_at IS NULL) as active_users,
            (SELECT COUNT(*) FROM posts WHERE deleted_at IS NULL) as total_posts,
            (SELECT COUNT(*) FROM posts WHERE created_at >= date_trunc('month', CURRENT_DATE) AND deleted_at IS NULL) as posts_this_month
    `).Scan(&stats)
    
    return c.JSON(stats)
}
```

## Database Middleware

```go
func DBMiddleware(db *gorm.DB) fiber.Handler {
    return func(c fiber.Ctx) error {
        c.Locals("db", db)
        return c.Next()
    }
}

// ব্যবহার
app.Use(DBMiddleware(DB))

app.Get("/users", func(c fiber.Ctx) error {
    db := c.Locals("db").(*gorm.DB)
    // db ব্যবহার করুন
})
```

## Repository Pattern

```go
// Repository Interface
type UserRepository interface {
    Create(user *User) error
    FindByID(id uint) (*User, error)
    FindByEmail(email string) (*User, error)
    Update(user *User) error
    Delete(id uint) error
    List(page, limit int) ([]User, int64, error)
}

// Implementation
type userRepository struct {
    db *gorm.DB
}

func NewUserRepository(db *gorm.DB) UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(user *User) error {
    return r.db.Create(user).Error
}

func (r *userRepository) FindByID(id uint) (*User, error) {
    var user User
    err := r.db.First(&user, id).Error
    if err != nil {
        return nil, err
    }
    return &user, nil
}

func (r *userRepository) FindByEmail(email string) (*User, error) {
    var user User
    err := r.db.Where("email = ?", email).First(&user).Error
    if err != nil {
        return nil, err
    }
    return &user, nil
}

func (r *userRepository) Update(user *User) error {
    return r.db.Save(user).Error
}

func (r *userRepository) Delete(id uint) error {
    return r.db.Delete(&User{}, id).Error
}

func (r *userRepository) List(page, limit int) ([]User, int64, error) {
    var users []User
    var total int64
    
    r.db.Model(&User{}).Count(&total)
    err := r.db.Offset((page - 1) * limit).Limit(limit).Find(&users).Error
    
    return users, total, err
}
```

## সারসংক্ষেপ

| Feature | Description |
|---------|-------------|
| `gorm.Open()` | Database connection |
| `AutoMigrate()` | Schema migration |
| `Create()` | Insert |
| `First()`/`Find()` | Select |
| `Save()`/`Updates()` | Update |
| `Delete()` | Soft delete |
| `Preload()` | Eager loading |
| `Begin()`/`Commit()` | Transactions |

---

[← আগের: ভ্যালিডেশন](02-validation.md) | [পরবর্তী: অথেনটিকেশন →](04-authentication.md)
