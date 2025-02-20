# Real-Time Chat Application

## Overview
This is a real-time chat application built using **Spring Boot, WebSockets (STOMP), and SockJS** for real-time messaging. The front-end is developed using **HTML, JavaScript, and Bootstrap**. The application allows multiple users to send and receive messages in real time.

## Features
- Real-time messaging using WebSockets and STOMP
- User-friendly interface with Bootstrap styling
- Automatic scrolling in the chat window
- Message broadcasting to all connected users
- Easy integration with a backend API

## Technologies Used
- **Backend:** Spring Boot, WebSockets (STOMP), SockJS
- **Frontend:** HTML, JavaScript, Bootstrap
- **Messaging Protocol:** STOMP (Simple Text Oriented Messaging Protocol)

## Project Structure
```
com.chat.application
│── config
│   └── WebSocketConfig.java    # Configures WebSocket and STOMP
│── controller
│   └── ChatController.java     # Handles chat message endpoints
│── model
│   └── ChatMessage.java        # Represents chat message model
│── resources
│   └── static/chat.html        # Frontend UI for chat
```

## Code Explanation
### 1. WebSocket Configuration - `WebSocketConfig.java`
This class sets up WebSockets with STOMP support.
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/chat").setAllowedOrigins("http://localhost:8080")
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic"); // Enables topic-based messaging
        registry.setApplicationDestinationPrefixes("/app");
    }
}
```
- `@EnableWebSocketMessageBroker` enables WebSocket messaging with STOMP.
- `registerStompEndpoints` registers `/chat` as the WebSocket endpoint, enabling SockJS for fallback options.
- `configureMessageBroker` configures a simple broker on `/topic` for broadcasting messages and sets `/app` as the prefix for client messages.

### 2. Chat Controller - `ChatController.java`
Handles incoming messages and broadcasts them to all connected clients.
```java
@Controller
public class ChatController {
    @MessageMapping("/sendMessage")
    @SendTo("/topic/messages")
    public ChatMessage sendMessage(ChatMessage message) {
        return message;
    }

    @GetMapping("/chat")
    public String chat() {
        return "chat";
    }
}
```
- `@MessageMapping("/sendMessage")` maps messages sent to `/app/sendMessage`.
- `@SendTo("/topic/messages")` broadcasts the received message to all subscribers of `/topic/messages`.
- `@GetMapping("/chat")` returns the `chat.html` page.

### 3. Chat Model - `ChatMessage.java`
Defines the chat message structure.
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ChatMessage {
    private Long id;
    private String sender;
    private String content;
}
```
- Uses Lombok annotations to generate boilerplate code (`@Data`, `@NoArgsConstructor`, `@AllArgsConstructor`).
- The model contains an ID, sender name, and message content.

### 4. Frontend - `chat.html`
Handles the UI and WebSocket client logic.
```html
<script>
    var stompClient = null;

    function connect() {
        var socket = new SockJS('/chat');
        stompClient = Stomp.over(socket);
        stompClient.connect({}, function (frame) {
            stompClient.subscribe('/topic/messages', function (message) {
                showMessage(JSON.parse(message.body));
            });
        });
    }

    function sendMessage() {
        var sender = document.getElementById('senderInput').value.trim();
        var content = document.getElementById('messageInput').value.trim();

        if (stompClient && stompClient.connected) {
            stompClient.send("/app/sendMessage", {}, JSON.stringify({sender: sender, content: content}));
        }
    }

    window.onload = connect;
</script>
```
- `connect()` establishes a WebSocket connection and subscribes to `/topic/messages`.
- `sendMessage()` sends user input to the `/app/sendMessage` endpoint.
- Messages are displayed dynamically in the chat window.

## How to Run
1. Clone the repository:
   ```sh
   git clone https://github.com/your-repo-name.git
   ```
2. Navigate to the project directory:
   ```sh
   cd realtime-chat-application
   ```
3. Build and run the application:
   ```sh
   mvn spring-boot:run
   ```
4. Open the browser and go to:
   ```
   http://localhost:8080/chat
   ```

## API Endpoints
| Endpoint  | Description |
|-----------|------------|
| `/chat`   | WebSocket connection endpoint |
| `/app/sendMessage` | Endpoint to send messages |
| `/topic/messages`  | Topic for receiving messages |

## WebSocket Flow
1. The frontend establishes a WebSocket connection to `/chat`.
2. Messages are sent to `/app/sendMessage`.
3. The server processes and forwards messages to `/topic/messages`.
4. All connected clients receive the message in real time.

## How to Revise and Explain in an Interview
- **Understanding WebSockets:** Explain how WebSockets provide a persistent connection for real-time communication.
- **STOMP Protocol:** Describe how STOMP is used as a messaging protocol over WebSockets.
- **Spring Boot Integration:** Explain how WebSocketConfig.java sets up the message broker.
- **Message Handling:** Demonstrate how messages are mapped, processed, and broadcasted.
- **Client-Side Communication:** Discuss how SockJS and Stomp.js handle real-time communication.

/*
The answer for the revision
*/
1. Understanding WebSockets
WebSockets provide a persistent, full-duplex communication channel between a client and a server, enabling real-time messaging. Unlike traditional HTTP, which follows a request-response model, WebSockets maintain a continuous connection, reducing latency and improving efficiency.

2. STOMP Protocol
STOMP (Simple Text Oriented Messaging Protocol) is a lightweight protocol used over WebSockets for messaging. It defines how clients and servers communicate, enabling message routing, subscriptions, and broadcasting. STOMP simplifies handling WebSocket connections in Spring Boot by providing a structured way to send and receive messages.

3. Spring Boot Integration
WebSocketConfig.java configures WebSockets and STOMP in the Spring Boot backend.
@EnableWebSocketMessageBroker enables WebSocket message handling.
The registerStompEndpoints() method sets up the WebSocket endpoint /chat and enables SockJS as a fallback.
The configureMessageBroker() method sets up a message broker with /topic for broadcasting and /app for sending messages.

5. Message Handling
The ChatController.java listens for messages sent to /app/sendMessage using @MessageMapping.
The @SendTo("/topic/messages") annotation ensures messages are broadcasted to all subscribers.
Messages follow this path:
Client sends message → Backend processes it → Message is broadcasted to subscribers

7. Client-Side Communication
SockJS and Stomp.js handle WebSocket connections in the frontend.
SockJS provides a fallback mechanism when WebSockets are not supported.
Stomp.js enables clients to subscribe to /topic/messages and send messages to /app/sendMessage.
The frontend dynamically updates the chat window when new messages arrive.

