# WebSockets - WebSocket constructor, send/receive, events, reconnection

## Introduction

WebSockets provide full-duplex communication channels over a single TCP connection, enabling real-time, bidirectional data transfer between client and server. Unlike HTTP's request-response model, WebSockets allow the server to push data to clients without polling. This makes them essential for applications requiring low-latency, real-time updates such as chat applications, live dashboards, collaborative editing, and online gaming.

## WebSocket constructor

### What It Is

The WebSocket constructor creates a new WebSocket connection to a specified URL. It accepts the server URL (ws:// or wss:// protocol) and optional subprotocols.

```javascript
const socket = new WebSocket(url, protocols)
```

### Why It Is Important

The constructor initializes the WebSocket handshake, upgrading the HTTP connection to a WebSocket connection. It is the entry point for all WebSocket communication and handles the initial connection establishment, including the upgrade request and response.

### How It Works Internally

The browser sends an HTTP upgrade request with specific headers (Upgrade: websocket, Connection: Upgrade, Sec-WebSocket-Key, Sec-WebSocket-Version). The server responds with HTTP 101 Switching Protocols and a Sec-WebSocket-Accept header. After the handshake, the connection transitions from HTTP to the WebSocket protocol, and data frames can flow in both directions.

```javascript
// Handshake process:
// Client -> Server: GET /chat HTTP/1.1
//                   Host: example.com
//                   Upgrade: websocket
//                   Connection: Upgrade
//                   Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
//                   Sec-WebSocket-Version: 13

// Server -> Client: HTTP/1.1 101 Switching Protocols
//                   Upgrade: websocket
//                   Connection: Upgrade
//                   Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### Syntax

```javascript
// Basic connection
const socket = new WebSocket('wss://api.example.com/ws')

// With subprotocols
const socket = new WebSocket('wss://chat.example.com', ['json', 'msgpack'])

// With multiple subprotocols (array)
const socket = new WebSocket('wss://game.example.com', ['v1.protocol', 'v2.protocol'])
```

### Beginner Examples

```javascript
// Create and connect
const socket = new WebSocket('wss://echo.websocket.org')

// Connection opened
socket.addEventListener('open', function(event) {
  console.log('Connected to server')
  socket.send('Hello Server!')
})

// Listen for messages
socket.addEventListener('message', function(event) {
  console.log('Message from server:', event.data)
})

// Listen for errors
socket.addEventListener('error', function(event) {
  console.error('WebSocket error:', event)
})

// Listen for connection close
socket.addEventListener('close', function(event) {
  console.log('Disconnected:', event.code, event.reason)
})

// Send a message
socket.send('Hello!')

// Close the connection
socket.close(1000, 'Client closing')
```

### Intermediate Examples

```javascript
// Chat application client
class ChatClient {
  constructor(url, username) {
    this.url = url
    this.username = username
    this.socket = null
    this.reconnectAttempts = 0
    this.maxReconnectAttempts = 5
    this.listeners = {}
  }

  connect() {
    this.socket = new WebSocket(this.url)

    this.socket.onopen = () => {
      console.log('Connected to chat server')
      this.reconnectAttempts = 0

      // Authenticate
      this.send({ type: 'auth', username: this.username })

      this.emit('connected')
    }

    this.socket.onmessage = (event) => {
      try {
        const message = JSON.parse(event.data)
        this.emit(message.type, message.data)
        this.emit('message', message)
      } catch (error) {
        console.error('Failed to parse message:', error)
      }
    }

    this.socket.onclose = (event) => {
      this.emit('disconnected', { code: event.code, reason: event.reason })

      if (!event.wasClean) {
        this.reconnect()
      }
    }

    this.socket.onerror = (error) => {
      console.error('Chat WebSocket error:', error)
      this.emit('error', error)
    }
  }

  send(data) {
    if (this.socket && this.socket.readyState === WebSocket.OPEN) {
      this.socket.send(JSON.stringify(data))
    } else {
      console.error('Cannot send: socket not open')
    }
  }

  reconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      this.emit('reconnect_failed')
      return
    }

    this.reconnectAttempts++
    const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000)
    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`)
    this.emit('reconnecting', { attempt: this.reconnectAttempts, delay })

    setTimeout(() => this.connect(), delay)
  }

  disconnect() {
    this.maxReconnectAttempts = 0
    if (this.socket) {
      this.socket.close(1000, 'Client disconnecting')
    }
  }

  on(event, callback) {
    if (!this.listeners[event]) {
      this.listeners[event] = []
    }
    this.listeners[event].push(callback)
  }

  emit(event, data) {
    const eventListeners = this.listeners[event]
    if (eventListeners) {
      eventListeners.forEach(callback => callback(data))
    }
  }
}

// Usage
const chat = new ChatClient('wss://chat.example.com/ws', 'Alice')

chat.on('connected', () => console.log('Ready to chat!'))
chat.on('message', (msg) => {
  if (msg.type === 'chat') {
    displayMessage(msg.data.user, msg.data.text)
  }
})
chat.on('reconnecting', ({ attempt }) => showStatus(`Reconnecting (${attempt})...`))

chat.connect()

function sendMessage(text) {
  chat.send({ type: 'chat', data: { text, timestamp: Date.now() } })
}
```

### Advanced Examples

```javascript
// WebSocket with heartbeat/ping-pong
class HeartbeatWebSocket {
  constructor(url, options = {}) {
    this.url = url
    this.heartbeatInterval = options.heartbeatInterval || 30000
    this.heartbeatTimeout = options.heartbeatTimeout || 5000
    this.reconnectDelay = options.reconnectDelay || 1000
    this.maxReconnectDelay = options.maxReconnectDelay || 30000
    this.subprotocols = options.protocols || []

    this.socket = null
    this.isConnected = false
    this.isManualClose = false
    this.heartbeatTimer = null
    this.heartbeatResponseTimer = null
    this.reconnectTimer = null
    this.listeners = {}
    this.pendingMessages = []
    this.messageId = 0
  }

  connect() {
    this.isManualClose = false

    if (this.subprotocols.length) {
      this.socket = new WebSocket(this.url, this.subprotocols)
    } else {
      this.socket = new WebSocket(this.url)
    }

    this.socket.onopen = () => {
      this.isConnected = true
      this.flushPendingMessages()
      this.startHeartbeat()
      this.emit('open')
    }

    this.socket.onmessage = (event) => {
      this.handleMessage(event)
    }

    this.socket.onclose = (event) => {
      this.isConnected = false
      this.stopHeartbeat()
      this.emit('close', event)

      if (!this.isManualClose) {
        this.scheduleReconnect()
      }
    }

    this.socket.onerror = (error) => {
      this.emit('error', error)
    }
  }

  handleMessage(event) {
    try {
      const data = JSON.parse(event.data)

      // Handle acknowledgment
      if (data.type === 'ack' && data.id) {
        this.emit(`ack:${data.id}`, data)
        return
      }

      // Handle heartbeat response
      if (data.type === 'pong') {
        clearTimeout(this.heartbeatResponseTimer)
        return
      }

      // Emit typed event
      if (data.type) {
        this.emit(data.type, data.payload)
      }

      this.emit('message', data)
    } catch {
      this.emit('raw', event.data)
    }
  }

  send(type, payload = {}, options = {}) {
    const message = {
      id: ++this.messageId,
      type,
      payload,
      timestamp: Date.now()
    }

    const data = JSON.stringify(message)

    if (this.isConnected && this.socket.readyState === WebSocket.OPEN) {
      this.socket.send(data)
      return message.id
    }

    if (options.queue) {
      this.pendingMessages.push(data)
    } else {
      console.warn('WebSocket not connected, dropping message')
    }

    return null
  }

  sendWithAck(type, payload = {}, timeout = 5000) {
    return new Promise((resolve, reject) => {
      const id = this.send(type, payload, { queue: true })

      const ackTimer = setTimeout(() => {
        this.off(`ack:${id}`)
        reject(new Error('Acknowledgment timeout'))
      }, timeout)

      this.on(`ack:${id}`, (data) => {
        clearTimeout(ackTimer)
        resolve(data)
      })
    })
  }

  startHeartbeat() {
    this.heartbeatTimer = setInterval(() => {
      if (this.isConnected) {
        this.socket.send(JSON.stringify({ type: 'ping' }))

        this.heartbeatResponseTimer = setTimeout(() => {
          console.warn('Heartbeat timeout - closing connection')
          this.socket.close()
        }, this.heartbeatTimeout)
      }
    }, this.heartbeatInterval)
  }

  stopHeartbeat() {
    clearInterval(this.heartbeatTimer)
    clearTimeout(this.heartbeatResponseTimer)
  }

  flushPendingMessages() {
    while (this.pendingMessages.length) {
      const msg = this.pendingMessages.shift()
      this.socket.send(msg)
    }
  }

  scheduleReconnect() {
    const delay = Math.min(
      this.reconnectDelay * Math.pow(2, this._reconnectAttempt || 0),
      this.maxReconnectDelay
    )
    this._reconnectAttempt = (this._reconnectAttempt || 0) + 1

    this.emit('reconnecting', { attempt: this._reconnectAttempt, delay })

    this.reconnectTimer = setTimeout(() => {
      this.connect()
    }, delay)
  }

  close(code = 1000, reason = 'Client close') {
    this.isManualClose = true
    this.stopHeartbeat()
    clearTimeout(this.reconnectTimer)

    if (this.socket) {
      this.socket.close(code, reason)
    }
  }

  on(event, callback) {
    if (!this.listeners[event]) {
      this.listeners[event] = []
    }
    this.listeners[event].push(callback)
    return () => this.off(event, callback)
  }

  off(event, callback) {
    const eventListeners = this.listeners[event]
    if (eventListeners) {
      this.listeners[event] = eventListeners.filter(cb => cb !== callback)
    }
  }

  emit(event, data) {
    const eventListeners = this.listeners[event]
    if (eventListeners) {
      eventListeners.forEach(callback => {
        try {
          callback(data)
        } catch (error) {
          console.error(`Error in ${event} handler:`, error)
        }
      })
    }
  }
}

// Server-side WebSocket (Node.js with ws library)
const WebSocket = require('ws')

class WebSocketServer {
  constructor(port) {
    this.wss = new WebSocket.Server({ port })
    this.clients = new Map()

    this.wss.on('connection', (ws, req) => {
      const clientId = generateId()
      const clientInfo = {
        id: clientId,
        ws,
        ip: req.socket.remoteAddress,
        connectedAt: Date.now(),
        authenticated: false,
        userId: null
      }

      this.clients.set(clientId, clientInfo)
      this.emit('connect', clientInfo)

      ws.on('message', (data) => {
        try {
          const message = JSON.parse(data)
          this.handleMessage(clientInfo, message)
        } catch {
          ws.send(JSON.stringify({ type: 'error', payload: { message: 'Invalid JSON' } }))
        }
      })

      ws.on('close', (code, reason) => {
        this.clients.delete(clientId)
        this.emit('disconnect', clientInfo)
      })

      ws.on('error', (error) => {
        console.error(`WebSocket error for client ${clientId}:`, error)
      })

      // Send welcome message
      ws.send(JSON.stringify({
        type: 'welcome',
        payload: { clientId, timestamp: Date.now() }
      }))
    })
  }

  handleMessage(client, message) {
    switch (message.type) {
      case 'auth':
        this.authenticate(client, message.payload)
        break
      case 'ping':
        client.ws.send(JSON.stringify({ type: 'pong' }))
        break
      case 'chat':
        this.broadcast('chat', {
          userId: client.userId,
          text: message.payload.text,
          timestamp: Date.now()
        }, client.id)
        break
      default:
        this.emit('message', { client, message })
    }
  }

  authenticate(client, { token }) {
    try {
      const user = verifyToken(token)
      client.authenticated = true
      client.userId = user.id
      client.ws.send(JSON.stringify({
        type: 'auth_success',
        payload: { userId: user.id }
      }))
    } catch {
      client.ws.send(JSON.stringify({
        type: 'auth_error',
        payload: { message: 'Invalid token' }
      }))
    }
  }

  broadcast(type, payload, excludeClientId = null) {
    const message = JSON.stringify({ type, payload, timestamp: Date.now() })

    this.clients.forEach((client) => {
      if (client.id !== excludeClientId && client.ws.readyState === WebSocket.OPEN) {
        client.ws.send(message)
      }
    })
  }

  sendToUser(userId, type, payload) {
    const message = JSON.stringify({ type, payload })

    this.clients.forEach((client) => {
      if (client.userId === userId && client.ws.readyState === WebSocket.OPEN) {
        client.ws.send(message)
      }
    })
  }

  broadcastExcept(type, payload, excludeUserIds = []) {
    const message = JSON.stringify({ type, payload })

    this.clients.forEach((client) => {
      if (!excludeUserIds.includes(client.userId) && client.ws.readyState === WebSocket.OPEN) {
        client.ws.send(message)
      }
    })
  }

  getConnectedClients() {
    return [...this.clients.values()].map(c => ({
      id: c.id,
      userId: c.userId,
      authenticated: c.authenticated,
      connectedAt: c.connectedAt
    }))
  }
}
```

### Real-World Use Cases

- **Live chat applications**: Real-time messaging between users
- **Collaborative editing**: Google Docs-style real-time collaboration
- **Live dashboards**: Real-time monitoring and analytics dashboards
- **Online gaming**: Multiplayer game state synchronization
- **Financial tickers**: Real-time stock prices and cryptocurrency updates
- **Notifications**: Push notifications without polling
- **Live sports updates**: Real-time scores and statistics
- **IoT devices**: Real-time sensor data streaming
- **Live streaming**: Real-time captions and overlays

### Common Mistakes

```javascript
// Mistake: Missing error handling for JSON.parse
socket.onmessage = (event) => {
  const data = JSON.parse(event.data) // Can throw!
}

// Mistake: Sending non-string data without serialization
socket.send({ message: 'hello' }) // [object Object] sent!
// Correct: socket.send(JSON.stringify({ message: 'hello' }))

// Mistake: Not handling connection drops
// Messages sent while disconnected are silently lost

// Mistake: No reconnection logic
// User has to refresh the page to reconnect

// Mistake: Not checking readyState before sending
socket.send('data') // Fails if not OPEN

// Mistake: Forgetting to use wss:// in production
// Unencrypted ws:// exposes all traffic

// Mistake: No heartbeat mechanism
// Half-open connections are not detected until next send attempt
```

### Best Practices

```javascript
// Always check readyState before sending
if (socket.readyState === WebSocket.OPEN) {
  socket.send(data)
}

// Use JSON for structured messages
socket.send(JSON.stringify({ type: 'action', payload: { id: 1 } }))

// Implement exponential backoff reconnection
function reconnect(attempt) {
  const delay = Math.min(1000 * Math.pow(2, attempt), 30000)
  setTimeout(() => connect(), delay)
}

// Use heartbeat to detect half-open connections
setInterval(() => {
  if (socket.readyState === WebSocket.OPEN) {
    socket.send(JSON.stringify({ type: 'ping' }))
  }
}, 30000)

// Always close cleanly
function cleanup() {
  socket.close(1000, 'Page unload')
}
window.addEventListener('beforeunload', cleanup)

// Use wss:// in production
const WS_URL = location.protocol === 'https:' ? 'wss://' : 'ws://' + location.host + '/ws'
```

### Performance Considerations

- WebSocket connections consume TCP socket resources (limit ~6-8 per browser)
- Each connection has memory overhead for buffering and state
- Message fragmentation occurs for messages > 16KB (configurable)
- Binary messages (ArrayBuffer/Blob) are more efficient than text for large data
- Compression (permessage-deflate) reduces bandwidth but increases CPU usage
- Many concurrent connections require careful server resource planning
- Message batching can reduce overhead for high-frequency updates
- Consider using SharedWorker for WebSocket connections shared across tabs

### Interview Questions

**Q: What is the difference between WebSocket and HTTP?**

A: HTTP is request-response (half-duplex), while WebSocket is bidirectional (full-duplex). HTTP requires a new connection for each request (or connection reuse with keep-alive), while WebSocket maintains a persistent connection. HTTP has headers overhead on every request, while WebSocket has minimal framing overhead after the initial upgrade.

**Q: How does WebSocket handle message fragmentation?**

A: WebSocket protocol supports message fragmentation using frames. A message can be split into multiple frames, each with a FIN bit indicating if it's the final frame. This allows streaming large messages without buffering the entire message in memory. The application layer receives the complete reassembled message.

**Q: How do you handle authentication in WebSocket connections?**

A: Authenticate during the WebSocket handshake using URL parameters, cookies (sent with the upgrade request), or a custom subprotocol. After connection, send an authentication message. Common patterns include: validating a JWT from the handshake URL query, using cookies with SameSite, or exchanging auth tokens after connection. Token validation during handshake is preferred to avoid unauthenticated connections consuming resources.

### Coding Challenges

```javascript
// Challenge 1: WebSocket message queue with delivery guarantees
class MessageQueueWS {
  constructor(url) {
    this.url = url
    this.queue = []
    this.socket = null
    this.connecting = false
  }

  async ensureConnected() {
    if (this.socket && this.socket.readyState === WebSocket.OPEN) return

    if (this.connecting) {
      await new Promise(resolve => {
        const check = () => {
          if (this.socket?.readyState === WebSocket.OPEN) resolve()
          else setTimeout(check, 50)
        }
        check()
      })
      return
    }

    this.connecting = true
    return new Promise((resolve, reject) => {
      this.socket = new WebSocket(this.url)
      this.socket.onopen = () => { this.connecting = false; this.flush(); resolve() }
      this.socket.onerror = (e) => { this.connecting = false; reject(e) }
    })
  }

  async send(data) {
    if (this.socket?.readyState === WebSocket.OPEN) {
      this.socket.send(data)
    } else {
      this.queue.push(data)
      await this.ensureConnected()
    }
  }

  flush() {
    while (this.queue.length) {
      this.socket.send(this.queue.shift())
    }
  }
}

// Challenge 2: WebSocket rate limiter
function createRateLimitedWS(url, maxMessagesPerSecond = 10) {
  const socket = new WebSocket(url)
  const queue = []
  let lastSendTime = 0
  const minInterval = 1000 / maxMessagesPerSecond

  const sendInterval = setInterval(() => {
    const now = Date.now()
    while (queue.length && now - lastSendTime >= minInterval) {
      socket.send(queue.shift())
      lastSendTime = now
    }
  }, minInterval)

  return {
    send: (data) => queue.push(data),
    close: () => { clearInterval(sendInterval); socket.close() },
    socket
  }
}

// Challenge 3: Multiplex WebSocket connections
class MultiplexedWS {
  constructor(url) {
    this.socket = new WebSocket(url)
    this.channels = new Map()

    this.socket.onmessage = (event) => {
      const { channel, data } = JSON.parse(event.data)
      const handlers = this.channels.get(channel)
      if (handlers) {
        handlers.forEach(h => h(data))
      }
    }
  }

  subscribe(channel, handler) {
    if (!this.channels.has(channel)) {
      this.channels.set(channel, [])
      this.socket.send(JSON.stringify({ type: 'subscribe', channel }))
    }
    this.channels.get(channel).push(handler)
    return () => this.unsubscribe(channel, handler)
  }

  publish(channel, data) {
    this.socket.send(JSON.stringify({ type: 'publish', channel, data }))
  }

  unsubscribe(channel, handler) {
    const handlers = this.channels.get(channel)
    if (handlers) {
      const index = handlers.indexOf(handler)
      if (index > -1) handlers.splice(index, 1)
    }
  }
}
```

### Related Topics

- `Fetch API` - HTTP-based alternative for request-response patterns
- `Server-Sent Events` - Unidirectional server-to-client streaming
- `Socket.IO` - WebSocket library with fallbacks and rooms
- `Service Workers` - Can intercept WebSocket connections for offline support
- `Authentication` - WebSocket authentication patterns
- `Real-time applications` - Architecture patterns for real-time systems
- `TCP/IP` - Underlying transport protocol
- `HTTP/2` - Server push as alternative unidirectional approach
