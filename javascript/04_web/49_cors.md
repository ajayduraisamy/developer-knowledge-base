# CORS - Cross-Origin Resource Sharing, preflight, headers, simple requests

## Introduction

Cross-Origin Resource Sharing (CORS) is a security mechanism that allows web applications running at one origin to access resources from a different origin. Implemented primarily through HTTP headers, CORS relaxes the Same-Origin Policy enforced by browsers. Understanding CORS is essential for any developer building web applications that consume APIs from different domains, integrate third-party services, or implement multi-subdomain architectures.

## Same-origin policy

### What It Is

The same-origin policy is a critical security mechanism that restricts how a document or script loaded from one origin can interact with resources from another origin. An origin is defined by the scheme (protocol), host (domain), and port number. Resources from the same origin are allowed to interact freely; cross-origin interactions are restricted by default.

```javascript
// Same origin examples (same protocol, host, port):
// https://api.example.com/page1   vs   https://api.example.com/page2

// Different origins:
// https://example.com          vs   https://api.example.com     (different host)
// https://example.com          vs   http://example.com          (different scheme)
// https://example.com:443      vs   https://example.com:8080    (different port)
```

### Why It Is Important

Without the same-origin policy, any website could read your emails, access your bank data, or steal your session cookies. It prevents malicious websites from making authenticated requests to other sites and reading the responses. It is the foundation of web security that CORS selectively relaxes.

### What It Restricts

```javascript
// Restricted cross-origin operations:
// 1. DOM access: iframe.contentDocument from different origin
document.getElementById('other-origin-iframe').contentDocument // Error

// 2. AJAX requests: XHR and Fetch to different origins
fetch('https://api.other.com/data') // Blocked without CORS headers

// 3. Reading cross-origin responses even if request succeeds
// Response is received by browser but JavaScript cannot access it

// 4. Cookie access for different origin
document.cookie // Only same-origin cookies visible
```

## CORS mechanism

### What It Is

CORS is a system of HTTP headers that allows servers to specify which origins are permitted to access their resources. The browser acts as an enforcement agent: it checks CORS headers in responses and restricts JavaScript access to cross-origin resources if the headers don't permit it.

### How It Works Internally

When a browser makes a cross-origin request, it adds the `Origin` header to identify the requesting origin. The server responds with `Access-Control-Allow-Origin` indicating which origins are allowed. The browser compares the two and either exposes the response to JavaScript or blocks it. For certain requests, the browser first sends a preflight OPTIONS request to check permissions.

```javascript
// Request flow:
// 1. Browser detects cross-origin request
// 2. Determines if simple or preflight request
// 3. Adds Origin header to request
// 4. Server responds with CORS headers
// 5. Browser validates headers against request origin
// 6. If valid: expose response to JavaScript
// 7. If invalid: block access, fire error in JavaScript
```

### CORS Headers

```javascript
// Response headers (server sets these):
// Access-Control-Allow-Origin: https://example.com (or *)
// Access-Control-Allow-Methods: GET, POST, PUT, DELETE
// Access-Control-Allow-Headers: Content-Type, Authorization
// Access-Control-Allow-Credentials: true
// Access-Control-Max-Age: 86400 (seconds to cache preflight)
// Access-Control-Expose-Headers: X-Total-Count, X-RateLimit

// Request headers (browser adds automatically):
// Origin: https://example.com
// Access-Control-Request-Method: PUT
// Access-Control-Request-Headers: Content-Type, Authorization
```

### Beginner Examples

```javascript
// Basic CORS-enabled Fetch
fetch('https://api.github.com/users/octocat')
  .then(response => response.json())
  .then(data => console.log(data))
  // This works because GitHub's API sets:
  // Access-Control-Allow-Origin: *

// CORS error example
fetch('https://api.otherdomain.com/data')
  .then(response => response.json())
  .catch(error => console.error('CORS error:', error.message))
  // "Failed to fetch" - likely CORS blocking

// Setting withCredentials for cookies
fetch('https://api.example.com/user', {
  credentials: 'include' // Send cookies cross-origin
})
// Server must respond with:
// Access-Control-Allow-Credentials: true
// Access-Control-Allow-Origin: https://example.com (NOT *)
```

### Intermediate Examples

```javascript
// CORS proxy for development
async function corsProxyFetch(url) {
  const proxyUrl = 'https://cors-anywhere.herokuapp.com/'
  const response = await fetch(proxyUrl + url)
  return response.json()
}

// Handling CORS in Express.js
const express = require('express')
const app = express()

// Global CORS middleware
app.use((req, res, next) => {
  const origin = req.headers.origin
  const allowedOrigins = ['https://app.example.com', 'https://admin.example.com']

  if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin)
    res.setHeader('Access-Control-Allow-Credentials', 'true')
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH')
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization')
    res.setHeader('Access-Control-Max-Age', '86400')
  }

  // Handle preflight
  if (req.method === 'OPTIONS') {
    return res.sendStatus(204)
  }

  next()
})

// Dynamic origin validation
const allowedOrigins = ['https://app.example.com', /\.trusted-partner\.com$/]

app.use((req, res, next) => {
  const origin = req.headers.origin

  const isAllowed = allowedOrigins.some(allowed => {
    if (allowed instanceof RegExp) return allowed.test(origin)
    return allowed === origin
  })

  if (isAllowed) {
    res.setHeader('Access-Control-Allow-Origin', origin)
  }

  next()
})
```

### Advanced Examples

```javascript
// CORS middleware with whitelist and complex options
function corsMiddleware(options = {}) {
  const {
    origins = ['*'],
    methods = ['GET', 'POST'],
    headers = ['Content-Type'],
    credentials = false,
    maxAge = null,
    exposeHeaders = []
  } = options

  return (req, res, next) => {
    const origin = req.headers.origin

    // Check if origin is allowed
    const isAllowed = origins.some(allowed => {
      if (allowed === '*') return true
      if (allowed instanceof RegExp) return allowed.test(origin)
      return allowed === origin
    })

    if (!isAllowed) {
      return next()
    }

    // Set CORS headers
    const allowOrigin = origins.includes('*') && !credentials
      ? '*'
      : origin

    res.setHeader('Access-Control-Allow-Origin', allowOrigin)

    if (credentials) {
      res.setHeader('Access-Control-Allow-Credentials', 'true')
    }

    if (methods.length) {
      res.setHeader('Access-Control-Allow-Methods', methods.join(', '))
    }

    if (headers.length) {
      res.setHeader('Access-Control-Allow-Headers', headers.join(', '))
    }

    if (maxAge) {
      res.setHeader('Access-Control-Max-Age', String(maxAge))
    }

    if (exposeHeaders.length) {
      res.setHeader('Access-Control-Expose-Headers', exposeHeaders.join(', '))
    }

    // Handle preflight
    if (req.method === 'OPTIONS') {
      return res.sendStatus(204)
    }

    next()
  }
}

// Using the middleware
app.use(corsMiddleware({
  origins: ['https://app.example.com', /\.example\.com$/],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  headers: ['Content-Type', 'Authorization', 'X-Requested-With'],
  credentials: true,
  maxAge: 7200,
  exposeHeaders: ['X-Total-Count', 'X-RateLimit-Remaining']
}))

// CORS error handling in frontend
function corsSafeFetch(url, options = {}) {
  return fetch(url, {
    ...options,
    credentials: 'same-origin',
    mode: 'cors',
    headers: {
      'Content-Type': 'application/json',
      ...options.headers
    }
  }).catch(error => {
    if (error instanceof TypeError && error.message.includes('Failed to fetch')) {
      throw new CORSProxyError('Request blocked by CORS. Use a proxy in development.')
    }
    throw error
  })
}

class CORSProxyError extends Error {
  constructor(message) {
    super(message)
    this.name = 'CORSProxyError'
  }
}
```

### Real-World Use Cases

- **Single Page Applications**: SPAs hosted on one domain consuming APIs on another
- **Microservices architecture**: Frontend consuming multiple backend services
- **Third-party API integration**: Using external APIs like Stripe, GitHub, or Google
- **CDN-hosted assets**: Fonts, scripts, and styles from different origins
- **Subdomain isolation**: app.example.com vs api.example.com
- **Development environments**: Localhost frontend with staging API
- **Embedded widgets**: Third-party widgets making authenticated requests
- **Serverless functions**: Frontend calling cloud functions on different domains

### Common Mistakes

```javascript
// Mistake: Using wildcard with credentials
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
// Error: Cannot use wildcard with credentials

// Mistake: Forgetting preflight for PUT/DELETE requests
fetch('https://api.example.com/data', {
  method: 'PUT', // Triggers preflight!
  body: JSON.stringify({ id: 1 })
})

// Mistake: Not handling OPTIONS preflight on server
app.put('/api/data', handler) // 405 Method Not Allowed for OPTIONS

// Mistake: Mismatched origin (trailing slash)
// https://example.com/ !== https://example.com

// Mistake: Assuming CORS protects the server
// CORS is browser-enforced; server should still validate origins

// Mistake: Localhost origin mismatches
// http://localhost:3000 !== http://127.0.0.1:3000
```

### Best Practices

```javascript
// Server-side: Always validate Origin header
function isValidOrigin(origin) {
  if (!origin) return false
  try {
    const url = new URL(origin)
    return ALLOWED_DOMAINS.includes(url.hostname)
  } catch {
    return false
  }
}

// Don't use wildcard for APIs requiring authentication
// Instead, use dynamic origin matching

// Keep preflight cache reasonable (max-age: 2 hours = 7200)
res.setHeader('Access-Control-Max-Age', '7200')

// Only expose headers that are needed
res.setHeader('Access-Control-Expose-Headers', 'X-Request-Id')

// Use environment-specific CORS config
const corsConfig = {
  development: { origins: ['http://localhost:3000', 'http://localhost:5173'] },
  staging: { origins: [/\.staging\.example\.com$/] },
  production: { origins: ['https://app.example.com'] }
}

// Frontend: Set mode explicitly
fetch(url, { mode: 'cors' })

// Handle CORS errors gracefully
fetch(url).catch(err => {
  if (err.name === 'TypeError') {
    showCorsWarning('API access blocked')
  }
})
```

### Performance Considerations

- Preflight requests add an extra HTTP round trip (OPTIONS)
- Set `Access-Control-Max-Age` to cache preflight results (reduce OPTIONS requests)
- Keep the allowed origins list small for faster header processing
- Consider same-origin architecture for latency-sensitive applications
- Proxy through same-origin server to avoid CORS entirely in production
- CDN CORS configuration can affect caching behavior
- Credentialed requests don't cache at all with wildcard origins

### Interview Questions

**Q: What is the difference between simple and preflight requests?**

A: Simple requests meet all these criteria: GET/HEAD/POST method, only CORS-safe headers (Accept, Accept-Language, Content-Language, Content-Type with values application/x-www-form-urlencoded, multipart/form-data, or text/plain), no event listeners on XMLHttpRequest upload, and no ReadableStream used. Everything else triggers a preflight OPTIONS request to check permissions.

**Q: Can you use `Access-Control-Allow-Origin: *` with credentials?**

A: No. When `Access-Control-Allow-Credentials: true` is set, `Access-Control-Allow-Origin` cannot be `*`. It must specify an explicit origin. The browser enforces this rule. The server must dynamically echo back the requesting origin after validation.

**Q: How does CORS affect performance?**

A: Preflight requests (OPTIONS) add latency to every non-simple request. This can be mitigated by setting `Access-Control-Max-Age` to cache preflight results. For APIs with many endpoints, consider grouping them under fewer routes. Also, credentialed requests don't participate in CDN caching with wildcard origins.

### Coding Challenges

```javascript
// Challenge 1: CORS proxy server
const http = require('http')

http.createServer((req, res) => {
  const targetUrl = req.url.slice(1) // Remove leading /

  http.get(targetUrl, (targetRes) => {
    res.writeHead(targetRes.statusCode, {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': targetRes.headers['content-type']
    })
    targetRes.pipe(res)
  }).on('error', () => {
    res.writeHead(500, { 'Access-Control-Allow-Origin': '*' })
    res.end('Proxy error')
  })
}).listen(8080)

// Challenge 2: Validate CORS configuration for a given origin
function validateCORSConfig(config, origin, requestMethod, requestHeaders) {
  const errors = []

  // Check origin
  const allowedOrigins = config['Access-Control-Allow-Origin']
  if (!allowedOrigins || allowedOrigins === '*' && config['Access-Control-Allow-Credentials']) {
    errors.push('Credentials not allowed with wildcard origin')
  }

  // Check method
  if (requestMethod !== 'GET' && requestMethod !== 'POST') {
    const allowedMethods = config['Access-Control-Allow-Methods']
    if (!allowedMethods || !allowedMethods.includes(requestMethod)) {
      errors.push(`Method ${requestMethod} not allowed`)
    }
  }

  // Check headers
  if (requestHeaders) {
    const allowedHeaders = config['Access-Control-Allow-Headers']
    const headers = requestHeaders.split(', ')
    headers.forEach(h => {
      if (!allowedHeaders?.includes(h)) {
        errors.push(`Header ${h} not allowed`)
      }
    })
  }

  return { valid: errors.length === 0, errors }
}

// Challenge 3: Detect if a response would be blocked by CORS
async function checkCORSAvailability(url) {
  try {
    const response = await fetch(url, { method: 'OPTIONS' })
    const allowOrigin = response.headers.get('access-control-allow-origin')
    const allowMethods = response.headers.get('access-control-allow-methods')
    return {
      corsEnabled: !!allowOrigin,
      allowedOrigin: allowOrigin,
      allowedMethods: allowMethods
    }
  } catch {
    return { corsEnabled: false, error: 'Cannot check CORS headers' }
  }
}
```

### Related Topics

- `Fetch API` - Subject to CORS restrictions
- `AJAX (XHR)` - Also subject to CORS; withCredentials property
- `Cookies` - SameSite attribute affects cross-origin cookie sending
- `Authentication` - CORS restrictions for token-based auth
- `Content Security Policy` - Another security layer for resource loading
- `OWASP` - Web security best practices including CORS configuration
- `HTTP Headers` - All CORS mechanisms are header-based
