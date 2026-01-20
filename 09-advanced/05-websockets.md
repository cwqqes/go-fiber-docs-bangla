# ওয়েবসকেট (WebSockets)

## WebSocket কি?

WebSocket হলো একটি full-duplex communication protocol যা একই TCP connection এ bidirectional data transfer করতে দেয়।

```bash
go get github.com/gofiber/contrib/websocket
```

## বেসিক WebSocket

```go
package main

import (
    "log"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/contrib/websocket"
)

func main() {
    app := fiber.New()

    // WebSocket upgrade middleware
    app.Use("/ws", func(c fiber.Ctx) error {
        if websocket.IsWebSocketUpgrade(c) {
            return c.Next()
        }
        return fiber.ErrUpgradeRequired
    })

    app.Get("/ws", websocket.New(func(c *websocket.Conn) {
        log.Println("New WebSocket connection")
        
        for {
            // Message পড়ুন
            messageType, message, err := c.ReadMessage()
            if err != nil {
                log.Println("Read error:", err)
                break
            }
            
            log.Printf("Received: %s\n", message)
            
            // Echo back
            if err := c.WriteMessage(messageType, message); err != nil {
                log.Println("Write error:", err)
                break
            }
        }
        
        log.Println("WebSocket disconnected")
    }))

    log.Fatal(app.Listen(":3000"))
}
```

## Chat Application

```go
package main

import (
    "encoding/json"
    "log"
    "sync"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/contrib/websocket"
    "github.com/google/uuid"
)

type Client struct {
    ID       string
    Username string
    Conn     *websocket.Conn
    Room     string
}

type Message struct {
    Type      string    `json:"type"`
    From      string    `json:"from"`
    Content   string    `json:"content"`
    Room      string    `json:"room"`
    Timestamp time.Time `json:"timestamp"`
}

type Hub struct {
    clients    map[string]*Client
    rooms      map[string]map[string]*Client
    broadcast  chan Message
    register   chan *Client
    unregister chan *Client
    mu         sync.RWMutex
}

func NewHub() *Hub {
    return &Hub{
        clients:    make(map[string]*Client),
        rooms:      make(map[string]map[string]*Client),
        broadcast:  make(chan Message),
        register:   make(chan *Client),
        unregister: make(chan *Client),
    }
}

func (h *Hub) Run() {
    for {
        select {
        case client := <-h.register:
            h.mu.Lock()
            h.clients[client.ID] = client
            
            if _, ok := h.rooms[client.Room]; !ok {
                h.rooms[client.Room] = make(map[string]*Client)
            }
            h.rooms[client.Room][client.ID] = client
            h.mu.Unlock()
            
            log.Printf("Client %s joined room %s\n", client.Username, client.Room)
            
            // Notify room
            h.broadcast <- Message{
                Type:      "system",
                Content:   client.Username + " joined the room",
                Room:      client.Room,
                Timestamp: time.Now(),
            }
            
        case client := <-h.unregister:
            h.mu.Lock()
            if _, ok := h.clients[client.ID]; ok {
                delete(h.clients, client.ID)
                if room, ok := h.rooms[client.Room]; ok {
                    delete(room, client.ID)
                    if len(room) == 0 {
                        delete(h.rooms, client.Room)
                    }
                }
                client.Conn.Close()
            }
            h.mu.Unlock()
            
            log.Printf("Client %s left room %s\n", client.Username, client.Room)
            
            h.broadcast <- Message{
                Type:      "system",
                Content:   client.Username + " left the room",
                Room:      client.Room,
                Timestamp: time.Now(),
            }
            
        case message := <-h.broadcast:
            h.mu.RLock()
            if room, ok := h.rooms[message.Room]; ok {
                for _, client := range room {
                    data, _ := json.Marshal(message)
                    if err := client.Conn.WriteMessage(websocket.TextMessage, data); err != nil {
                        log.Println("Write error:", err)
                    }
                }
            }
            h.mu.RUnlock()
        }
    }
}

func (h *Hub) HandleWebSocket(c *websocket.Conn) {
    username := c.Query("username", "Anonymous")
    room := c.Query("room", "general")
    
    client := &Client{
        ID:       uuid.New().String(),
        Username: username,
        Conn:     c,
        Room:     room,
    }
    
    h.register <- client
    
    defer func() {
        h.unregister <- client
    }()
    
    for {
        _, msg, err := c.ReadMessage()
        if err != nil {
            break
        }
        
        var incoming struct {
            Content string `json:"content"`
        }
        
        if err := json.Unmarshal(msg, &incoming); err != nil {
            continue
        }
        
        h.broadcast <- Message{
            Type:      "message",
            From:      client.Username,
            Content:   incoming.Content,
            Room:      client.Room,
            Timestamp: time.Now(),
        }
    }
}

func main() {
    hub := NewHub()
    go hub.Run()

    app := fiber.New()

    app.Use("/ws", func(c fiber.Ctx) error {
        if websocket.IsWebSocketUpgrade(c) {
            return c.Next()
        }
        return fiber.ErrUpgradeRequired
    })

    app.Get("/ws/chat", websocket.New(hub.HandleWebSocket))

    // API endpoints
    app.Get("/rooms", func(c fiber.Ctx) error {
        hub.mu.RLock()
        defer hub.mu.RUnlock()
        
        rooms := make([]map[string]interface{}, 0)
        for name, clients := range hub.rooms {
            rooms = append(rooms, map[string]interface{}{
                "name":   name,
                "users":  len(clients),
            })
        }
        return c.JSON(rooms)
    })

    log.Fatal(app.Listen(":3000"))
}
```

## HTML Client

```html
<!DOCTYPE html>
<html>
<head>
    <title>Chat</title>
</head>
<body>
    <div id="messages"></div>
    <input type="text" id="message" placeholder="Type a message...">
    <button onclick="sendMessage()">Send</button>

    <script>
        const username = prompt("Enter your username:");
        const room = "general";
        
        const ws = new WebSocket(`ws://localhost:3000/ws/chat?username=${username}&room=${room}`);
        
        ws.onmessage = function(event) {
            const message = JSON.parse(event.data);
            const div = document.createElement('div');
            
            if (message.type === 'system') {
                div.innerHTML = `<em>${message.content}</em>`;
            } else {
                div.innerHTML = `<strong>${message.from}:</strong> ${message.content}`;
            }
            
            document.getElementById('messages').appendChild(div);
        };
        
        function sendMessage() {
            const input = document.getElementById('message');
            ws.send(JSON.stringify({ content: input.value }));
            input.value = '';
        }
        
        ws.onclose = function() {
            console.log('Disconnected');
        };
    </script>
</body>
</html>
```

## Real-time Notifications

```go
package main

import (
    "encoding/json"
    "log"
    "sync"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/contrib/websocket"
)

type NotificationHub struct {
    clients map[uint][]*websocket.Conn // userID -> connections
    mu      sync.RWMutex
}

func NewNotificationHub() *NotificationHub {
    return &NotificationHub{
        clients: make(map[uint][]*websocket.Conn),
    }
}

func (h *NotificationHub) Subscribe(userID uint, conn *websocket.Conn) {
    h.mu.Lock()
    defer h.mu.Unlock()
    h.clients[userID] = append(h.clients[userID], conn)
}

func (h *NotificationHub) Unsubscribe(userID uint, conn *websocket.Conn) {
    h.mu.Lock()
    defer h.mu.Unlock()
    
    connections := h.clients[userID]
    for i, c := range connections {
        if c == conn {
            h.clients[userID] = append(connections[:i], connections[i+1:]...)
            break
        }
    }
    
    if len(h.clients[userID]) == 0 {
        delete(h.clients, userID)
    }
}

func (h *NotificationHub) SendToUser(userID uint, notification interface{}) {
    h.mu.RLock()
    defer h.mu.RUnlock()
    
    connections, ok := h.clients[userID]
    if !ok {
        return
    }
    
    data, _ := json.Marshal(notification)
    
    for _, conn := range connections {
        if err := conn.WriteMessage(websocket.TextMessage, data); err != nil {
            log.Println("Write error:", err)
        }
    }
}

func (h *NotificationHub) Broadcast(notification interface{}) {
    h.mu.RLock()
    defer h.mu.RUnlock()
    
    data, _ := json.Marshal(notification)
    
    for _, connections := range h.clients {
        for _, conn := range connections {
            conn.WriteMessage(websocket.TextMessage, data)
        }
    }
}

var notificationHub = NewNotificationHub()

func main() {
    app := fiber.New()

    app.Use("/ws", func(c fiber.Ctx) error {
        if websocket.IsWebSocketUpgrade(c) {
            return c.Next()
        }
        return fiber.ErrUpgradeRequired
    })

    // WebSocket endpoint
    app.Get("/ws/notifications", websocket.New(func(c *websocket.Conn) {
        // JWT থেকে user ID নিন
        userID := uint(1) // সিমুলেটেড
        
        notificationHub.Subscribe(userID, c)
        defer notificationHub.Unsubscribe(userID, c)
        
        // Keep connection alive
        for {
            _, _, err := c.ReadMessage()
            if err != nil {
                break
            }
        }
    }))

    // Send notification API
    app.Post("/notify/:userId", func(c fiber.Ctx) error {
        userID, _ := c.ParamsInt("userId")
        
        var notification map[string]interface{}
        c.Bind().JSON(&notification)
        
        notificationHub.SendToUser(uint(userID), notification)
        
        return c.JSON(fiber.Map{"message": "Notification sent"})
    })

    // Broadcast API
    app.Post("/broadcast", func(c fiber.Ctx) error {
        var notification map[string]interface{}
        c.Bind().JSON(&notification)
        
        notificationHub.Broadcast(notification)
        
        return c.JSON(fiber.Map{"message": "Broadcast sent"})
    })

    log.Fatal(app.Listen(":3000"))
}
```

## Live Dashboard

```go
package main

import (
    "encoding/json"
    "math/rand"
    "time"
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/contrib/websocket"
)

type DashboardData struct {
    CPU       float64 `json:"cpu"`
    Memory    float64 `json:"memory"`
    Requests  int     `json:"requests"`
    Errors    int     `json:"errors"`
    Timestamp int64   `json:"timestamp"`
}

func main() {
    app := fiber.New()

    app.Use("/ws", func(c fiber.Ctx) error {
        if websocket.IsWebSocketUpgrade(c) {
            return c.Next()
        }
        return fiber.ErrUpgradeRequired
    })

    app.Get("/ws/dashboard", websocket.New(func(c *websocket.Conn) {
        ticker := time.NewTicker(1 * time.Second)
        defer ticker.Stop()
        
        done := make(chan struct{})
        
        // Read goroutine (for disconnect detection)
        go func() {
            for {
                if _, _, err := c.ReadMessage(); err != nil {
                    close(done)
                    return
                }
            }
        }()
        
        for {
            select {
            case <-done:
                return
            case <-ticker.C:
                data := DashboardData{
                    CPU:       rand.Float64() * 100,
                    Memory:    rand.Float64() * 100,
                    Requests:  rand.Intn(1000),
                    Errors:    rand.Intn(10),
                    Timestamp: time.Now().Unix(),
                }
                
                jsonData, _ := json.Marshal(data)
                if err := c.WriteMessage(websocket.TextMessage, jsonData); err != nil {
                    return
                }
            }
        }
    }))

    app.Listen(":3000")
}
```

## Binary Data

```go
app.Get("/ws/binary", websocket.New(func(c *websocket.Conn) {
    for {
        messageType, message, err := c.ReadMessage()
        if err != nil {
            break
        }
        
        switch messageType {
        case websocket.TextMessage:
            // Text handling
            log.Println("Text:", string(message))
            
        case websocket.BinaryMessage:
            // Binary handling (images, files, etc.)
            log.Printf("Binary: %d bytes\n", len(message))
            
            // Process and send back
            processedData := processBinary(message)
            c.WriteMessage(websocket.BinaryMessage, processedData)
        }
    }
}))
```

## Ping/Pong

```go
app.Get("/ws/ping", websocket.New(func(c *websocket.Conn) {
    // Set handlers
    c.SetPingHandler(func(appData string) error {
        log.Println("Ping received")
        return c.WriteControl(websocket.PongMessage, []byte(appData), time.Now().Add(time.Second))
    })
    
    c.SetPongHandler(func(appData string) error {
        log.Println("Pong received")
        return nil
    })
    
    // Send periodic pings
    go func() {
        ticker := time.NewTicker(30 * time.Second)
        defer ticker.Stop()
        
        for range ticker.C {
            if err := c.WriteControl(websocket.PingMessage, nil, time.Now().Add(time.Second)); err != nil {
                return
            }
        }
    }()
    
    // Main loop
    for {
        _, msg, err := c.ReadMessage()
        if err != nil {
            break
        }
        c.WriteMessage(websocket.TextMessage, msg)
    }
}))
```

## WebSocket with Auth

```go
app.Get("/ws/protected", websocket.New(func(c *websocket.Conn) {
    // First message must be auth
    c.SetReadDeadline(time.Now().Add(10 * time.Second))
    
    _, authMsg, err := c.ReadMessage()
    if err != nil {
        c.WriteJSON(map[string]string{"error": "Auth timeout"})
        return
    }
    
    var auth struct {
        Token string `json:"token"`
    }
    
    if err := json.Unmarshal(authMsg, &auth); err != nil {
        c.WriteJSON(map[string]string{"error": "Invalid auth message"})
        return
    }
    
    claims, err := ValidateToken(auth.Token)
    if err != nil {
        c.WriteJSON(map[string]string{"error": "Invalid token"})
        return
    }
    
    // Clear deadline
    c.SetReadDeadline(time.Time{})
    
    // Send success
    c.WriteJSON(map[string]interface{}{
        "type":    "auth_success",
        "user_id": claims.UserID,
    })
    
    // Continue with authenticated connection
    for {
        _, msg, err := c.ReadMessage()
        if err != nil {
            break
        }
        
        // Handle authenticated messages
        log.Printf("User %d: %s\n", claims.UserID, msg)
    }
}))
```

## সারসংক্ষেপ

| Feature | Description |
|---------|-------------|
| `websocket.New()` | Handler তৈরি |
| `ReadMessage()` | Message পড়া |
| `WriteMessage()` | Message পাঠানো |
| `WriteJSON()` | JSON পাঠানো |
| `Close()` | Connection বন্ধ |
| Ping/Pong | Keep-alive |

---

[← আগের: অথেনটিকেশন](04-authentication.md) | [পরবর্তী: ডিপ্লয়মেন্ট →](06-deployment.md)
