# CSRF - Cross-Site Request Forgery, tokens, SameSite cookies, prevention

## Introduction

Cross-Site Request Forgery (CSRF, pronounced "sea-surf") is an attack that forces an authenticated user to execute unwanted actions on a web application in which they are currently authenticated. The attacker crafts a malicious request that the victim's browser automatically includes authentication credentials (cookies, HTTP Basic auth, etc.), making the server process the request as if it were legitimate. CSRF exploits the trust a web application has in the authenticated user's browser. Understanding CSRF is essential for any JavaScript developer building web applications that handle state-changing operations like transfers, password changes, or data modification.

## CSRF Attack Mechanism

### What It Is

A CSRF attack works by tricking an authenticated user into submitting a request to a target application without their consent or knowledge. The attacker cannot see the response, so they target state-changing requests rather than data theft. The attack relies on the browser's automatic inclusion of credentials (cookies) when making requests to a domain. If a user is logged into a banking site, and the attacker tricks them into visiting a page that submits a fund-transfer form to the banking site, the request includes the user's session cookie, and the bank processes the transfer.

### Why It Is Important

CSRF has historically been one of the most damaging web vulnerabilities because it exploits a fundamental feature of HTTP authentication. Major platforms like Twitter, Facebook, and YouTube have suffered CSRF vulnerabilities that allowed attackers to post messages, change settings, or perform actions as the victim. CSRF is particularly dangerous for applications that rely solely on cookies for authentication and have no additional verification for state-changing requests. Modern frameworks have built-in CSRF protection, but understanding the mechanism is still critical for configuring and maintaining security.

### How It Works Internally

The CSRF attack chain proceeds as follows:

1. The victim logs into a legitimate application (e.g., `bank.com`) and receives a session cookie.
2. The attacker crafts a malicious page or email that contains an auto-submitting form, an image tag, or a script tag that makes a request to `bank.com/transfer`.
3. The victim visits the attacker's page while still authenticated to `bank.com`.
4. The victim's browser sends the crafted request to `bank.com`, automatically including the session cookie.
5. The server receives the request. Because the request includes valid authentication and appears to come from the user's browser, the server processes the action.

The key is that the browser enforces the same-origin policy for reading responses but does not prevent cross-origin *sending* of requests with credentials. For cookies, the browser automatically attaches them regardless of which site initiated the request.

### Syntax

```javascript
// Attacker's malicious page (attacker.com/attack.html)
<!DOCTYPE html>
<html>
<body>
  <form action="https://bank.com/transfer" method="POST" id="csrf-form">
    <input type="hidden" name="toAccount" value="attacker-account">
    <input type="hidden" name="amount" value="10000">
  </form>
  <script>
    // Auto-submit the form when the page loads
    document.getElementById('csrf-form').submit();
  </script>
</body>
</html>
```

Alternatively, using an image tag for GET-based requests:

```html
<!-- Attacker uses an img tag for GET-based CSRF -->
<img src="https://bank.com/transfer?to=attacker&amount=10000" width="0" height="0">
```

### Beginner Examples

```html
<!-- Simple CSRF via image tag (GET-based) -->
<!-- Victim's bank has an endpoint: -->
<!-- GET /api/delete-account -->
<!-- Attacker embeds: -->
<img src="https://vulnerable-app.com/api/delete-account" style="display:none">

<!-- CSRF via auto-submitting form (POST-based) -->
<form id="evil" action="https://social-network.com/api/post" method="POST" target="hidden-frame">
  <input name="content" value="I won the lottery! Click here: http://scam.com">
</form>
<script>document.getElementById('evil').submit()</script>
```

### Intermediate Examples

```javascript
// Intermediate: CSRF via XHR + CORS misconfiguration
const xhr = new XMLHttpRequest();
xhr.open('POST', 'https://api.target.com/change-email');
xhr.withCredentials = true; // Send cookies
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(JSON.stringify({email: 'attacker@evil.com'}));

// This only works if the target server has:
// Access-Control-Allow-Origin: *
// Access-Control-Allow-Credentials: true
// (This combination is insecure and rare, but possible)
```

### Advanced Examples

```javascript
// Advanced: CSRF bypass via CORS misconfiguration with dynamic origin reflection
// If server reflects Origin header in Access-Control-Allow-Origin:
// Attacker makes request from attacker.com
// Server sees Origin: https://attacker.com
// Server responds with Access-Control-Allow-Origin: https://attacker.com
// CORS allows the request

// Advanced: CSRF via JSONP endpoints
// JSONP endpoints can be called cross-origin via script tags
<script>
function callback(data) {
  // Data leaked via CSRF
}
</script>
<script src="https://api.target.com/user/profile?callback=callback"></script>
```

### Real-World Use Cases

- Changing email/password on a web application
- Making financial transactions
- Posting content on social media
- Modifying application settings
- Deleting user accounts or data
- Adding administrative users

### Common Mistakes

- Assuming POST requests alone prevent CSRF (attackers can auto-submit forms)
- Only checking `Referer` header (can be spoofed or missing due to privacy extensions)
- Using CSRF tokens only on login forms (forgetting other state-changing endpoints)
- Exposing CSRF tokens in URLs (leaked via Referer header)
- Not regenerating CSRF tokens after login
- Using weak CSRF token generation (predictable values)

### Best Practices

- Use CSRF tokens for all state-changing requests (POST, PUT, DELETE, PATCH)
- Use SameSite cookies (Strict or Lax) as a primary defense
- Implement double-submit cookie pattern as an alternative
- Regenerate CSRF tokens on user authentication
- Use framework-provided CSRF protection (Express.js csurf, Django CSRF middleware, Rails CSRF protection)
- For API-only applications, use token-based authentication (JWT, OAuth) sent via Authorization header (not cookies)

### Performance Considerations

- CSRF token generation has negligible cost (random string generation)
- Token validation per request adds minimal CPU overhead
- Store tokens in server-side session or signed cookies; avoid database lookups per request
- CSRF middleware typically adds <1ms per request
- SameSite cookie evaluation is done entirely by the browser

### Interview Questions

**Q: Explain how a CSRF attack works in three steps.**
A: Step 1: The victim authenticates to the target site and receives a session cookie. Step 2: The attacker tricks the victim into visiting a malicious page (via email, social media, etc.). Step 3: The malicious page makes an automated request to the target site; the browser automatically includes the victim's session cookie, and the server processes the request as legitimate.

**Q: Why doesn't the same-origin policy prevent CSRF?**
A: The same-origin policy restricts reading cross-origin responses, but sending cross-origin requests (with cookies) is not restricted. CSRF doesn't require reading the response; it only needs the request to be processed. The browser enforces the same-origin policy on reading, not on writing.

### Coding Challenges

**Challenge 1:** Create a server endpoint that implements CSRF protection.

```javascript
// Vulnerable endpoint
app.post('/transfer', (req, res) => {
  const { toAccount, amount } = req.body;
  // Process transfer without CSRF check
  processTransfer(req.user.id, toAccount, amount);
  res.json({ success: true });
});
```

**Solution:**

```javascript
const crypto = require('crypto');

// CSRF token middleware
function csrfProtection(req, res, next) {
  // Generate token if not exists
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString('hex');
  }
  
  // For state-changing methods, validate token
  if (['POST', 'PUT', 'DELETE', 'PATCH'].includes(req.method)) {
    const token = req.body._csrf || req.headers['x-csrf-token'];
    if (!token || token !== req.session.csrfToken) {
      return res.status(403).json({ error: 'CSRF token invalid' });
    }
  }
  
  next();
}

app.use(csrfProtection);

app.post('/transfer', (req, res) => {
  const { toAccount, amount } = req.body;
  processTransfer(req.user.id, toAccount, amount);
  res.json({ success: true });
});
```

## CSRF Tokens

### What It Is

CSRF tokens are unique, unpredictable, server-generated values embedded in web forms or API requests. The server validates the token for every state-changing request, ensuring the request originated from the legitimate application and not from an external site. Tokens are bound to the user's session and are typically random strings of sufficient entropy (128 bits or more).

### Why It Is Important

CSRF tokens are the most widely deployed defense against CSRF attacks. When correctly implemented, they provide strong protection because an attacker cannot guess or derive the token value. Even if the victim is authenticated and clicks a malicious link, the forged request will lack the correct CSRF token, and the server rejects it. Tokens are the recommended defense in the OWASP CSRF Prevention Cheat Sheet.

### How It Works Internally

The server generates a random token during the user's session and stores it server-side (typically in session storage). When rendering forms, the server includes the token as a hidden input field or in a meta tag. When the form is submitted, the client sends the token back. For AJAX requests, the token is sent in a custom HTTP header (X-CSRF-Token). The server compares the received token against the stored value. Because cross-origin attackers cannot read the token from the legitimate page (same-origin policy), they cannot include it in forged requests.

```javascript
// Server-side token generation
function generateToken() {
  return crypto.randomBytes(32).toString('hex'); // 64 hex chars
}

// Token binding to session
req.session.csrfToken = generateToken();
```

### Syntax

```javascript
// Including CSRF token in HTML form
<form action="/transfer" method="POST">
  <input type="hidden" name="_csrf" value="<%= csrfToken %>">
  <input type="text" name="toAccount">
  <input type="number" name="amount">
  <button type="submit">Transfer</button>
</form>

// Sending CSRF token via AJAX (Express.js example)
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': csrfToken // Custom header
  },
  body: JSON.stringify({ toAccount, amount }),
  credentials: 'include'
});
```

### Beginner Examples

```javascript
// Full example: CSRF token implementation in Express
const express = require('express');
const session = require('express-session');
const crypto = require('crypto');

const app = express();

app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: true
}));

// Generate and expose CSRF token
app.get('/form', (req, res) => {
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString('hex');
  }
  res.send(`
    <form action="/transfer" method="POST">
      <input type="hidden" name="_csrf" value="${req.session.csrfToken}">
      <input type="text" name="toAccount" placeholder="Account">
      <input type="number" name="amount" placeholder="Amount">
      <button type="submit">Transfer</button>
    </form>
  `);
});
```

### Intermediate Examples

```javascript
// Intermediate: Double-submit cookie pattern (no server-side token storage)
// Server sets a random cookie
app.use((req, res, next) => {
  if (!req.cookies.csrfToken) {
    const token = crypto.randomBytes(32).toString('hex');
    res.cookie('csrfToken', token, {
      httpOnly: true,
      sameSite: 'strict',
      secure: true
    });
  }
  next();
});

// Client reads cookie and sends as header
// Server validates header matches cookie
app.post('/transfer', (req, res) => {
  const headerToken = req.headers['x-csrf-token'];
  const cookieToken = req.cookies.csrfToken;
  
  if (!headerToken || !crypto.timingSafeEqual(
    Buffer.from(headerToken),
    Buffer.from(cookieToken)
  )) {
    return res.status(403).json({ error: 'CSRF token mismatch' });
  }
  
  // Process request...
});
```

### Advanced Examples

```javascript
// Advanced: Stateless CSRF protection with signed tokens
const crypto = require('crypto');

class CSRFProtection {
  constructor(secret) {
    this.secret = secret;
  }
  
  generateToken(sessionId, expiryMs = 3600000) {
    const expiry = Date.now() + expiryMs;
    const payload = `${sessionId}:${expiry}`;
    const hmac = crypto.createHmac('sha256', this.secret)
      .update(payload)
      .digest('hex');
    return Buffer.from(`${payload}:${hmac}`).toString('base64');
  }
  
  validateToken(token, sessionId) {
    try {
      const decoded = Buffer.from(token, 'base64').toString();
      const [sid, expiry, hmac] = decoded.split(':');
      
      if (sid !== sessionId) return false;
      if (Date.now() > parseInt(expiry)) return false;
      
      const expectedHmac = crypto.createHmac('sha256', this.secret)
        .update(`${sid}:${expiry}`)
        .digest('hex');
      
      return crypto.timingSafeEqual(Buffer.from(hmac), Buffer.from(expectedHmac));
    } catch {
      return false;
    }
  }
}

// Usage
const csrf = new CSRFProtection(process.env.CSRF_SECRET);
app.post('/transfer', (req, res) => {
  const token = req.headers['x-csrf-token'];
  if (!csrf.validateToken(token, req.session.id)) {
    return res.status(403).json({ error: 'Invalid token' });
  }
  // Process...
});
```

### Real-World Use Cases

- Banking and financial transaction systems
- E-commerce checkout and payment processing
- Social media posting and content management
- Email account settings changes
- Admin panels for user management
- API endpoints that modify state

### Common Mistakes

- Using GET requests for state-changing operations
- Including CSRF tokens in URL parameters (leak via Referer)
- Using predictable token values (sequential IDs, timestamps)
- Not binding tokens to specific user sessions
- Exposing CSRF tokens in JSON responses fetched by other origins
- Using CSRF protection only on form pages but not on API endpoints

### Best Practices

- Use cryptographically secure random token generation
- Store tokens server-side in session or use signed tokens
- Set tokens with appropriate expiry
- Include tokens in custom HTTP headers for AJAX/API requests
- Use `SameSite` cookies for additional protection
- Validate tokens using constant-time comparison
- Regenerate tokens on login/logout
- Include CSRF protection in all state-changing methods

### Performance Considerations

- Token generation (random bytes) is extremely fast
- Server-side token storage: in-memory session storage is fastest
- Token validation per request adds <0.1ms
- HMAC-based stateless tokens eliminate server-side storage lookup
- Cookie-based token storage avoids session lookups

### Interview Questions

**Q: What is the double-submit cookie pattern for CSRF protection?**
A: The server sets a random value as a cookie. The client reads this cookie and sends the same value as a custom request header or request body parameter. The server validates that the header/body value matches the cookie value. An attacker cannot read the cookie value cross-origin due to same-origin policy, so they cannot include it in forged requests.

**Q: Why are stateless CSRF tokens useful?**
A: Stateless tokens (e.g., HMAC-signed tokens containing user ID and expiry) eliminate server-side token storage, reducing database lookups and memory usage. They're ideal for distributed/stateless architectures where session state is not shared across servers. The token carries its own verification data in a cryptographically signed package.

### Coding Challenges

**Challenge 1:** Implement a CSRF token validation middleware.

```javascript
function csrfValidator(req, res, next) {
  // Your implementation here
}
```

**Solution:**

```javascript
const crypto = require('crypto');

function csrfValidator(req, res, next) {
  const methods = ['POST', 'PUT', 'DELETE', 'PATCH'];
  if (!methods.includes(req.method)) return next();
  
  const token = req.body._csrf || 
                req.query._csrf || 
                req.headers['x-csrf-token'] || 
                req.headers['csrf-token'];
  
  const storedToken = req.session?.csrfToken;
  
  if (!token || !storedToken) {
    return res.status(403).json({ error: 'CSRF token missing' });
  }
  
  // Constant-time comparison
  try {
    const tokenBuf = Buffer.from(token);
    const storedBuf = Buffer.from(storedToken);
    if (tokenBuf.length !== storedBuf.length || 
        !crypto.timingSafeEqual(tokenBuf, storedBuf)) {
      return res.status(403).json({ error: 'CSRF token invalid' });
    }
  } catch {
    return res.status(403).json({ error: 'CSRF token error' });
  }
  
  next();
}
```

## SameSite Cookie Attribute

### What It Is

The `SameSite` cookie attribute tells the browser when to send cookies with cross-origin requests. Introduced in RFC 6265bis, it provides three modes: `Strict` (cookies sent only for same-site requests), `Lax` (cookies sent for same-site requests and top-level navigation GET requests from other sites), and `None` (cookies sent for all cross-origin requests, requires `Secure` flag). Modern browsers default to `Lax` if `SameSite` is not specified.

### Why It Is Important

SameSite cookies provide a clean, browser-enforced CSRF defense that doesn't require server-side token validation for many scenarios. When set to `Strict` or `Lax`, the browser automatically prevents CSRF by withholding the session cookie from cross-site requests. This eliminates the CSRF attack vector for cookie-authenticated applications. It's a defense-in-depth measure that works even if CSRF token validation fails.

### How It Works Internally

The browser determines whether a request is "same-site" or "cross-site" by comparing the site (registrable domain + scheme) of the initiating request with the target request's site. A "same-site" request originates from the same registrable domain. A "cross-site" request originates from a different registrable domain. The `SameSite` attribute controls cookie attachment based on this determination.

`Strict`: Cookies are never sent for cross-site requests, including when the user clicks a link from another site. This provides maximum security but can break user experience when following links.

`Lax`: Cookies are sent for top-level cross-site navigations that use safe HTTP methods (GET, HEAD, OPTIONS, TRACE). This allows users to follow links to a site while still preventing CSRF via form POST or XHR from other origins.

`None`: Cookies are sent for all requests, including cross-origin. This is the traditional behavior. `None` requires the `Secure` flag (cookies only sent over HTTPS).

### Syntax

```javascript
// Setting SameSite cookie in Express
res.cookie('sessionId', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict' // 'lax' | 'strict' | 'none'
});

// Setting SameSite cookie via Set-Cookie header
Set-Cookie: sessionId=abc123; SameSite=Strict; Secure; HttpOnly

// Reading SameSite setting (server-side)
const cookie = req.cookies.sessionId;
// Browser automatically enforces SameSite, server doesn't check it
```

### Beginner Examples

```javascript
// Setting session cookie with SameSite=Lax (good default for most sites)
app.use(session({
  secret: 'secret-key',
  resave: false,
  saveUninitialized: true,
  cookie: {
    secure: true,
    httpOnly: true,
    sameSite: 'lax' // Balanced security and usability
  }
}));

// For banking apps, use Strict
app.use(session({
  // ...
  cookie: {
    sameSite: 'strict',
    secure: true,
    httpOnly: true
  }
}));
```

### Intermediate Examples

```javascript
// Intermediate: Handling SameSite=None for embedded content
// If your app is embedded in iframes on other sites,
// you need SameSite=None; Secure
app.use(session({
  cookie: {
    sameSite: 'none', // Required for third-party use
    secure: true,     // Required when sameSite=none
    httpOnly: true
  }
}));

// Intermediate: Progressive enhancement with SameSite
// Server-side check for SameSite support
function getCookieOptions(req) {
  const ua = req.headers['user-agent'] || '';
  const hasSameSiteSupport = !ua.includes('Chrome/5') && !ua.includes('Chrome/6');
  
  return {
    secure: true,
    httpOnly: true,
    sameSite: hasSameSiteSupport ? 'lax' : undefined
    // Fallback for old browsers that don't support SameSite
  };
}
```

### Advanced Examples

```javascript
// Advanced: SameSite partitioning (CHIPS)
// Cookies with Partitioned attribute are scoped to the top-level site
// This allows embedded services to have cookies without leaking across sites
res.cookie('embedPrefs', 'dark', {
  sameSite: 'none',
  secure: true,
  partitioned: true // Cookie is scoped to the embedding site
});

// Advanced: Testing SameSite behavior
// Middleware to log SameSite for debugging
app.use((req, res, next) => {
  const cookies = req.headers['cookie'];
  const sameSiteInfo = {
    cookiesSet: !!cookies,
    sameSite: req.session?.cookie?.sameSite,
    origin: req.headers['origin'],
    referer: req.headers['referer']
  };
  console.log('SameSite context:', sameSiteInfo);
  next();
});
```

### Real-World Use Cases

- Social login flows (OAuth callbacks use top-level navigation, compatible with Lax)
- E-commerce checkout (multi-step forms, Lax allows initial navigation)
- Banking applications (Strict for sensitive operations)
- Embedded widgets and iframes (None with Partitioned)
- Analytics services tracking across sites (None)

### Common Mistakes

- Setting SameSite=None without the Secure flag (rejected by modern browsers)
- Using SameSite=Strict for all sites (breaks user experience for external links)
- Assuming SameSite protects against all CSRF (subdomain-based attacks still possible)
- Not testing SameSite behavior across different browsers
- Forgetting that SameSite only applies to cookies, not other auth mechanisms
- Not handling the case where SameSite causes session loss

### Best Practices

- Use SameSite=Lax as the default for most web applications
- Use SameSite=Strict for high-security applications (banking, admin panels)
- Only use SameSite=None when third-party cookie access is required (embedded content)
- Always set Secure=true when using SameSite=None
- Implement server-side CSRF tokens as defense-in-depth even with SameSite
- Test SameSite behavior across major browsers
- Consider user experience impact when choosing Strict vs Lax

### Performance Considerations

- SameSite enforcement is entirely browser-side, zero server overhead
- No additional network round-trips or computation
- Cookie header size is unchanged
- No storage or memory impact on the server
- The only "cost" is potential session loss leading to re-authentication requests

### Interview Questions

**Q: Explain the difference between SameSite=Lax and SameSite=Strict.**
A: SameSite=Strict never sends the cookie with cross-site requests, even for top-level navigations via links. This prevents CSRF completely but breaks user experience when a user clicks a link from another site (they appear logged out). SameSite=Lax sends cookies for top-level navigations using safe HTTP methods (typically GET), so clicking a link from another site preserves the session, but POST forms or JavaScript-initiated requests from other sites don't include the cookie.

**Q: What is the default SameSite behavior in modern browsers?**
A: As of 2021, all major browsers (Chrome 84+, Firefox 79+, Safari 13.1+) treat cookies without a SameSite attribute as SameSite=Lax by default. Previously, the default was effectively SameSite=None. This change significantly reduces CSRF risk for applications that haven't explicitly set SameSite.

### Coding Challenges

**Challenge 1:** Configure session cookies with appropriate SameSite settings for a multi-tenant app embedded in iframes.

**Solution:**

```javascript
function getSessionConfig(isEmbedded = false) {
  if (isEmbedded) {
    return {
      secret: process.env.SESSION_SECRET,
      resave: false,
      saveUninitialized: false,
      cookie: {
        sameSite: 'none',
        secure: true,
        httpOnly: true,
        partitioned: true // Chrome's CHIPS
      }
    };
  }
  
  return {
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      sameSite: 'lax',
      secure: process.env.NODE_ENV === 'production',
      httpOnly: true
    }
  };
}
```

## CSRF Prevention Best Practices

### What It Is

CSRF prevention encompasses a layered set of techniques that collectively protect web applications against cross-site request forgery. No single measure provides complete protection; defense-in-depth combines browser-level protections (SameSite cookies), application-level protections (CSRF tokens), and architectural decisions (RESTful API design, token-based authentication).

### Why It Is Important

CSRF attacks remain relevant because applications evolve, misconfigurations happen, and browser behavior changes. A single vulnerability in one layer can be mitigated by another layer. Comprehensive prevention strategies ensure that even if a CSRF token is compromised or SameSite is misconfigured, the application remains protected. Understanding the full landscape of prevention techniques allows developers to make informed security decisions.

### How It Works Internally

The layered defense strategy works as follows:

1. **Browser layer (SameSite cookies)**: The browser refuses to attach cookies to cross-origin requests based on policy. This is the first line of defense and requires no server-side verification.
2. **Application layer (CSRF tokens)**: The server validates a token on every state-changing request. This protects against CSRF even when SameSite is set to None or when cookies are sent for other reasons.
3. **Architectural layer (API design)**: Using token-based authentication (JWT, OAuth) in custom headers rather than cookies eliminates automatic credential attachment.
4. **Origin validation (Referer/Origin headers)**: The server checks incoming request headers to verify the request originated from a trusted source.

### Syntax

```javascript
// Complete CSRF prevention middleware
const crypto = require('crypto');

function comprehensiveCsrfProtection(req, res, next) {
  const methods = ['POST', 'PUT', 'DELETE', 'PATCH'];
  if (!methods.includes(req.method)) return next();
  
  // Layer 1: Origin/Referer header check
  const origin = req.headers['origin'];
  const referer = req.headers['referer'];
  const allowedOrigins = ['https://myapp.com', 'https://admin.myapp.com'];
  
  if (origin && !allowedOrigins.includes(origin)) {
    return res.status(403).json({ error: 'Cross-origin request denied' });
  }
  
  // Layer 2: CSRF token validation
  const token = req.headers['x-csrf-token'] || req.body._csrf;
  const storedToken = req.session?.csrfToken;
  
  if (!token || !storedToken) {
    return res.status(403).json({ error: 'CSRF token missing' });
  }
  
  try {
    if (!crypto.timingSafeEqual(Buffer.from(token), Buffer.from(storedToken))) {
      return res.status(403).json({ error: 'CSRF token invalid' });
    }
  } catch {
    return res.status(403).json({ error: 'CSRF validation error' });
  }
  
  next();
}

// Layer 3: SameSite cookie
app.use(session({
  cookie: {
    sameSite: 'lax',
    secure: true,
    httpOnly: true
  }
}));

app.use(comprehensiveCsrfProtection);
```

### Beginner Examples

```javascript
// Minimal CSRF prevention checklist
const express = require('express');
const app = express();

// 1. Use framework CSRF protection
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });
app.use(csrfProtection);

// 2. Set secure cookies
app.use(session({
  cookie: {
    sameSite: 'lax',
    secure: true,
    httpOnly: true
  }
}));

// 3. Expose token for client-side usage
app.get('/api/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// 4. Validate on state-changing endpoints
app.post('/api/update-profile', (req, res) => {
  // csurf middleware already validated the token
  // ...
});
```

### Intermediate Examples

```javascript
// Intermediate: Custom CSRF protection for SPA
const crypto = require('crypto');

class CsrfService {
  constructor(secret, options = {}) {
    this.secret = secret;
    this.tokenLength = options.tokenLength || 32;
    this.tokenExpiry = options.tokenExpiry || 86400000; // 24h
  }
  
  generateToken(sessionId) {
    const random = crypto.randomBytes(this.tokenLength).toString('hex');
    const expiry = Date.now() + this.tokenExpiry;
    const data = `${sessionId}:${random}:${expiry}`;
    const signature = crypto
      .createHmac('sha256', this.secret)
      .update(data)
      .digest('hex');
    return Buffer.from(`${data}:${signature}`).toString('base64url');
  }
  
  validateToken(token, sessionId) {
    try {
      const decoded = Buffer.from(token, 'base64url').toString();
      const parts = decoded.split(':');
      if (parts.length !== 4) return false;
      
      const [sid, random, expiry, signature] = parts;
      if (sid !== sessionId) return false;
      if (Date.now() > parseInt(expiry)) return false;
      
      const data = `${sid}:${random}:${expiry}`;
      const expectedSig = crypto
        .createHmac('sha256', this.secret)
        .update(data)
        .digest('hex');
      
      return crypto.timingSafeEqual(
        Buffer.from(signature),
        Buffer.from(expectedSig)
      );
    } catch {
      return false;
    }
  }
}

// API usage
const csrfService = new CsrfService(process.env.CSRF_SECRET);

app.use('/api', (req, res, next) => {
  if (['POST', 'PUT', 'DELETE', 'PATCH'].includes(req.method)) {
    const token = req.headers['x-csrf-token'];
    if (!token || !csrfService.validateToken(token, req.session.id)) {
      return res.status(403).json({ error: 'Invalid CSRF token' });
    }
  }
  next();
});
```

### Advanced Examples

```javascript
// Advanced: Multi-layer CSRF prevention for microservices
class CSRFGateway {
  constructor(config) {
    this.allowedOrigins = config.allowedOrigins;
    this.secret = config.secret;
    this.tokenExpiry = config.tokenExpiry || 300000; // 5 min
  }
  
  // Layer 1: Origin validation
  validateOrigin(req) {
    const origin = req.headers['origin'] || req.headers['referer'];
    if (!origin) return false;
    
    try {
      const url = new URL(origin);
      return this.allowedOrigins.some(allowed => 
        url.origin === allowed || url.hostname.endsWith('.' + new URL(allowed).hostname)
      );
    } catch {
      return false;
    }
  }
  
  // Layer 2: Custom header check for AJAX
  validateCustomHeader(req) {
    const header = req.headers['x-requested-with'];
    return header === 'XMLHttpRequest';
  }
  
  // Layer 3: Token validation
  validateCsrfToken(req) {
    const token = req.headers['x-csrf-token'];
    const userToken = req.session?.csrfToken;
    return token && userToken && crypto.timingSafeEqual(
      Buffer.from(token),
      Buffer.from(userToken)
    );
  }
  
  // Combined check
  validate(req) {
    // Bypass for idempotent methods
    if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) return true;
    
    const checks = [
      this.validateOrigin(req),
      this.validateCustomHeader(req),
      this.validateCsrfToken(req)
    ];
    
    // At least 2 of 3 must pass
    return checks.filter(Boolean).length >= 2;
  }
}

// Usage in gateway
const gateway = new CSRFGateway({
  allowedOrigins: ['https://app.example.com'],
  secret: process.env.CSRF_SECRET
});

app.use((req, res, next) => {
  if (!gateway.validate(req)) {
    return res.status(403).json({ error: 'CSRF check failed' });
  }
  next();
});
```

### Real-World Use Cases

- Enterprise applications with strict security compliance (PCI DSS, HIPAA)
- Multi-tenant SaaS platforms with custom domains
- Microservice architectures where each service needs independent protection
- Legacy applications being retrofitted with modern security
- Mobile apps using web views for authentication flows

### Common Mistakes

- Relying on only one prevention technique
- Not regenerating CSRF tokens after login
- Exposing CSRF tokens in URLs or console logs
- Using CSRF only on POST but not PUT/DELETE/PATCH
- Not protecting file upload endpoints
- Forgetting to check CSRF on API routes that appear to be read-only but have side effects

### Best Practices

- Implement defense-in-depth: SameSite cookies + CSRF tokens + Origin header checks
- Use framework-provided CSRF protection when available
- For APIs, prefer token-based authentication (Bearer tokens) over cookies
- Implement proper CORS configuration alongside CSRF protection
- Regularly test CSRF protection with automated scanners
- Keep SameSite behavior in mind when designing user flows
- Document CSRF protection requirements for third-party integrators
- Use double-submit cookie pattern for stateless architectures

### Performance Considerations

- Origin header parsing adds negligible overhead
- Timing-safe comparisons are marginally slower than direct `===` but necessary for security
- Multi-layer validation adds <0.5ms per request in total
- Stateless token validation avoids session store lookups
- Session-based token storage may add latency in distributed systems
- CDN and caching layers should not cache responses with CSRF tokens

### Interview Questions

**Q: What is the defense-in-depth approach to CSRF prevention?**
A: Layer 1: SameSite cookies tell the browser when to attach credentials. Layer 2: CSRF tokens require a secret value that an attacker cannot forge. Layer 3: Origin/Referer header validation verifies the request source. Layer 4: Custom headers (X-Requested-With) as additional validation for AJAX requests. If one layer fails, others still provide protection.

**Q: Why are APIs with JWT authentication naturally CSRF-resistant?**
A: JWTs are typically sent in the Authorization header, not in cookies. The browser does not automatically attach Authorization headers to cross-origin requests. The JavaScript code must explicitly set the header, and cross-origin JavaScript cannot read the token from another origin. Additionally, CORS must be explicitly configured to allow credentials. This eliminates the automatic credential attachment that enables CSRF.

### Coding Challenges

**Challenge 1:** Implement a comprehensive CSRF prevention strategy for an Express application.

**Solution:**

```javascript
const express = require('express');
const session = require('express-session');
const crypto = require('crypto');

const app = express();

// Layer 1: SameSite cookies
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  name: '__Host-session', // Prefix indicates cookie is host-only
  cookie: {
    sameSite: 'lax',
    secure: true,
    httpOnly: true,
    path: '/'
  }
}));

// Layer 2: CSRF token middleware
app.use((req, res, next) => {
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString('hex');
  }
  
  res.locals.csrfToken = req.session.csrfToken;
  
  next();
});

// Layer 3: Token validation on state changes
app.use((req, res, next) => {
  const methods = ['POST', 'PUT', 'DELETE', 'PATCH'];
  if (!methods.includes(req.method)) return next();
  
  const token = req.headers['x-csrf-token'] || req.body._csrf;
  
  if (!token || !crypto.timingSafeEqual(
    Buffer.from(token),
    Buffer.from(req.session.csrfToken)
  )) {
    return res.status(403).json({ error: 'CSRF validation failed' });
  }
  
  next();
});

// Layer 4: Origin validation for extra protection
app.use((req, res, next) => {
  const methods = ['POST', 'PUT', 'DELETE', 'PATCH'];
  if (!methods.includes(req.method)) return next();
  
  const origin = req.headers['origin'];
  if (origin && origin !== 'https://myapp.com') {
    return res.status(403).json({ error: 'Cross-origin request blocked' });
  }
  
  next();
});
```

### Related Topics

- SameSite cookie attribute (detailed in previous section)
- CSRF tokens and generation strategies
- Content Security Policy (CSP)
- CORS (Cross-Origin Resource Sharing)
- Cookie prefixes (__Host-, __Secure-)
- Session management and security
- OAuth 2.0 and state parameter for CSRF protection in OAuth flows
- HTTPOnly and Secure cookie flags
- Double-submit cookie pattern
- Signed cookies vs encrypted cookies
