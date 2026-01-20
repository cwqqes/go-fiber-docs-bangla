# এরর হ্যান্ডলিং (Error Handling)

## Go-এ Error Handling

Go-তে error হলো একটি built-in interface:

```go
type error interface {
    Error() string
}
```

## বেসিক Error Handling

```go
package main

import (
    "errors"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New()

    app.Get("/users/:id", func(c fiber.Ctx) error {
        id := c.Params("id")
        
        user, err := findUser(id)
        if err != nil {
            return c.Status(404).JSON(fiber.Map{
                "error": err.Error(),
            })
        }
        
        return c.JSON(user)
    })

    app.Listen(":3000")
}

func findUser(id string) (map[string]string, error) {
    if id == "" {
        return nil, errors.New("user ID is required")
    }
    
    // সিমুলেটেড ডেটাবেস লুকআপ
    if id != "1" {
        return nil, errors.New("user not found")
    }
    
    return map[string]string{"id": id, "name": "User"}, nil
}
```

## Custom Error Types

```go
package main

import (
    "fmt"
    "github.com/gofiber/fiber/v3"
)

// Custom Error Type
type AppError struct {
    Code    int    `json:"code"`
    Message string `json:"message"`
    Details string `json:"details,omitempty"`
}

func (e *AppError) Error() string {
    return fmt.Sprintf("%d: %s", e.Code, e.Message)
}

// Error Constructors
func NewNotFoundError(resource string) *AppError {
    return &AppError{
        Code:    404,
        Message: fmt.Sprintf("%s not found", resource),
    }
}

func NewValidationError(details string) *AppError {
    return &AppError{
        Code:    400,
        Message: "Validation failed",
        Details: details,
    }
}

func NewInternalError(details string) *AppError {
    return &AppError{
        Code:    500,
        Message: "Internal server error",
        Details: details,
    }
}

func main() {
    app := fiber.New()

    app.Get("/users/:id", func(c fiber.Ctx) error {
        id := c.Params("id")
        
        user, err := getUser(id)
        if err != nil {
            if appErr, ok := err.(*AppError); ok {
                return c.Status(appErr.Code).JSON(appErr)
            }
            return c.Status(500).JSON(fiber.Map{"error": err.Error()})
        }
        
        return c.JSON(user)
    })

    app.Listen(":3000")
}

func getUser(id string) (interface{}, error) {
    if id == "" {
        return nil, NewValidationError("user ID is required")
    }
    
    if id == "999" {
        return nil, NewNotFoundError("User")
    }
    
    return map[string]string{"id": id, "name": "User"}, nil
}
```

## Global Error Handler

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
)

func main() {
    app := fiber.New(fiber.Config{
        // Global Error Handler
        ErrorHandler: func(c fiber.Ctx, err error) error {
            // Default status code
            code := fiber.StatusInternalServerError
            message := "Internal Server Error"
            
            // Fiber error চেক করুন
            if e, ok := err.(*fiber.Error); ok {
                code = e.Code
                message = e.Message
            }
            
            // Custom error চেক করুন
            if e, ok := err.(*AppError); ok {
                code = e.Code
                return c.Status(code).JSON(e)
            }
            
            // লগ করুন
            log.Printf("Error: %v\n", err)
            
            return c.Status(code).JSON(fiber.Map{
                "error":   message,
                "success": false,
            })
        },
    })

    app.Get("/", func(c fiber.Ctx) error {
        return fiber.NewError(fiber.StatusBadRequest, "Bad Request")
    })

    app.Get("/panic", func(c fiber.Ctx) error {
        panic("Something went wrong!")
    })

    app.Listen(":3000")
}
```

## Error Wrapping

```go
package main

import (
    "errors"
    "fmt"
)

var (
    ErrNotFound     = errors.New("resource not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrForbidden    = errors.New("forbidden")
)

func getUser(id string) (interface{}, error) {
    // সিমুলেটেড ডেটাবেস এরর
    err := databaseLookup(id)
    if err != nil {
        return nil, fmt.Errorf("failed to get user %s: %w", id, err)
    }
    return nil, nil
}

func databaseLookup(id string) error {
    return ErrNotFound
}

func handleError(err error) {
    // Error unwrap করুন
    if errors.Is(err, ErrNotFound) {
        fmt.Println("Resource not found")
    } else if errors.Is(err, ErrUnauthorized) {
        fmt.Println("User not authorized")
    } else {
        fmt.Println("Unknown error:", err)
    }
}
```

## Fiber-এ Error Middleware

```go
package main

import (
    "log"
    "runtime/debug"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/recover"
)

func main() {
    app := fiber.New()

    // Recover middleware
    app.Use(recover.New(recover.Config{
        EnableStackTrace: true,
        StackTraceHandler: func(c fiber.Ctx, e interface{}) {
            log.Printf("Panic: %v\n%s", e, debug.Stack())
        },
    }))

    // Custom error logging middleware
    app.Use(func(c fiber.Ctx) error {
        err := c.Next()
        
        if err != nil {
            log.Printf("[ERROR] %s %s: %v\n", 
                c.Method(), c.Path(), err)
        }
        
        return err
    })

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello!")
    })

    app.Listen(":3000")
}
```

## Validation Errors

```go
package main

import (
    "fmt"
    "github.com/go-playground/validator/v10"
    "github.com/gofiber/fiber/v3"
)

type ValidationError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
}

type ValidationErrors struct {
    Errors []ValidationError `json:"errors"`
}

func (v *ValidationErrors) Error() string {
    return fmt.Sprintf("validation failed: %d errors", len(v.Errors))
}

var validate = validator.New()

func ValidateStruct(s interface{}) error {
    err := validate.Struct(s)
    if err == nil {
        return nil
    }
    
    var errors []ValidationError
    
    for _, err := range err.(validator.ValidationErrors) {
        errors = append(errors, ValidationError{
            Field:   err.Field(),
            Message: getErrorMessage(err),
        })
    }
    
    return &ValidationErrors{Errors: errors}
}

func getErrorMessage(err validator.FieldError) string {
    switch err.Tag() {
    case "required":
        return "This field is required"
    case "email":
        return "Invalid email format"
    case "min":
        return fmt.Sprintf("Minimum length is %s", err.Param())
    case "max":
        return fmt.Sprintf("Maximum length is %s", err.Param())
    default:
        return "Invalid value"
    }
}

type CreateUserRequest struct {
    Name     string `json:"name" validate:"required,min=2,max=50"`
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
}

func main() {
    app := fiber.New(fiber.Config{
        ErrorHandler: func(c fiber.Ctx, err error) error {
            if ve, ok := err.(*ValidationErrors); ok {
                return c.Status(400).JSON(fiber.Map{
                    "success": false,
                    "errors":  ve.Errors,
                })
            }
            
            return c.Status(500).JSON(fiber.Map{
                "success": false,
                "error":   err.Error(),
            })
        },
    })

    app.Post("/users", func(c fiber.Ctx) error {
        var req CreateUserRequest
        
        if err := c.Bind().JSON(&req); err != nil {
            return err
        }
        
        if err := ValidateStruct(&req); err != nil {
            return err
        }
        
        return c.JSON(fiber.Map{
            "success": true,
            "message": "User created",
        })
    })

    app.Listen(":3000")
}
```

## Error Response Standardization

```go
package main

import (
    "time"
    "github.com/gofiber/fiber/v3"
)

type APIResponse struct {
    Success   bool        `json:"success"`
    Data      interface{} `json:"data,omitempty"`
    Error     *APIError   `json:"error,omitempty"`
    Timestamp time.Time   `json:"timestamp"`
}

type APIError struct {
    Code    string `json:"code"`
    Message string `json:"message"`
    Details interface{} `json:"details,omitempty"`
}

func SuccessResponse(c fiber.Ctx, data interface{}) error {
    return c.JSON(APIResponse{
        Success:   true,
        Data:      data,
        Timestamp: time.Now(),
    })
}

func ErrorResponse(c fiber.Ctx, status int, code, message string, details interface{}) error {
    return c.Status(status).JSON(APIResponse{
        Success: false,
        Error: &APIError{
            Code:    code,
            Message: message,
            Details: details,
        },
        Timestamp: time.Now(),
    })
}

func main() {
    app := fiber.New()

    app.Get("/users/:id", func(c fiber.Ctx) error {
        id := c.Params("id")
        
        if id == "0" {
            return ErrorResponse(c, 400, "INVALID_ID", "User ID cannot be 0", nil)
        }
        
        if id == "999" {
            return ErrorResponse(c, 404, "NOT_FOUND", "User not found", fiber.Map{
                "searched_id": id,
            })
        }
        
        user := fiber.Map{"id": id, "name": "User"}
        return SuccessResponse(c, user)
    })

    app.Listen(":3000")
}
```

## Error Logging with Context

```go
package main

import (
    "log"
    "os"
    "github.com/gofiber/fiber/v3"
)

type Logger struct {
    *log.Logger
}

func NewLogger() *Logger {
    return &Logger{
        Logger: log.New(os.Stdout, "[APP] ", log.LstdFlags|log.Lshortfile),
    }
}

func (l *Logger) ErrorWithContext(c fiber.Ctx, err error, message string) {
    l.Printf("[ERROR] %s %s | IP: %s | Error: %v | Message: %s",
        c.Method(),
        c.Path(),
        c.IP(),
        err,
        message,
    )
}

var logger = NewLogger()

func main() {
    app := fiber.New()

    app.Use(func(c fiber.Ctx) error {
        c.Locals("logger", logger)
        return c.Next()
    })

    app.Get("/", func(c fiber.Ctx) error {
        log := c.Locals("logger").(*Logger)
        
        err := doSomething()
        if err != nil {
            log.ErrorWithContext(c, err, "Failed to do something")
            return c.Status(500).JSON(fiber.Map{"error": "Internal error"})
        }
        
        return c.SendString("OK")
    })

    app.Listen(":3000")
}

func doSomething() error {
    return nil
}
```

## Error Codes Catalog

```go
package main

// Error codes
const (
    ErrCodeUnknown          = "UNKNOWN_ERROR"
    ErrCodeValidation       = "VALIDATION_ERROR"
    ErrCodeNotFound         = "NOT_FOUND"
    ErrCodeUnauthorized     = "UNAUTHORIZED"
    ErrCodeForbidden        = "FORBIDDEN"
    ErrCodeConflict         = "CONFLICT"
    ErrCodeRateLimit        = "RATE_LIMIT_EXCEEDED"
    ErrCodeInternalServer   = "INTERNAL_SERVER_ERROR"
    ErrCodeDatabaseError    = "DATABASE_ERROR"
    ErrCodeExternalService  = "EXTERNAL_SERVICE_ERROR"
)

// Error messages
var errorMessages = map[string]string{
    ErrCodeUnknown:         "An unknown error occurred",
    ErrCodeValidation:      "Validation failed",
    ErrCodeNotFound:        "Resource not found",
    ErrCodeUnauthorized:    "Authentication required",
    ErrCodeForbidden:       "Access denied",
    ErrCodeConflict:        "Resource already exists",
    ErrCodeRateLimit:       "Too many requests",
    ErrCodeInternalServer:  "Internal server error",
    ErrCodeDatabaseError:   "Database operation failed",
    ErrCodeExternalService: "External service unavailable",
}

func GetErrorMessage(code string) string {
    if msg, ok := errorMessages[code]; ok {
        return msg
    }
    return errorMessages[ErrCodeUnknown]
}
```

## সারসংক্ষেপ

| প্যাটার্ন | ব্যবহার |
|---------|--------|
| Custom Error Types | স্ট্রাকচার্ড এরর |
| Error Wrapping | কনটেক্সট যোগ করা |
| Global Handler | সেন্ট্রালাইজড হ্যান্ডলিং |
| Validation Errors | ইনপুট ভ্যালিডেশন |
| Error Logging | ডিবাগিং ও মনিটরিং |
| Standard Response | কনসিস্টেন্ট API |

---

[← আগের: টাস্ক কিউ](../08-scheduler/04-task-queues.md) | [পরবর্তী: ভ্যালিডেশন →](02-validation.md)
