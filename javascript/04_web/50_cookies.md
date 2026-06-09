# Cookies - document.cookie, set/get/delete, attributes, HttpOnly/Secure/SameSite

## Introduction

Cookies are small pieces of data stored by the browser, associated with specific origins, and sent with every HTTP request to that origin. They are the oldest form of client-side storage on the web and remain essential for session management, personalization, and tracking. JavaScript interacts with cookies through the `document.cookie` API, while server-side cookies are set via HTTP headers (`Set-Cookie`). Modern cookie security attributes like HttpOnly, Secure, and SameSite provide critical protection against common attacks.

## document.cookie

### What It Is

`document.cookie` is a property that allows reading and writing cookies from JavaScript. When read, it returns a semicolon-separated string of all cookies for the current origin (excluding those with HttpOnly flag). When written, it sets or updates a single cookie.

```javascript
// Read all cookies
console.log(document.cookie)
// "sessionId=abc123; theme=dark; language=en"

// Write a cookie (sets or updates one cookie at a time)
document.cookie = "username=John"
```

### Why It Is Important

`document.cookie` provides client-side access to cookies for personalization, preference storage, and frontend session management. However, its string-based API is primitive compared to modern storage APIs, and its synchronous nature can impact performance.

### How It Works Internally

Reading `document.cookie` returns all cookies for the current origin and path that aren't HttpOnly. The browser maintains a cookie jar indexed by (domain, path, name). Writing appends or updates a cookie in this jar. The jar is limited in size (typically 4KB per cookie, ~50-150 cookies per domain). When making HTTP requests, the browser automatically includes matching cookies in the Cookie header.

### Syntax

```javascript
// Reading
document.cookie
// Returns: "key1=value1; key2=value2; key3=value3"

// Writing (sets a single cookie with optional attributes)
document.cookie = "key=value; expires=date; path=/; domain=.example.com; Secure; HttpOnly; SameSite=Lax"

// Deleting (set expires in the past)
document.cookie = "key=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/"
```

### Beginner Examples

```javascript
// Set a simple cookie
document.cookie = "username=Alice"

// Read all cookies
const cookies = document.cookie
console.log(cookies) // "username=Alice"

// Parse cookies into an object
function getCookies() {
  const pairs = document.cookie.split('; ')
  const cookies = {}
  pairs.forEach(pair => {
    const [key, ...rest] = pair.split('=')
    cookies[key] = decodeURIComponent(rest.join('='))
  })
  return cookies
}

console.log(getCookies()) // { username: "Alice" }

// Simple cookie getter
function getCookie(name) {
  const match = document.cookie.match(new RegExp(`(^| )${name}=([^;]+)`))
  return match ? decodeURIComponent(match[2]) : null
}

console.log(getCookie('username')) // "Alice"
```

### Intermediate Examples

```javascript
// Full cookie utility library
const CookieUtil = {
  set(name, value, options = {}) {
    let cookie = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`

    if (options.expires instanceof Date) {
      cookie += `; expires=${options.expires.toUTCString()}`
    } else if (typeof options.maxAge === 'number') {
      const expires = new Date(Date.now() + options.maxAge * 1000)
      cookie += `; expires=${expires.toUTCString()}`
    }

    if (options.path) cookie += `; path=${options.path}`
    if (options.domain) cookie += `; domain=${options.domain}`
    if (options.secure) cookie += '; Secure'
    if (options.httpOnly) cookie += '; HttpOnly'
    if (options.sameSite) cookie += `; SameSite=${options.sameSite}`

    document.cookie = cookie
  },

  get(name) {
    const match = document.cookie.match(
      new RegExp(`(?:^|;\\s*)${encodeURIComponent(name)}\\s*=\\s*([^;]*)`)
    )
    return match ? decodeURIComponent(match[1]) : null
  },

  getAll() {
    const cookies = {}
    document.cookie.split('; ').forEach(pair => {
      const separatorIndex = pair.indexOf('=')
      if (separatorIndex > 0) {
        const name = decodeURIComponent(pair.slice(0, separatorIndex))
        const value = decodeURIComponent(pair.slice(separatorIndex + 1))
        cookies[name] = value
      }
    })
    return cookies
  },

  delete(name, options = {}) {
    this.set(name, '', {
      ...options,
      expires: new Date(0),
      maxAge: 0
    })
  },

  exists(name) {
    return this.get(name) !== null
  }
}

// Usage
CookieUtil.set('theme', 'dark', { path: '/', maxAge: 3600 })
CookieUtil.set('session', 'abc123', { secure: true, sameSite: 'Strict' })
console.log(CookieUtil.get('theme')) // "dark"
CookieUtil.delete('theme')

// Cookie consent management
class CookieConsentManager {
  constructor() {
    this.consentKey = 'cookie_consent'
  }

  grantConsent(categories = ['necessary']) {
    CookieUtil.set(this.consentKey, JSON.stringify({
      granted: categories,
      timestamp: new Date().toISOString(),
      version: 1
    }), { path: '/', maxAge: 31536000 }) // 1 year
  }

  hasConsent(category) {
    const consent = CookieUtil.get(this.consentKey)
    if (!consent) return false
    try {
      const parsed = JSON.parse(consent)
      return parsed.granted.includes(category) || parsed.granted.includes('all')
    } catch {
      return false
    }
  }

  withdrawConsent() {
    CookieUtil.delete(this.consentKey, { path: '/' })
  }
}
```

### Advanced Examples

```javascript
// Encrypted cookies for sensitive data (client-side encryption)
async function setSecureCookie(name, value, password) {
  const encoder = new TextEncoder()
  const data = encoder.encode(JSON.stringify(value))

  const key = await crypto.subtle.importKey(
    'raw',
    encoder.encode(password.padEnd(32).slice(0, 32)),
    { name: 'AES-GCM' },
    false,
    ['encrypt']
  )

  const iv = crypto.getRandomValues(new Uint8Array(12))
  const encrypted = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, key, data)

  const combined = new Uint8Array(iv.length + encrypted.byteLength)
  combined.set(iv)
  combined.set(new Uint8Array(encrypted), iv.length)

  CookieUtil.set(name, btoa(String.fromCharCode(...combined)), {
    path: '/',
    secure: true,
    sameSite: 'Strict'
  })
}

async function getSecureCookie(name, password) {
  const raw = CookieUtil.get(name)
  if (!raw) return null

  const combined = Uint8Array.from(atob(raw), c => c.charCodeAt(0))
  const iv = combined.slice(0, 12)
  const data = combined.slice(12)

  const encoder = new TextEncoder()
  const key = await crypto.subtle.importKey(
    'raw',
    encoder.encode(password.padEnd(32).slice(0, 32)),
    { name: 'AES-GCM' },
    false,
    ['decrypt']
  )

  const decrypted = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, key, data)
  return JSON.parse(new TextDecoder().decode(decrypted))
}

// Cookie change observer
function observeCookieChanges(callback) {
  let lastCookies = document.cookie

  setInterval(() => {
    const currentCookies = document.cookie
    if (currentCookies !== lastCookies) {
      const oldParsed = CookieUtil.getAll()
      const newParsed = CookieUtil.getAll()

      const changes = []
      Object.keys({ ...oldParsed, ...newParsed }).forEach(name => {
        if (oldParsed[name] !== newParsed[name]) {
          changes.push({
            name,
            oldValue: oldParsed[name] || null,
            newValue: newParsed[name] || null,
            action: newParsed[name] === undefined ? 'deleted'
              : oldParsed[name] === undefined ? 'added'
              : 'updated'
          })
        }
      })

      if (changes.length) {
        callback(changes)
      }
      lastCookies = currentCookies
    }
  }, 100)
}

// Server-side cookie parsing (Node.js)
function parseCookies(request) {
  const cookieHeader = request.headers?.cookie || ''
  const cookies = {}

  cookieHeader.split(';').forEach(pair => {
    const separatorIndex = pair.indexOf('=')
    if (separatorIndex > 0) {
      const name = pair.slice(0, separatorIndex).trim()
      const value = pair.slice(separatorIndex + 1).trim()
      cookies[name] = decodeURIComponent(value)
    }
  })

  return cookies
}

function setServerCookie(response, name, value, options = {}) {
  let cookie = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`

  if (options.expires) cookie += `; Expires=${options.expires.toUTCString()}`
  if (options.maxAge) cookie += `; Max-Age=${options.maxAge}`
  if (options.path) cookie += `; Path=${options.path}`
  if (options.domain) cookie += `; Domain=${options.domain}`
  if (options.secure) cookie += '; Secure'
  if (options.httpOnly) cookie += '; HttpOnly'
  if (options.sameSite) cookie += `; SameSite=${options.sameSite}`

  const existing = response.getHeader('Set-Cookie') || []
  response.setHeader('Set-Cookie', [...(Array.isArray(existing) ? existing : [existing]), cookie])
}
```

### Real-World Use Cases

- **Session management**: Server-generated session IDs for authenticated sessions
- **Personalization**: User preferences (theme, language, layout)
- **Tracking**: Analytics cookies for user behavior tracking
- **A/B testing**: Cookie-based assignment to experiment groups
- **CSRF tokens**: Anti-CSRF tokens stored in cookies
- **Cookie consent**: GDPR/CCPA compliance cookie preference storage
- **Shopping carts**: E-commerce cart data (though increasingly replaced by server-side storage)
- **Authentication tokens**: Session tokens with HttpOnly flag

### Common Mistakes

```javascript
// Mistake: Trying to set multiple cookies in one assignment
document.cookie = "a=1; b=2" // Only "a=1" is set

// Mistake: Forgetting path when deleting
CookieUtil.set('test', 'value', { path: '/app' })
CookieUtil.delete('test') // Doesn't delete because path mismatch
// Must use: CookieUtil.delete('test', { path: '/app' })

// Mistake: Not encoding special characters
document.cookie = "name=John Doe" // Space breaks the cookie
// Correct: encodeURIComponent

// Mistake: Expecting cookies to work in file:// protocol
// Cookies require http:// or https://

// Mistake: Reading HttpOnly cookies from JavaScript
document.cookie // Does not include HttpOnly cookies

// Mistake: Setting SameSite=None without Secure
document.cookie = "x=1; SameSite=None" // Browser rejects this
// Must be: "x=1; SameSite=None; Secure"

// Mistake: Cookies with different paths on same domain
// Cookie for /app and cookie for /admin are both sent to /admin
// This can cause unexpected behavior
```

### Best Practices

```javascript
// Always encode cookie names and values
function safeSetCookie(name, value) {
  document.cookie = `${encodeURIComponent(name)}=${encodeURIComponent(value)}; path=/; SameSite=Lax`
}

// Use secure cookies in production
document.cookie = "session=abc; Secure; SameSite=Strict"

// Set explicit path to avoid scope ambiguity
document.cookie = "pref=dark; path=/"

// Use HttpOnly for sensitive cookies (server-side only)
// Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict

// Implement cookie size limiting
function setSafeCookie(name, value) {
  const encoded = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`
  if (encoded.length > 4096) {
    throw new Error('Cookie exceeds 4KB size limit')
  }
  document.cookie = `${encoded}; path=/`
}

// Use max-age instead of expires for modern applications
document.cookie = "x=1; max-age=3600; path=/"
```

### Performance Considerations

- Cookies are sent with every HTTP request to the domain, increasing request size
- Large cookies (>1KB) can significantly impact performance on sites with many requests
- Each cookie has a maximum size of ~4KB
- Browsers limit ~50-150 cookies per domain
- Reading `document.cookie` is synchronous and can be slow with many cookies
- Minimize cookie count and size; use Web Storage for client-only data
- CDNs may not cache responses with `Set-Cookie` headers

### Interview Questions

**Q: What is the difference between localStorage and cookies?**

A: Cookies are sent with every HTTP request automatically (typically 4KB size limit), have built-in expiration, support Secure/HttpOnly/SameSite attributes for security, and can be set from both server and client. localStorage is only client-side, has much larger capacity (~5-10MB), is not sent with requests, and has no expiration. Cookies are best for session management; localStorage is better for client-only storage.

**Q: What does the SameSite attribute do?**

A: SameSite controls whether cookies are sent with cross-origin requests. `Strict` sends cookies only for same-site requests. `Lax` sends cookies for top-level navigations (GET) from other sites but not for embedded requests (images, iframes, fetch). `None` sends cookies with all cross-origin requests (requires Secure flag). SameSite=Lax is the modern default and prevents most CSRF attacks.

**Q: How do you delete a cookie?**

A: Set the cookie with the same name, path, and domain, but with an expiration date in the past (e.g., `expires=Thu, 01 Jan 1970 00:00:00 GMT` or `max-age=0`). If the original cookie was set with a specific path or domain, those must be provided for deletion to work.

### Coding Challenges

```javascript
// Challenge 1: Cookie jar with namespace isolation
class CookieJar {
  constructor(namespace) {
    this.namespace = namespace
    this.prefix = `__${namespace}_`
  }

  set(key, value, options) {
    CookieUtil.set(this.prefix + key, value, options)
  }

  get(key) {
    return CookieUtil.get(this.prefix + key)
  }

  delete(key, options) {
    CookieUtil.delete(this.prefix + key, options)
  }

  getAll() {
    const all = CookieUtil.getAll()
    const ns = {}
    Object.keys(all).forEach(key => {
      if (key.startsWith(this.prefix)) {
        ns[key.slice(this.prefix.length)] = all[key]
      }
    })
    return ns
  }

  clear(options) {
    Object.keys(this.getAll()).forEach(key => {
      this.delete(key, options)
    })
  }
}

// Challenge 2: Cookie size monitor
function getCookieSize() {
  const bytes = new TextEncoder().encode(document.cookie).length
  return {
    bytes,
    kilobytes: (bytes / 1024).toFixed(2),
    count: document.cookie ? document.cookie.split('; ').length : 0
  }
}

// Challenge 3: Cookie-based CSRF token
function generateCSRFToken() {
  const array = new Uint8Array(32)
  crypto.getRandomValues(array)
  return Array.from(array, b => b.toString(16).padStart(2, '0')).join('')
}

function setupCSRFProtection() {
  if (!CookieUtil.get('csrf_token')) {
    CookieUtil.set('csrf_token', generateCSRFToken(), {
      path: '/',
      secure: true,
      sameSite: 'Strict'
    })
  }
}
```

### Related Topics

- `Authentication` - Cookies are a primary mechanism for session tokens
- `CORS` - Cross-origin cookie behavior affected by SameSite and credentials
- `localStorage` - Alternative client-side storage without HTTP auto-inclusion
- `sessionStorage` - Tab-scoped client-side storage
- `CSRF` - Cookies require CSRF protection strategies
- `Content Security Policy` - Can restrict cookie usage
- `GDPR/CCPA` - Cookie consent regulations and compliance
- `IndexedDB` - Larger-scale client-side storage option
