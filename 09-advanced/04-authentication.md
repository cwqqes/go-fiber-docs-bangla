# অথেনটিকেশন (Authentication)

## JWT Authentication

JSON Web Token (JWT) হলো সবচেয়ে জনপ্রিয় stateless authentication পদ্ধতি।

```bash
go get github.com/golang-jwt/jwt/v5
```

## JWT Basics

```go
package main

import (
    "time"
    "github.com/golang-jwt/jwt/v5"
)

var jwtSecret = []byte("your-secret-key") // Production এ ENV থেকে নিন

type Claims struct {
    UserID uint   `json:"user_id"`
    Email  string `json:"email"`
    Role   string `json:"role"`
    jwt.RegisteredClaims
}

func GenerateToken(userID uint, email, role string) (string, error) {
    claims := &Claims{
        UserID: userID,
        Email:  email,
        Role:   role,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            Issuer:    "my-app",
        },
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(jwtSecret)
}

func ValidateToken(tokenString string) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(token *jwt.Token) (interface{}, error) {
        return jwtSecret, nil
    })
    
    if err != nil {
        return nil, err
    }
    
    if claims, ok := token.Claims.(*Claims); ok && token.Valid {
        return claims, nil
    }
    
    return nil, jwt.ErrSignatureInvalid
}
```

## Auth Middleware

```go
package main

import (
    "strings"
    "github.com/gofiber/fiber/v3"
)

func AuthMiddleware() fiber.Handler {
    return func(c fiber.Ctx) error {
        authHeader := c.Get("Authorization")
        
        if authHeader == "" {
            return c.Status(401).JSON(fiber.Map{
                "error": "Authorization header missing",
            })
        }
        
        // Bearer token চেক
        parts := strings.Split(authHeader, " ")
        if len(parts) != 2 || parts[0] != "Bearer" {
            return c.Status(401).JSON(fiber.Map{
                "error": "Invalid authorization format",
            })
        }
        
        tokenString := parts[1]
        claims, err := ValidateToken(tokenString)
        if err != nil {
            return c.Status(401).JSON(fiber.Map{
                "error": "Invalid or expired token",
            })
        }
        
        // Claims সংরক্ষণ করুন
        c.Locals("user_id", claims.UserID)
        c.Locals("email", claims.Email)
        c.Locals("role", claims.Role)
        c.Locals("claims", claims)
        
        return c.Next()
    }
}

// Role-based middleware
func RoleMiddleware(allowedRoles ...string) fiber.Handler {
    return func(c fiber.Ctx) error {
        role := c.Locals("role").(string)
        
        for _, allowed := range allowedRoles {
            if role == allowed {
                return c.Next()
            }
        }
        
        return c.Status(403).JSON(fiber.Map{
            "error": "Access denied",
        })
    }
}
```

## Complete Auth System

```go
package main

import (
    "log"
    "time"
    "github.com/gofiber/fiber/v3"
    "golang.org/x/crypto/bcrypt"
    "gorm.io/gorm"
)

type User struct {
    ID        uint           `gorm:"primaryKey" json:"id"`
    Name      string         `gorm:"size:100" json:"name"`
    Email     string         `gorm:"size:100;uniqueIndex" json:"email"`
    Password  string         `gorm:"size:255" json:"-"`
    Role      string         `gorm:"size:20;default:user" json:"role"`
    IsActive  bool           `gorm:"default:true" json:"is_active"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
}

type AuthService struct {
    db *gorm.DB
}

func NewAuthService(db *gorm.DB) *AuthService {
    return &AuthService{db: db}
}

// Register
func (s *AuthService) Register(c fiber.Ctx) error {
    var req struct {
        Name     string `json:"name" validate:"required,min=2"`
        Email    string `json:"email" validate:"required,email"`
        Password string `json:"password" validate:"required,min=8"`
    }
    
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
    }
    
    // Email চেক
    var existingUser User
    if err := s.db.Where("email = ?", req.Email).First(&existingUser).Error; err == nil {
        return c.Status(400).JSON(fiber.Map{"error": "Email already exists"})
    }
    
    // Password hash
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
    if err != nil {
        return c.Status(500).JSON(fiber.Map{"error": "Failed to hash password"})
    }
    
    user := User{
        Name:     req.Name,
        Email:    req.Email,
        Password: string(hashedPassword),
        Role:     "user",
    }
    
    if err := s.db.Create(&user).Error; err != nil {
        return c.Status(500).JSON(fiber.Map{"error": "Failed to create user"})
    }
    
    // Token generate
    token, err := GenerateToken(user.ID, user.Email, user.Role)
    if err != nil {
        return c.Status(500).JSON(fiber.Map{"error": "Failed to generate token"})
    }
    
    return c.Status(201).JSON(fiber.Map{
        "message": "Registration successful",
        "token":   token,
        "user": fiber.Map{
            "id":    user.ID,
            "name":  user.Name,
            "email": user.Email,
            "role":  user.Role,
        },
    })
}

// Login
func (s *AuthService) Login(c fiber.Ctx) error {
    var req struct {
        Email    string `json:"email" validate:"required,email"`
        Password string `json:"password" validate:"required"`
    }
    
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
    }
    
    var user User
    if err := s.db.Where("email = ?", req.Email).First(&user).Error; err != nil {
        return c.Status(401).JSON(fiber.Map{"error": "Invalid credentials"})
    }
    
    if !user.IsActive {
        return c.Status(401).JSON(fiber.Map{"error": "Account is deactivated"})
    }
    
    // Password verify
    if err := bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(req.Password)); err != nil {
        return c.Status(401).JSON(fiber.Map{"error": "Invalid credentials"})
    }
    
    // Token generate
    token, err := GenerateToken(user.ID, user.Email, user.Role)
    if err != nil {
        return c.Status(500).JSON(fiber.Map{"error": "Failed to generate token"})
    }
    
    return c.JSON(fiber.Map{
        "message": "Login successful",
        "token":   token,
        "user": fiber.Map{
            "id":    user.ID,
            "name":  user.Name,
            "email": user.Email,
            "role":  user.Role,
        },
    })
}

// Get Profile
func (s *AuthService) GetProfile(c fiber.Ctx) error {
    userID := c.Locals("user_id").(uint)
    
    var user User
    if err := s.db.First(&user, userID).Error; err != nil {
        return c.Status(404).JSON(fiber.Map{"error": "User not found"})
    }
    
    return c.JSON(fiber.Map{
        "id":         user.ID,
        "name":       user.Name,
        "email":      user.Email,
        "role":       user.Role,
        "created_at": user.CreatedAt,
    })
}

// Change Password
func (s *AuthService) ChangePassword(c fiber.Ctx) error {
    userID := c.Locals("user_id").(uint)
    
    var req struct {
        CurrentPassword string `json:"current_password" validate:"required"`
        NewPassword     string `json:"new_password" validate:"required,min=8"`
    }
    
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
    }
    
    var user User
    if err := s.db.First(&user, userID).Error; err != nil {
        return c.Status(404).JSON(fiber.Map{"error": "User not found"})
    }
    
    // Current password verify
    if err := bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(req.CurrentPassword)); err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Current password is incorrect"})
    }
    
    // New password hash
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(req.NewPassword), bcrypt.DefaultCost)
    if err != nil {
        return c.Status(500).JSON(fiber.Map{"error": "Failed to hash password"})
    }
    
    user.Password = string(hashedPassword)
    s.db.Save(&user)
    
    return c.JSON(fiber.Map{"message": "Password changed successfully"})
}

func main() {
    // DB setup
    // ...
    
    authService := NewAuthService(DB)

    app := fiber.New()

    // Public routes
    auth := app.Group("/auth")
    auth.Post("/register", authService.Register)
    auth.Post("/login", authService.Login)

    // Protected routes
    api := app.Group("/api", AuthMiddleware())
    api.Get("/profile", authService.GetProfile)
    api.Post("/change-password", authService.ChangePassword)

    // Admin routes
    admin := api.Group("/admin", RoleMiddleware("admin"))
    admin.Get("/users", func(c fiber.Ctx) error {
        // Admin only
        return c.JSON(fiber.Map{"message": "Admin area"})
    })

    app.Listen(":3000")
}
```

## Refresh Token

```go
type RefreshToken struct {
    ID        uint      `gorm:"primaryKey"`
    UserID    uint      `gorm:"index"`
    Token     string    `gorm:"uniqueIndex"`
    ExpiresAt time.Time
    CreatedAt time.Time
}

func GenerateTokenPair(userID uint, email, role string) (accessToken, refreshToken string, err error) {
    // Access Token (short-lived)
    accessClaims := &Claims{
        UserID: userID,
        Email:  email,
        Role:   role,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(15 * time.Minute)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
        },
    }
    accessJwt := jwt.NewWithClaims(jwt.SigningMethodHS256, accessClaims)
    accessToken, err = accessJwt.SignedString(jwtSecret)
    if err != nil {
        return "", "", err
    }
    
    // Refresh Token (long-lived)
    refreshClaims := jwt.RegisteredClaims{
        ExpiresAt: jwt.NewNumericDate(time.Now().Add(7 * 24 * time.Hour)),
        IssuedAt:  jwt.NewNumericDate(time.Now()),
        Subject:   fmt.Sprintf("%d", userID),
    }
    refreshJwt := jwt.NewWithClaims(jwt.SigningMethodHS256, refreshClaims)
    refreshToken, err = refreshJwt.SignedString(refreshSecret)
    
    return accessToken, refreshToken, err
}

func (s *AuthService) RefreshToken(c fiber.Ctx) error {
    var req struct {
        RefreshToken string `json:"refresh_token"`
    }
    
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Invalid request"})
    }
    
    // Refresh token validate
    token, err := jwt.Parse(req.RefreshToken, func(token *jwt.Token) (interface{}, error) {
        return refreshSecret, nil
    })
    
    if err != nil || !token.Valid {
        return c.Status(401).JSON(fiber.Map{"error": "Invalid refresh token"})
    }
    
    claims := token.Claims.(jwt.MapClaims)
    userID, _ := strconv.ParseUint(claims["sub"].(string), 10, 32)
    
    // User fetch
    var user User
    if err := s.db.First(&user, userID).Error; err != nil {
        return c.Status(401).JSON(fiber.Map{"error": "User not found"})
    }
    
    // New tokens generate
    accessToken, refreshToken, err := GenerateTokenPair(user.ID, user.Email, user.Role)
    if err != nil {
        return c.Status(500).JSON(fiber.Map{"error": "Failed to generate tokens"})
    }
    
    return c.JSON(fiber.Map{
        "access_token":  accessToken,
        "refresh_token": refreshToken,
    })
}
```

## OAuth2 (Google)

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "golang.org/x/oauth2"
    "golang.org/x/oauth2/google"
)

var googleOAuthConfig = &oauth2.Config{
    ClientID:     "your-client-id",
    ClientSecret: "your-client-secret",
    RedirectURL:  "http://localhost:3000/auth/google/callback",
    Scopes: []string{
        "https://www.googleapis.com/auth/userinfo.email",
        "https://www.googleapis.com/auth/userinfo.profile",
    },
    Endpoint: google.Endpoint,
}

func GoogleLogin(c fiber.Ctx) error {
    state := generateRandomState() // Random state for CSRF protection
    c.Cookie(&fiber.Cookie{
        Name:     "oauth_state",
        Value:    state,
        HTTPOnly: true,
    })
    
    url := googleOAuthConfig.AuthCodeURL(state)
    return c.Redirect().To(url)
}

func GoogleCallback(c fiber.Ctx) error {
    // State verify
    state := c.Cookies("oauth_state")
    if state != c.Query("state") {
        return c.Status(400).JSON(fiber.Map{"error": "Invalid state"})
    }
    
    code := c.Query("code")
    token, err := googleOAuthConfig.Exchange(c.Context(), code)
    if err != nil {
        return c.Status(400).JSON(fiber.Map{"error": "Failed to exchange token"})
    }
    
    // Get user info
    client := googleOAuthConfig.Client(c.Context(), token)
    resp, err := client.Get("https://www.googleapis.com/oauth2/v2/userinfo")
    if err != nil {
        return c.Status(500).JSON(fiber.Map{"error": "Failed to get user info"})
    }
    defer resp.Body.Close()
    
    body, _ := ioutil.ReadAll(resp.Body)
    
    var googleUser struct {
        ID      string `json:"id"`
        Email   string `json:"email"`
        Name    string `json:"name"`
        Picture string `json:"picture"`
    }
    json.Unmarshal(body, &googleUser)
    
    // Find or create user
    var user User
    result := DB.Where("email = ?", googleUser.Email).First(&user)
    
    if result.Error != nil {
        // Create new user
        user = User{
            Name:     googleUser.Name,
            Email:    googleUser.Email,
            Password: "", // OAuth user has no password
            Role:     "user",
        }
        DB.Create(&user)
    }
    
    // Generate JWT
    jwtToken, _ := GenerateToken(user.ID, user.Email, user.Role)
    
    // Redirect with token
    return c.Redirect().To(fmt.Sprintf("/auth/success?token=%s", jwtToken))
}
```

## API Key Authentication

```go
type APIKey struct {
    ID        uint      `gorm:"primaryKey"`
    UserID    uint      `gorm:"index"`
    Key       string    `gorm:"uniqueIndex"`
    Name      string
    Scopes    string    // comma-separated
    ExpiresAt *time.Time
    LastUsed  *time.Time
    CreatedAt time.Time
}

func APIKeyMiddleware() fiber.Handler {
    return func(c fiber.Ctx) error {
        apiKey := c.Get("X-API-Key")
        if apiKey == "" {
            apiKey = c.Query("api_key")
        }
        
        if apiKey == "" {
            return c.Status(401).JSON(fiber.Map{
                "error": "API key required",
            })
        }
        
        var key APIKey
        if err := DB.Where("key = ?", apiKey).First(&key).Error; err != nil {
            return c.Status(401).JSON(fiber.Map{
                "error": "Invalid API key",
            })
        }
        
        // Expiry check
        if key.ExpiresAt != nil && time.Now().After(*key.ExpiresAt) {
            return c.Status(401).JSON(fiber.Map{
                "error": "API key expired",
            })
        }
        
        // Update last used
        now := time.Now()
        key.LastUsed = &now
        DB.Save(&key)
        
        c.Locals("api_key", &key)
        c.Locals("user_id", key.UserID)
        
        return c.Next()
    }
}

func GenerateAPIKey() string {
    bytes := make([]byte, 32)
    rand.Read(bytes)
    return base64.URLEncoding.EncodeToString(bytes)
}
```

## সারসংক্ষেপ

| Method | Use Case |
|--------|----------|
| JWT | Stateless API auth |
| Session | Traditional web apps |
| OAuth2 | Third-party login |
| API Key | Machine-to-machine |
| Basic Auth | Simple scenarios |

---

[← আগের: ডেটাবেস](03-database.md) | [পরবর্তী: ওয়েবসকেট →](05-websockets.md)
