# ভ্যালিডেশন (Validation)

## go-playground/validator

Go-এর সবচেয়ে জনপ্রিয় validation library।

```bash
go get github.com/go-playground/validator/v10
```

## বেসিক Validation

```go
package main

import (
    "github.com/go-playground/validator/v10"
    "github.com/gofiber/fiber/v3"
)

var validate = validator.New()

type User struct {
    Name     string `json:"name" validate:"required,min=2,max=50"`
    Email    string `json:"email" validate:"required,email"`
    Age      int    `json:"age" validate:"required,gte=18,lte=120"`
    Password string `json:"password" validate:"required,min=8"`
}

func main() {
    app := fiber.New()

    app.Post("/users", func(c fiber.Ctx) error {
        var user User
        
        if err := c.Bind().JSON(&user); err != nil {
            return c.Status(400).JSON(fiber.Map{
                "error": "Invalid JSON",
            })
        }
        
        if err := validate.Struct(&user); err != nil {
            errors := err.(validator.ValidationErrors)
            return c.Status(400).JSON(fiber.Map{
                "errors": formatValidationErrors(errors),
            })
        }
        
        return c.JSON(fiber.Map{
            "message": "User valid",
            "user":    user,
        })
    })

    app.Listen(":3000")
}

func formatValidationErrors(errors validator.ValidationErrors) []map[string]string {
    result := make([]map[string]string, len(errors))
    
    for i, err := range errors {
        result[i] = map[string]string{
            "field":   err.Field(),
            "tag":     err.Tag(),
            "value":   fmt.Sprintf("%v", err.Value()),
            "message": getErrorMessage(err),
        }
    }
    
    return result
}

func getErrorMessage(err validator.FieldError) string {
    switch err.Tag() {
    case "required":
        return "এই ফিল্ড আবশ্যক"
    case "email":
        return "বৈধ ইমেইল দিন"
    case "min":
        return fmt.Sprintf("সর্বনিম্ন %s অক্ষর", err.Param())
    case "max":
        return fmt.Sprintf("সর্বোচ্চ %s অক্ষর", err.Param())
    case "gte":
        return fmt.Sprintf("সর্বনিম্ন মান %s", err.Param())
    case "lte":
        return fmt.Sprintf("সর্বোচ্চ মান %s", err.Param())
    default:
        return "অবৈধ মান"
    }
}
```

## Common Validation Tags

```go
type Product struct {
    // String validations
    Name        string `validate:"required"`
    SKU         string `validate:"required,alphanum,len=8"`
    Description string `validate:"max=500"`
    
    // Numeric validations
    Price    float64 `validate:"required,gt=0"`
    Quantity int     `validate:"required,gte=0,lte=10000"`
    Discount float64 `validate:"gte=0,lte=100"`
    
    // Comparisons
    MinOrder int `validate:"ltfield=MaxOrder"`
    MaxOrder int `validate:"gtfield=MinOrder"`
    
    // Format validations
    Email    string `validate:"email"`
    URL      string `validate:"url"`
    UUID     string `validate:"uuid"`
    IP       string `validate:"ip"`
    
    // Conditional
    Type     string `validate:"required,oneof=physical digital service"`
    
    // Nested
    Category Category `validate:"required"`
    Tags     []string `validate:"min=1,max=5,dive,min=2,max=20"`
}

type Category struct {
    ID   int    `validate:"required,gt=0"`
    Name string `validate:"required,min=2"`
}
```

## Custom Validators

```go
package main

import (
    "regexp"
    "unicode"
    "github.com/go-playground/validator/v10"
)

var validate *validator.Validate

func init() {
    validate = validator.New()
    
    // বাংলাদেশি ফোন নম্বর
    validate.RegisterValidation("bd_phone", validateBDPhone)
    
    // শক্তিশালী পাসওয়ার্ড
    validate.RegisterValidation("strong_password", validateStrongPassword)
    
    // স্লাগ
    validate.RegisterValidation("slug", validateSlug)
}

func validateBDPhone(fl validator.FieldLevel) bool {
    phone := fl.Field().String()
    // +880 বা 880 দিয়ে শুরু, তারপর 11 ডিজিট
    pattern := `^(\+?880)?1[3-9]\d{8}$`
    matched, _ := regexp.MatchString(pattern, phone)
    return matched
}

func validateStrongPassword(fl validator.FieldLevel) bool {
    password := fl.Field().String()
    
    var (
        hasUpper   bool
        hasLower   bool
        hasNumber  bool
        hasSpecial bool
    )
    
    for _, char := range password {
        switch {
        case unicode.IsUpper(char):
            hasUpper = true
        case unicode.IsLower(char):
            hasLower = true
        case unicode.IsDigit(char):
            hasNumber = true
        case unicode.IsPunct(char) || unicode.IsSymbol(char):
            hasSpecial = true
        }
    }
    
    return len(password) >= 8 && hasUpper && hasLower && hasNumber && hasSpecial
}

func validateSlug(fl validator.FieldLevel) bool {
    slug := fl.Field().String()
    pattern := `^[a-z0-9]+(?:-[a-z0-9]+)*$`
    matched, _ := regexp.MatchString(pattern, slug)
    return matched
}

// ব্যবহার
type UserRegistration struct {
    Phone    string `validate:"required,bd_phone"`
    Password string `validate:"required,strong_password"`
    Username string `validate:"required,slug,min=3,max=20"`
}
```

## Conditional Validation

```go
type Order struct {
    Type           string `validate:"required,oneof=delivery pickup"`
    
    // শুধু delivery হলে
    DeliveryAddress string `validate:"required_if=Type delivery"`
    DeliveryDate    string `validate:"required_if=Type delivery"`
    
    // শুধু pickup হলে
    PickupTime string `validate:"required_if=Type pickup"`
    
    // Payment method এর উপর নির্ভর করে
    PaymentMethod string `validate:"required,oneof=cash card bkash"`
    CardNumber    string `validate:"required_if=PaymentMethod card"`
    BkashNumber   string `validate:"required_if=PaymentMethod bkash,bd_phone"`
}
```

## Validation Middleware

```go
package main

import (
    "github.com/go-playground/validator/v10"
    "github.com/gofiber/fiber/v3"
)

var validate = validator.New()

type ValidationError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
}

func ValidateBody[T any]() fiber.Handler {
    return func(c fiber.Ctx) error {
        var body T
        
        if err := c.Bind().JSON(&body); err != nil {
            return c.Status(400).JSON(fiber.Map{
                "error": "Invalid JSON format",
            })
        }
        
        if err := validate.Struct(&body); err != nil {
            errors := make([]ValidationError, 0)
            
            for _, err := range err.(validator.ValidationErrors) {
                errors = append(errors, ValidationError{
                    Field:   err.Field(),
                    Message: getValidationMessage(err),
                })
            }
            
            return c.Status(400).JSON(fiber.Map{
                "success": false,
                "errors":  errors,
            })
        }
        
        // Validated body সংরক্ষণ করুন
        c.Locals("body", &body)
        
        return c.Next()
    }
}

func getValidationMessage(err validator.FieldError) string {
    messages := map[string]string{
        "required":        "এই ফিল্ড আবশ্যক",
        "email":           "বৈধ ইমেইল দিন",
        "min":             fmt.Sprintf("সর্বনিম্ন %s", err.Param()),
        "max":             fmt.Sprintf("সর্বোচ্চ %s", err.Param()),
        "gte":             fmt.Sprintf("সর্বনিম্ন %s হতে হবে", err.Param()),
        "lte":             fmt.Sprintf("সর্বোচ্চ %s হতে হবে", err.Param()),
        "oneof":           fmt.Sprintf("মান হতে হবে: %s", err.Param()),
        "strong_password": "পাসওয়ার্ড শক্তিশালী হতে হবে",
        "bd_phone":        "বৈধ বাংলাদেশি ফোন নম্বর দিন",
    }
    
    if msg, ok := messages[err.Tag()]; ok {
        return msg
    }
    return "অবৈধ মান"
}

// ব্যবহার
type CreateProductRequest struct {
    Name  string  `json:"name" validate:"required,min=2,max=100"`
    Price float64 `json:"price" validate:"required,gt=0"`
    SKU   string  `json:"sku" validate:"required,alphanum,len=8"`
}

func main() {
    app := fiber.New()

    app.Post("/products", ValidateBody[CreateProductRequest](), func(c fiber.Ctx) error {
        body := c.Locals("body").(*CreateProductRequest)
        
        return c.JSON(fiber.Map{
            "message": "Product created",
            "product": body,
        })
    })

    app.Listen(":3000")
}
```

## Query Params Validation

```go
type SearchParams struct {
    Query    string `query:"q" validate:"required,min=2"`
    Page     int    `query:"page" validate:"gte=1"`
    Limit    int    `query:"limit" validate:"gte=1,lte=100"`
    SortBy   string `query:"sort_by" validate:"omitempty,oneof=name price date"`
    SortDir  string `query:"sort_dir" validate:"omitempty,oneof=asc desc"`
}

func ValidateQuery[T any]() fiber.Handler {
    return func(c fiber.Ctx) error {
        var params T
        
        if err := c.Bind().Query(&params); err != nil {
            return c.Status(400).JSON(fiber.Map{
                "error": "Invalid query parameters",
            })
        }
        
        if err := validate.Struct(&params); err != nil {
            errors := formatValidationErrors(err.(validator.ValidationErrors))
            return c.Status(400).JSON(fiber.Map{
                "errors": errors,
            })
        }
        
        c.Locals("query", &params)
        return c.Next()
    }
}

// ব্যবহার
app.Get("/search", ValidateQuery[SearchParams](), func(c fiber.Ctx) error {
    params := c.Locals("query").(*SearchParams)
    // search লজিক
    return c.JSON(fiber.Map{"params": params})
})
```

## File Validation

```go
package main

import (
    "mime/multipart"
    "path/filepath"
    "strings"
    "github.com/gofiber/fiber/v3"
)

type FileValidator struct {
    MaxSize      int64    // bytes
    AllowedTypes []string // MIME types
    AllowedExts  []string // Extensions
}

func (fv *FileValidator) Validate(file *multipart.FileHeader) error {
    // সাইজ চেক
    if file.Size > fv.MaxSize {
        return fmt.Errorf("ফাইল সাইজ %d MB এর বেশি হতে পারবে না", fv.MaxSize/1024/1024)
    }
    
    // Extension চেক
    ext := strings.ToLower(filepath.Ext(file.Filename))
    if len(fv.AllowedExts) > 0 {
        allowed := false
        for _, e := range fv.AllowedExts {
            if ext == e {
                allowed = true
                break
            }
        }
        if !allowed {
            return fmt.Errorf("অনুমোদিত ফরম্যাট: %s", strings.Join(fv.AllowedExts, ", "))
        }
    }
    
    // MIME type চেক
    contentType := file.Header.Get("Content-Type")
    if len(fv.AllowedTypes) > 0 {
        allowed := false
        for _, t := range fv.AllowedTypes {
            if contentType == t {
                allowed = true
                break
            }
        }
        if !allowed {
            return fmt.Errorf("অনুমোদিত টাইপ: %s", strings.Join(fv.AllowedTypes, ", "))
        }
    }
    
    return nil
}

var imageValidator = &FileValidator{
    MaxSize:      5 * 1024 * 1024, // 5MB
    AllowedExts:  []string{".jpg", ".jpeg", ".png", ".gif"},
    AllowedTypes: []string{"image/jpeg", "image/png", "image/gif"},
}

func main() {
    app := fiber.New()

    app.Post("/upload", func(c fiber.Ctx) error {
        file, err := c.FormFile("image")
        if err != nil {
            return c.Status(400).JSON(fiber.Map{"error": "ফাইল আবশ্যক"})
        }
        
        if err := imageValidator.Validate(file); err != nil {
            return c.Status(400).JSON(fiber.Map{"error": err.Error()})
        }
        
        // ফাইল সেভ করুন
        c.SaveFile(file, fmt.Sprintf("./uploads/%s", file.Filename))
        
        return c.JSON(fiber.Map{"message": "আপলোড সফল"})
    })

    app.Listen(":3000")
}
```

## Struct-Level Validation

```go
package main

import (
    "github.com/go-playground/validator/v10"
)

type DateRange struct {
    StartDate time.Time `json:"start_date" validate:"required"`
    EndDate   time.Time `json:"end_date" validate:"required"`
}

func init() {
    validate = validator.New()
    
    // Struct-level validation
    validate.RegisterStructValidation(validateDateRange, DateRange{})
}

func validateDateRange(sl validator.StructLevel) {
    dr := sl.Current().Interface().(DateRange)
    
    if !dr.EndDate.After(dr.StartDate) {
        sl.ReportError(dr.EndDate, "EndDate", "end_date", "gtfield", "StartDate")
    }
}
```

## Validation Error Messages (Bangla)

```go
package main

var banglaMessages = map[string]string{
    "required":        "এই ফিল্ড পূরণ করা আবশ্যক",
    "email":           "সঠিক ইমেইল ঠিকানা দিন",
    "min":             "সর্বনিম্ন %s টি অক্ষর প্রয়োজন",
    "max":             "সর্বোচ্চ %s টি অক্ষর অনুমোদিত",
    "gte":             "মান কমপক্ষে %s হতে হবে",
    "lte":             "মান সর্বোচ্চ %s হতে পারবে",
    "gt":              "মান %s এর বেশি হতে হবে",
    "lt":              "মান %s এর কম হতে হবে",
    "len":             "দৈর্ঘ্য %s হতে হবে",
    "eq":              "মান %s এর সমান হতে হবে",
    "ne":              "মান %s এর সমান হতে পারবে না",
    "oneof":           "মান এর মধ্যে একটি হতে হবে: %s",
    "alphanum":        "শুধু অক্ষর এবং সংখ্যা অনুমোদিত",
    "alpha":           "শুধু অক্ষর অনুমোদিত",
    "numeric":         "শুধু সংখ্যা অনুমোদিত",
    "url":             "সঠিক URL দিন",
    "uuid":            "সঠিক UUID দিন",
    "contains":        "'%s' থাকতে হবে",
    "excludes":        "'%s' থাকতে পারবে না",
    "startswith":      "'%s' দিয়ে শুরু হতে হবে",
    "endswith":        "'%s' দিয়ে শেষ হতে হবে",
    "bd_phone":        "সঠিক বাংলাদেশি মোবাইল নম্বর দিন",
    "strong_password": "পাসওয়ার্ডে বড় হাতের অক্ষর, ছোট হাতের অক্ষর, সংখ্যা এবং বিশেষ চিহ্ন থাকতে হবে",
}

func getBanglaMessage(err validator.FieldError) string {
    if template, ok := banglaMessages[err.Tag()]; ok {
        if err.Param() != "" {
            return fmt.Sprintf(template, err.Param())
        }
        return template
    }
    return "অবৈধ মান"
}
```

## সারসংক্ষেপ

| Tag | বর্ণনা |
|-----|--------|
| `required` | আবশ্যক |
| `email` | ইমেইল ফরম্যাট |
| `min=n` | সর্বনিম্ন দৈর্ঘ্য/মান |
| `max=n` | সর্বোচ্চ দৈর্ঘ্য/মান |
| `gte=n` | Greater than or equal |
| `lte=n` | Less than or equal |
| `oneof=a b c` | নির্দিষ্ট মানগুলোর একটি |
| `dive` | Slice/Map এর উপাদান validate |

---

[← আগের: এরর হ্যান্ডলিং](01-error-handling.md) | [পরবর্তী: ডেটাবেস →](03-database.md)
