# Authentication - JWT, token-based auth, localStorage vs cookies, refresh tokens

## Introduction

Authentication is the process of verifying the identity of a user or system. In modern web applications, token-based authentication, particularly using JSON Web Tokens (JWT), has become the dominant approach. This system involves issuing a token upon successful login, which the client then presents with subsequent requests to prove identity. The choice of where to store these tokens (localStorage vs cookies) has significant security implications, and refresh tokens provide a mechanism for maintaining sessions without repeatedly asking for credentials.

## JWT tokens

### What It Is

JSON Web Token (JWT) is an open standard (RFC 7519) for securely transmitting information between parties as a JSON object. JWTs are digitally signed, typically using HMAC or RSA. They consist of three Base64-encoded parts separated by dots: header, payload, and signature.

```javascript
// JWT structure
// header.payload.signature
// eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

// Decoded header
{ "alg": "HS256", "typ": "JWT" }

// Decoded payload
{ "sub": "1234567890", "name": "John Doe", "iat": 1516239022 }

// Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

### Why It Is Important

JWTs enable stateless authentication, meaning servers don't need to store session data. All user information (claims) is embedded within the token itself, making it suitable for distributed systems, microservices, and single-page applications. JWTs are compact, URL-safe, and can be used across different domains.

### How It Works Internally

The server creates a JWT by encoding a header (algorithm and token type) and payload (claims), then signing the concatenation with a secret key. The client stores this token and sends it in the Authorization header. The server verifies the signature, checks expiration, and extracts user identity from the claims.

```javascript
// Authentication flow:
// 1. User submits credentials
// 2. Server validates credentials
// 3. Server creates JWT with claims and expiration
// 4. Server returns JWT to client
// 5. Client stores token (localStorage/cookie)
// 6. Client sends token with each request (Authorization header)
// 7. Server verifies token signature and expiration
// 8. Server processes request with user identity from claims
```

### Syntax

```javascript
// JWT claims (standard registered claims):
// iss (Issuer)     - Who issued the token
// sub (Subject)    - Who the token describes (typically user ID)
// aud (Audience)   - Who the token is intended for
// exp (Expiration) - Unix timestamp when token expires
// nbf (Not Before) - Token not valid before this time
// iat (Issued At)  - When the token was issued
// jti (JWT ID)     - Unique identifier for the token

// Creating JWT (using jsonwebtoken library)
const jwt = require('jsonwebtoken')
const token = jwt.sign(
  { userId: user.id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '1h', issuer: 'myapp' }
)

// Verifying JWT
try {
  const decoded = jwt.verify(token, process.env.JWT_SECRET)
  console.log(decoded.userId)
} catch (error) {
  console.error('Invalid token:', error.message)
}
```

### Beginner Examples

```javascript
// Frontend login and token storage
async function login(email, password) {
  const response = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  })

  if (!response.ok) {
    throw new Error('Login failed')
  }

  const { token, user } = await response.json()

  // Store token
  localStorage.setItem('auth_token', token)
  localStorage.setItem('user', JSON.stringify(user))

  return user
}

// Authenticated request
async function fetchUserProfile() {
  const token = localStorage.getItem('auth_token')

  const response = await fetch('/api/user/profile', {
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  })

  if (!response.ok) {
    if (response.status === 401) {
      // Token expired or invalid - redirect to login
      logout()
    }
    throw new Error('Failed to fetch profile')
  }

  return response.json()
}

// Logout
function logout() {
  localStorage.removeItem('auth_token')
  localStorage.removeItem('user')
  window.location.href = '/login'
}

// Decode JWT payload (without verification - frontend only)
function decodeJWTPayload(token) {
  try {
    const payload = token.split('.')[1]
    return JSON.parse(atob(payload.replace(/-/g, '+').replace(/_/g, '/')))
  } catch {
    return null
  }
}
```

### Intermediate Examples

```javascript
// Complete auth service
class AuthService {
  constructor() {
    this.tokenKey = 'auth_token'
    this.refreshTokenKey = 'refresh_token'
    this.userKey = 'auth_user'
  }

  async login(credentials) {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    })

    if (!response.ok) {
      const error = await response.json()
      throw new AuthError(error.message, response.status)
    }

    const { accessToken, refreshToken, user } = await response.json()
    this.setTokens(accessToken, refreshToken)
    this.setUser(user)

    return user
  }

  async register(userData) {
    const response = await fetch('/api/auth/register', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    })

    if (!response.ok) {
      throw new AuthError('Registration failed', response.status)
    }

    return response.json()
  }

  setTokens(accessToken, refreshToken) {
    localStorage.setItem(this.tokenKey, accessToken)
    if (refreshToken) {
      localStorage.setItem(this.refreshTokenKey, refreshToken)
    }
  }

  getAccessToken() {
    return localStorage.getItem(this.tokenKey)
  }

  getRefreshToken() {
    return localStorage.getItem(this.refreshTokenKey)
  }

  setUser(user) {
    localStorage.setItem(this.userKey, JSON.stringify(user))
  }

  getUser() {
    try {
      return JSON.parse(localStorage.getItem(this.userKey))
    } catch {
      return null
    }
  }

  isAuthenticated() {
    const token = this.getAccessToken()
    if (!token) return false

    try {
      const payload = this.decodeToken(token)
      return payload.exp * 1000 > Date.now()
    } catch {
      return false
    }
  }

  isTokenExpired(token) {
    try {
      const payload = this.decodeToken(token)
      return payload.exp * 1000 < Date.now()
    } catch {
      return true
    }
  }

  decodeToken(token) {
    const payload = token.split('.')[1]
    const decoded = atob(payload.replace(/-/g, '+').replace(/_/g, '/'))
    return JSON.parse(decoded)
  }

  async refreshToken() {
    const refreshToken = this.getRefreshToken()
    if (!refreshToken) {
      throw new AuthError('No refresh token available')
    }

    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ refreshToken })
    })

    if (!response.ok) {
      this.logout()
      throw new AuthError('Token refresh failed')
    }

    const { accessToken, refreshToken: newRefreshToken } = await response.json()
    this.setTokens(accessToken, newRefreshToken)
    return accessToken
  }

  async fetchWithAuth(url, options = {}) {
    const token = this.getAccessToken()

    if (this.isTokenExpired(token)) {
      await this.refreshToken()
    }

    const headers = {
      ...options.headers,
      'Authorization': `Bearer ${this.getAccessToken()}`
    }

    const response = await fetch(url, { ...options, headers })

    if (response.status === 401) {
      // Try refreshing once more
      await this.refreshToken()
      headers['Authorization'] = `Bearer ${this.getAccessToken()}`
      return fetch(url, { ...options, headers })
    }

    return response
  }

  logout() {
    localStorage.removeItem(this.tokenKey)
    localStorage.removeItem(this.refreshTokenKey)
    localStorage.removeItem(this.userKey)
    // Optionally notify server
    fetch('/api/auth/logout', { method: 'POST' }).catch(() => {})
  }
}

class AuthError extends Error {
  constructor(message, status) {
    super(message)
    this.name = 'AuthError'
    this.status = status
  }
}
```

### Advanced Examples

```javascript
// Server-side JWT implementation (Node.js/Express)
const jwt = require('jsonwebtoken')
const bcrypt = require('bcrypt')

class AuthController {
  constructor(userRepository) {
    this.userRepository = userRepository
    this.accessTokenSecret = process.env.JWT_ACCESS_SECRET
    this.refreshTokenSecret = process.env.JWT_REFRESH_SECRET
    this.accessTokenExpiry = '15m'
    this.refreshTokenExpiry = '7d'
  }

  generateAccessToken(user) {
    return jwt.sign(
      {
        sub: user.id,
        role: user.role,
        permissions: user.permissions
      },
      this.accessTokenSecret,
      { expiresIn: this.accessTokenExpiry }
    )
  }

  generateRefreshToken(user) {
    return jwt.sign(
      { sub: user.id, tokenId: require('crypto').randomUUID() },
      this.refreshTokenSecret,
      { expiresIn: this.refreshTokenExpiry }
    )
  }

  async login(req, res) {
    const { email, password } = req.body

    // Validate input
    if (!email || !password) {
      return res.status(400).json({ message: 'Email and password required' })
    }

    // Find user
    const user = await this.userRepository.findByEmail(email)
    if (!user) {
      return res.status(401).json({ message: 'Invalid credentials' })
    }

    // Verify password
    const validPassword = await bcrypt.compare(password, user.passwordHash)
    if (!validPassword) {
      return res.status(401).json({ message: 'Invalid credentials' })
    }

    // Generate tokens
    const accessToken = this.generateAccessToken(user)
    const refreshToken = this.generateRefreshToken(user)

    // Store refresh token (for revocation)
    await this.userRepository.storeRefreshToken(user.id, refreshToken)

    res.json({
      accessToken,
      refreshToken,
      user: this.sanitizeUser(user)
    })
  }

  async refresh(req, res) {
    const { refreshToken } = req.body

    if (!refreshToken) {
      return res.status(400).json({ message: 'Refresh token required' })
    }

    try {
      // Verify refresh token
      const decoded = jwt.verify(refreshToken, this.refreshTokenSecret)

      // Check if token exists in DB (not revoked)
      const isValid = await this.userRepository.verifyRefreshToken(
        decoded.sub,
        refreshToken
      )

      if (!isValid) {
        return res.status(401).json({ message: 'Refresh token revoked' })
      }

      // Revoke old refresh token (rotation)
      await this.userRepository.revokeRefreshToken(refreshToken)

      // Generate new tokens
      const user = await this.userRepository.findById(decoded.sub)
      const newAccessToken = this.generateAccessToken(user)
      const newRefreshToken = this.generateRefreshToken(user)

      await this.userRepository.storeRefreshToken(user.id, newRefreshToken)

      res.json({
        accessToken: newAccessToken,
        refreshToken: newRefreshToken
      })
    } catch (error) {
      if (error instanceof jwt.TokenExpiredError) {
        return res.status(401).json({ message: 'Refresh token expired' })
      }
      return res.status(401).json({ message: 'Invalid refresh token' })
    }
  }

  async logout(req, res) {
    const { refreshToken } = req.body
    if (refreshToken) {
      await this.userRepository.revokeRefreshToken(refreshToken)
    }
    res.json({ message: 'Logged out successfully' })
  }

  authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization']
    const token = authHeader && authHeader.split(' ')[1] // Bearer TOKEN

    if (!token) {
      return res.status(401).json({ message: 'Access token required' })
    }

    jwt.verify(token, this.accessTokenSecret, (err, decoded) => {
      if (err) {
        if (err instanceof jwt.TokenExpiredError) {
          return res.status(401).json({ message: 'Token expired', code: 'TOKEN_EXPIRED' })
        }
        return res.status(403).json({ message: 'Invalid token' })
      }

      req.user = decoded
      next()
    })
  }

  sanitizeUser(user) {
    const { passwordHash, refreshTokens, ...safeUser } = user
    return safeUser
  }
}

// Auth middleware with role checking
function authorize(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Not authenticated' })
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ message: 'Insufficient permissions' })
    }

    next()
  }
}

// Usage
router.post('/login', authController.login.bind(authController))
router.post('/refresh', authController.refresh.bind(authController))
router.post('/logout', authController.authenticateToken, authController.logout.bind(authController))
router.get('/admin', authController.authenticateToken, authorize('admin'), adminHandler)
```

### Real-World Use Cases

- **Single Page Applications**: Token-based auth with refresh token rotation
- **Microservices**: JWT as a stateless authentication mechanism between services
- **Mobile apps**: Access and refresh tokens for native applications
- **Third-party API access**: API keys and tokens for external integrations
- **OAuth2/OpenID Connect**: JWT as identity tokens in federated auth
- **Serverless applications**: Stateless auth suits serverless architectures
- **E-commerce**: Session management with token rotation

### Common Mistakes

```javascript
// Mistake: Storing sensitive data in JWT payload
// JWT payload is Base64-encoded, NOT encrypted
const token = jwt.sign({ password: 'secret123' }, secret) // Visible to anyone with the token!

// Mistake: Not checking token expiration
try { jwt.verify(token, secret) } catch (e) { /* catch all */ }
// Always let jwt.verify handle expiration checking

// Mistake: Using localStorage for tokens in XSS-vulnerable apps
// Use httpOnly cookies if XSS is a concern

// Mistake: Not rotating refresh tokens
// Using the same refresh token forever increases theft risk

// Mistake: Short access tokens without refresh mechanism
// Tokens expiring every 5 minutes without refresh will cause poor UX

// Mistake: Storing token in URL query parameters
// Tokens exposed in server logs and referrer headers

// Mistake: Not implementing token revocation
// Once a token is issued, it's valid until expiration unless you maintain a blocklist
```

### Best Practices

```javascript
// Use short-lived access tokens (15 minutes)
const accessToken = jwt.sign(payload, secret, { expiresIn: '15m' })

// Use longer-lived refresh tokens with rotation
const refreshToken = jwt.sign(payload, secret, { expiresIn: '7d' })

// Store refresh tokens securely (httpOnly, Secure, SameSite cookies)
res.cookie('refreshToken', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'Strict',
  path: '/api/auth',
  maxAge: 7 * 24 * 60 * 60 * 1000
})

// Validate all JWT claims
jwt.verify(token, secret, {
  issuer: 'myapp',
  audience: 'myapp-frontend'
})

// Implement token rotation for refresh tokens
// Each refresh request issues a new refresh token and revokes the old one

// Use asymmetric signing (RS256) for microservices
// Public key distribution, private key signing

// Keep JWT payload minimal - only non-sensitive claims
const minimalPayload = { sub: user.id, role: user.role }
```

### Performance Considerations

- JWT verification is a cryptographic operation (CPU cost)
- Large JWTs increase HTTP header size on every request
- Token decoding (without verification) is fast (just Base64 decode)
- Refresh token rotation adds database writes on each refresh
- Consider token size: 3-part Base64 encoding adds ~33% overhead
- For high-traffic APIs, cache public keys to avoid repeated fetching
- Token blacklisting requires database lookups; use short expiration instead
- Batch token validation when possible for server-to-server communication

### Interview Questions

**Q: What is the difference between JWT and OAuth?**

A: JWT is a token format, while OAuth is a protocol framework for authorization. OAuth can use JWTs as access tokens, but it's not required. OAuth defines flows for obtaining tokens, while JWT defines how tokens are structured and verified. They are complementary: OAuth handles the authorization flow, JWTs are the token format.

**Q: What is token refresh rotation and why is it important?**

A: Refresh token rotation replaces the refresh token with a new one each time it's used. This limits the window of opportunity if a refresh token is stolen - the old token is immediately invalidated. If a stolen refresh token is used after the legitimate user has already rotated it, the server can detect the theft and revoke all tokens for that user.

**Q: Why is local/essionStorage not recommended for storing JWTs?**

A: localStorage is accessible to any JavaScript running on the same origin, making it vulnerable to XSS attacks. If an attacker injects script, they can steal the token and impersonate the user. httpOnly, Secure cookies cannot be accessed by JavaScript, providing better XSS resistance. However, cookies are vulnerable to CSRF attacks, which can be mitigated with SameSite=Strict and CSRF tokens.

### Coding Challenges

```javascript
// Challenge 1: Implement a JWT verification function (without library)
function base64UrlDecode(str) {
  str = str.replace(/-/g, '+').replace(/_/g, '/')
  while (str.length % 4) str += '='
  return atob(str)
}

function verifyJWT(token, secret) {
  const [headerB64, payloadB64, signatureB64] = token.split('.')
  const signature = base64UrlDecode(signatureB64)

  // HMAC-SHA256 verification
  const key = new TextEncoder().encode(secret)
  const message = new TextEncoder().encode(`${headerB64}.${payloadB64}`)

  return crypto.subtle.importKey('raw', key, { name: 'HMAC', hash: 'SHA-256' }, false, ['verify'])
    .then(key => crypto.subtle.verify('HMAC', key, signature, message))
    .then(isValid => {
      if (!isValid) throw new Error('Invalid signature')
      return JSON.parse(base64UrlDecode(payloadB64))
    })
}

// Challenge 2: Token refresh interceptor
function createAuthInterceptor(authService) {
  let isRefreshing = false
  let failedQueue = []

  function processQueue(error, token = null) {
    failedQueue.forEach(prom => {
      if (error) prom.reject(error)
      else prom.resolve(token)
    })
    failedQueue = []
  }

  return {
    async interceptRequest(url, options) {
      const token = authService.getAccessToken()

      if (authService.isTokenExpired(token)) {
        if (!isRefreshing) {
          isRefreshing = true
          try {
            const newToken = await authService.refreshToken()
            processQueue(null, newToken)
            return this.makeRequest(url, options, newToken)
          } catch (error) {
            processQueue(error)
            authService.logout()
            throw error
          } finally {
            isRefreshing = false
          }
        } else {
          return new Promise((resolve, reject) => {
            failedQueue.push({ resolve, reject })
          }).then(newToken => this.makeRequest(url, options, newToken))
        }
      }

      return this.makeRequest(url, options, token)
    },

    makeRequest(url, options, token) {
      return fetch(url, {
        ...options,
        headers: {
          ...options.headers,
          'Authorization': `Bearer ${token}`
        }
      })
    }
  }
}

// Challenge 3: Blacklist-based token revocation
class TokenBlacklist {
  constructor(redisClient) {
    this.redis = redisClient
  }

  async revoke(jti, exp) {
    const ttl = Math.max(0, exp - Math.floor(Date.now() / 1000))
    await this.redis.set(`blacklist:${jti}`, '1', 'EX', ttl)
  }

  async isRevoked(jti) {
    const result = await this.redis.get(`blacklist:${jti}`)
    return result === '1'
  }
}
```

### Related Topics

- `Authorization` - What a user can access (complements authentication)
- `Cookies` - Cookie-based session management vs JWT
- `CORS` - Cross-origin token sending considerations
- `WebSockets` - Authentication for WebSocket connections
- `OAuth2` - Authorization framework often using JWTs
- `Session management` - Server-side sessions vs stateless JWT
- `XSS` - Cross-site scripting attacks on token storage
- `CSRF` - Cross-site request forgery protection for cookie-based auth
