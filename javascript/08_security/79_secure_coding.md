# Secure Coding - Input validation, output encoding, HTTPS, dependency security

## Introduction

Secure coding in JavaScript encompasses the practices that protect applications from security vulnerabilities. As JavaScript has expanded from client-side scripting to server-side (Node.js), mobile apps, and desktop applications, the attack surface has grown. This file covers input validation, output encoding, HTTPS enforcement, and dependency vulnerability management—the four pillars of defensive JavaScript development.

## Input Validation

### What It Is

Input validation verifies that user-supplied data conforms to expected formats, types, lengths, and ranges before processing. Validation can be syntactic (checking email format) or semantic (verifying a date is not in the past). It is the first line of defense against injection attacks, data corruption, and business logic abuse.

### Why It Is Important

Unvalidated input is the root cause of SQL injection, NoSQL injection, command injection, XSS, path traversal, and LDAP injection. Every external data source—form fields, URL parameters, HTTP headers, file uploads, API payloads, WebSocket messages—must be validated. OWASP and PCI DSS mandate input validation as a core security practice.

### How It Works Internally

Input validation establishes trust boundaries. Data from outside the boundary is checked against an allowlist of permitted patterns. Allowlist validation is strongly preferred: define what is permitted and reject everything else. Validation occurs before any processing, and the validated data is then used in queries, rendering, or storage.

### Syntax

```javascript
function validateUsername(input) {
  if (typeof input !== 'string') return null;
  if (input.length < 3 || input.length > 30) return null;
  if (!/^[a-zA-Z0-9_]+$/.test(input)) return null;
  return input;
}

function validateEmail(input) {
  if (typeof input !== 'string') return null;
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(input) || input.length > 254) return null;
  return input.toLowerCase();
}

function validateAge(input) {
  const age = parseInt(input, 10);
  if (isNaN(age) || age < 0 || age > 150) return null;
  return age;
}
```

### Beginner Examples

```javascript
app.post('/register', (req, res) => {
  const { username, email, age } = req.body;
  if (!username || typeof username !== 'string' || username.length < 3)
    return res.status(400).json({ error: 'Invalid username' });
  if (!email || !email.includes('@') || email.length > 254)
    return res.status(400).json({ error: 'Invalid email' });
  const parsedAge = parseInt(age, 10);
  if (isNaN(parsedAge) || parsedAge < 13 || parsedAge > 120)
    return res.status(400).json({ error: 'Invalid age' });
  createUser({ username, email, age: parsedAge });
  res.status(201).json({ success: true });
});
```

### Intermediate Examples

```javascript
const Joi = require('joi');
const userSchema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  password: Joi.string().pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/).required(),
  email: Joi.string().email().max(254).required(),
  age: Joi.number().integer().min(13).max(120).required(),
  role: Joi.string().valid('user', 'moderator', 'admin').default('user')
});

function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, { abortEarly: false, stripUnknown: true });
    if (error) return res.status(400).json({ errors: error.details.map(d => d.message) });
    req.validatedBody = value;
    next();
  };
}

app.post('/register', validate(userSchema), (req, res) => {
  createUser(req.validatedBody);
  res.status(201).json({ success: true });
});
```

### Advanced Examples

```javascript
class InputValidator {
  constructor() {
    this.rules = new Map();
    this.addRule('email', v => typeof v === 'string' && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v) && v.length <= 254);
    this.addRule('username', v => typeof v === 'string' && /^[a-zA-Z0-9_-]{3,30}$/.test(v));
    this.addRule('mongoId', v => typeof v === 'string' && /^[a-f\d]{24}$/i.test(v));
  }

  addRule(name, validator) { this.rules.set(name, validator); }

  validate(data, schema) {
    const errors = []; const cleaned = {};
    for (const [key, rules] of Object.entries(schema)) {
      const value = data[key];
      if (rules.required && (value === undefined || value === null)) { errors.push(`${key} is required`); continue; }
      if (value === undefined || value === null) { cleaned[key] = rules.default ?? value; continue; }
      if (rules.type && typeof value !== rules.type) { errors.push(`${key} must be ${rules.type}`); continue; }
      if (rules.rule && this.rules.has(rules.rule) && !this.rules.get(rules.rule)(value)) { errors.push(`Invalid ${rules.rule}`); continue; }
      if (rules.min !== undefined && value < rules.min) { errors.push(`${key} min ${rules.min}`); continue; }
      if (rules.max !== undefined && value > rules.max) { errors.push(`${key} max ${rules.max}`); continue; }
      cleaned[key] = rules.transform ? rules.transform(value) : value;
    }
    return { cleaned, errors };
  }

  sanitizeDeep(input) {
    if (typeof input === 'string') return input.replace(/[\x00-\x08\x0B\x0C\x0E-\x1F\x7F]/g, '');
    if (Array.isArray(input)) return input.map(i => this.sanitizeDeep(i));
    if (input && typeof input === 'object') {
      const sanitized = {};
      for (const [key, value] of Object.entries(input)) {
        if (['__proto__', 'constructor', 'prototype'].includes(key)) continue;
        sanitized[key] = this.sanitizeDeep(value);
      }
      return sanitized;
    }
    return input;
  }
}
```

### Real-World Use Cases

Registration forms, payment processing, file upload systems, API endpoints, search functionality, configuration validation.

### Common Mistakes

Client-side-only validation, blacklists instead of allowlists, trusting HTTP headers, not validating numeric ranges, failing to validate nested structures, using `typeof` without null guards.

### Best Practices

Server-side validation always, allowlist approach, validate all input sources, use proven libraries (Joi, Zod, Yup), apply least privilege, reject early.

### Performance Considerations

Validation adds <1ms typically. Complex regex can be slow. Schema libraries have startup costs but optimize repeated use.

### Interview Questions

**Q: Why is allowlist validation better than blocklist?** A: Allowlists define what is permitted and reject everything else. Blocklists must enumerate all possible attacks, which is infinite and incomplete.

**Q: What is prototype pollution?** A: An attacker modifies `Object.prototype` via `__proto__` or `constructor.prototype`, affecting all objects. Block these keys during input processing.

### Coding Challenges

**Challenge:** Create validation middleware that validates request body against a schema.

```javascript
function validateBody(schema) {
  return (req, res, next) => {
    const errors = []; const validated = {};
    for (const [field, rules] of Object.entries(schema)) {
      const value = req.body[field];
      if (rules.required && (value === undefined || value === null)) { errors.push(`${field} required`); continue; }
      if (value === undefined || value === null) { validated[field] = rules.default; continue; }
      if (rules.type && typeof value !== rules.type) { errors.push(`${field} must be ${rules.type}`); continue; }
      if (rules.minLength && value.length < rules.minLength) errors.push(`${field} min ${rules.minLength} chars`);
      if (rules.maxLength && value.length > rules.maxLength) errors.push(`${field} max ${rules.maxLength} chars`);
      if (rules.pattern && !rules.pattern.test(value)) errors.push(`${field} invalid format`);
      if (rules.enum && !rules.enum.includes(value)) errors.push(`${field} must be one of: ${rules.enum.join(', ')}`);
      validated[field] = value;
    }
    if (errors.length > 0) return res.status(400).json({ errors });
    req.validatedBody = validated;
    next();
  };
}
```

## Output Encoding

### What It Is

Output encoding converts special characters into safe representations before rendering. Different contexts need different encoding: HTML entity encoding for HTML, URL encoding for URLs, JavaScript string escaping for `<script>` contexts, and CSS escaping for styles.

### Why It Is Important

Output encoding is the primary XSS defense. Even if validation fails, encoding ensures `<`, `>`, `"`, `'`, and `&` are rendered harmlessly. Context-aware encoding is critical since encoding for one context may not suffice for another.

### How It Works Internally

Special characters are transformed: `<` becomes `&lt;`, `>` becomes `&gt;`, `&` becomes `&amp;`, `"` becomes `&quot;`, `'` becomes `&#x27;`. The browser renders entities as literal characters without interpreting as code.

### Syntax

```javascript
const he = require('he');
he.encode('<script>alert(1)</script>');
// &lt;script&gt;alert(1)&lt;/script&gt;

function encodeAttribute(value) {
  return value.replace(/"/g, '&quot;').replace(/&/g, '&amp;');
}

function encodeJsString(value) {
  return value.replace(/\\/g, '\\\\').replace(/'/g, "\\'")
    .replace(/"/g, '\\"').replace(/\n/g, '\\n')
    .replace(/<\/script>/gi, '<\\/script>');
}
```

### Beginner Examples

```javascript
// EJS auto-escapes with <%= %>
// <p>Welcome, <%= user.name %></p>  <!-- SAFE -->
// <p>Welcome, <%- user.name %></p>  <!-- UNSAFE -->

// React auto-escapes by default
function UserGreeting({ name }) {
  return <p>Welcome, {name}</p>; // SAFE
}
// dangerouslySetInnerHTML does NOT escape
function RichContent({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />; // Sanitize first!
}
```

### Intermediate Examples

```javascript
class OutputEncoder {
  static forHtml(str) {
    if (str == null) return '';
    return he.encode(String(str), { useNamedReferences: true });
  }
  static forHtmlAttribute(str) {
    if (str == null) return '';
    return String(str).replace(/&/g, '&amp;').replace(/"/g, '&quot;').replace(/'/g, '&#x27;');
  }
  static forUrl(str) {
    if (str == null) return '';
    return encodeURIComponent(String(str));
  }
  static forJavaScript(str) {
    if (str == null) return '';
    return JSON.stringify(String(str)).slice(1, -1).replace(/<\//g, '<\\/');
  }
}

const html = `
  <div data-user="${OutputEncoder.forHtmlAttribute(user.name)}">
    <p>${OutputEncoder.forHtml(user.name)}</p>
    <a href="/profile?name=${OutputEncoder.forUrl(user.name)}">Profile</a>
    <script>var userName = "${OutputEncoder.forJavaScript(user.name)}";</script>
  </div>
`;
```

### Advanced Examples

```javascript
class SafeStringRenderer {
  constructor() { this.contexts = ['html']; this.buffer = []; }
  
  text(str) {
    const ctx = this.contexts[this.contexts.length - 1];
    switch (ctx) {
      case 'html': this.buffer.push(OutputEncoder.forHtml(str)); break;
      case 'attribute': this.buffer.push(OutputEncoder.forHtmlAttribute(str)); break;
      case 'url': this.buffer.push(OutputEncoder.forUrl(str)); break;
      case 'javascript': this.buffer.push(OutputEncoder.forJavaScript(str)); break;
      default: this.buffer.push(String(str));
    }
  }
  
  html(open, close) {
    if (open) this.buffer.push(open);
    return {
      attr: (name) => {
        this.buffer.push(` ${name}="`);
        this.contexts.push('attribute');
        return {
          val: (v) => { OutputEncoder.forHtmlAttribute(v); this.contexts.pop(); return this; }
        };
      },
      end: () => { if (close) this.buffer.push(close); return this; }
    };
  }
  
  toString() { return this.buffer.join(''); }
}
```

### Real-World Use Cases

Template rendering, rich text editors, email generation, PDF generation, API response formatting.

### Common Mistakes

Using HTML encoding in JavaScript contexts, double-encoding, forgetting to encode in attributes (href, src), not encoding JSON strings in `<script>` tags.

### Best Practices

Context-aware encoding, use template engines with auto-escaping, prefer `textContent` over `innerHTML`, use DOMPurify for HTML content, never trust `dangerouslySetInnerHTML`.

### Performance Considerations

Encoding adds minimal overhead (<0.1ms per string). Libraries like `he` are optimized. Auto-escaping in templates has negligible cost.

### Interview Questions

**Q: Why must output encoding be context-aware?** A: Encoding for HTML body (`&lt;`) does not protect in JavaScript strings (need `\\n` escaping) or URLs (need percent-encoding). Using wrong encoding can still be exploitable.

**Q: How does React's JSX prevent XSS?** A: React escapes all string content by default in JSX expressions `{...}`. Values are converted to strings with HTML entity encoding before insertion.

## HTTPS Enforcement

### What It Is

HTTPS (HTTP over TLS) encrypts all communication between client and server, protecting data in transit from eavesdropping, tampering, and man-in-the-middle attacks. Enforcement means ensuring all traffic uses HTTPS, redirecting HTTP to HTTPS, and using security headers like HSTS.

### Why It Is Important

Without HTTPS, all data—passwords, cookies, API tokens, personal information—is sent in plain text. Attackers on the same network can intercept credentials, inject malicious content, or hijack sessions. HTTPS is mandatory for HTTP/2, service workers, geolocation, and many modern browser APIs. Search engines penalize non-HTTPS sites.

### How It Works Internally

TLS handshake establishes an encrypted channel: client sends supported cipher suites, server presents its certificate (signed by a trusted CA), they exchange keys via asymmetric cryptography, then switch to symmetric encryption for the session. Modern TLS 1.3 completes in 1 round trip (0-RTT for resumed sessions).

### Syntax

```javascript
const https = require('https');
const fs = require('fs');
const express = require('express');
const app = express();

const options = {
  key: fs.readFileSync('/etc/ssl/private/key.pem'),
  cert: fs.readFileSync('/etc/ssl/certs/cert.pem')
};

https.createServer(options, app).listen(443);

// Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (!req.secure && req.headers['x-forwarded-proto'] !== 'https') {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

### Beginner Examples

```javascript
const helmet = require('helmet');
app.use(helmet()); // Sets HSTS and other security headers

// Strict HSTS header
app.use(helmet.hsts({
  maxAge: 31536000,      // 1 year
  includeSubDomains: true,
  preload: true
}));

// Upgrade insecure requests
app.use((req, res, next) => {
  if (req.protocol === 'http' && process.env.NODE_ENV === 'production') {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

### Intermediate Examples

```javascript
// TLS configuration with strong ciphers
const options = {
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem'),
  minVersion: 'TLSv1.2',
  ciphers: [
    'TLS_AES_256_GCM_SHA384',
    'TLS_CHACHA20_POLY1305_SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256'
  ].join(':'),
  honorCipherOrder: true
};

// HTTP/2 with HTTPS
const http2 = require('http2');
const server = http2.createSecureServer(options, app);

// Certificate management with Let's Encrypt
const acme = require('acme-client');
// Automated certificate renewal
```

### Advanced Examples

```javascript
// Mutual TLS (mTLS) for service-to-service communication
const tls = require('tls');

const server = tls.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
  ca: [fs.readFileSync('client-ca.pem')],
  requestCert: true,
  rejectUnauthorized: true
}, (socket) => {
  const clientCert = socket.getPeerCertificate();
  if (!clientCert.subject) { socket.destroy(); return; }
  console.log(`Authenticated client: ${clientCert.subject.CN}`);
});

// OCSP stapling for performance
const options = {
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem'),
  requestOCSP: true
};
```

### Real-World Use Cases

All production web applications, payment processing, authentication flows, API endpoints, WebSocket connections (WSS), service mesh communication.

### Common Mistakes

Mixed content (HTTPS page loading HTTP resources), not redirecting all traffic, weak TLS ciphers, expired certificates, not using HSTS, incorrect reverse proxy headers.

### Best Practices

Enforce HTTPS at the reverse proxy level (Nginx, HAProxy), use Let's Encrypt for free certificates, set HSTS with `preload`, use secure TLS 1.2+ only, redirect all HTTP traffic, set `Secure` flag on all cookies.

### Performance Considerations

TLS handshake adds latency (1-2 RTT). TLS 1.3 reduces this. Session resumption eliminates handshake for returning visitors. OCSP stapling improves performance. Hardware acceleration (AES-NI) makes encryption fast. The performance cost is minimal (<5% CPU overhead).

### Interview Questions

**Q: What is HSTS and how does it prevent downgrade attacks?** A: HTTP Strict Transport Security tells browsers to always use HTTPS for a domain. Once set, the browser refuses HTTP connections, preventing SSL stripping attacks. The `preload` directive allows submission to browser hardcoded lists.

**Q: What is mixed content and why is it dangerous?** A: Mixed content occurs when an HTTPS page loads HTTP resources (scripts, stylesheets, images). HTTP scripts can be modified by attackers to compromise the entire page. Modern browsers block active mixed content by default.

## Dependency Vulnerability Scanning

### What It Is

Dependency vulnerability scanning is the automated process of identifying known security vulnerabilities in third-party packages and libraries. Tools like `npm audit`, Snyk, and Dependabot scan `package-lock.json` against the National Vulnerability Database (NVD) and other advisory databases to flag vulnerable dependencies.

### Why It Is Important

Modern JavaScript applications often have hundreds or thousands of dependencies (direct + transitive). A vulnerability in any single dependency—even a deeply nested transitive one—can compromise the entire application. High-profile attacks like event-stream (cryptocurrency theft), left-pad (supply chain disruption), and ua-parser-js (malware) demonstrate the critical nature of this practice.

### How It Works Internally

The scanner reads the dependency tree from `package-lock.json` or `yarn.lock`, which contains exact versions of every installed package. Each package name and version is compared against a vulnerability database (maintained by npm, GitHub, Snyk, etc.). When a match is found, the advisory details are reported: severity, vulnerable versions, patched versions, CVE identifiers, and remediation steps.

### Syntax

```bash
# npm audit
npm audit                     # Scan and report
npm audit fix                 # Auto-fix vulnerabilities
npm audit --audit-level=high  # Only report high/critical

# yarn
yarn audit

# Using snyk
npm install -g snyk
snyk test
snyk monitor                  # Continuous monitoring
```

### Beginner Examples

```json
// package.json scripts
{
  "scripts": {
    "audit": "npm audit",
    "audit:fix": "npm audit fix",
    "predeploy": "npm run audit && npm test"
  }
}
```

```yaml
# GitHub Actions - Dependabot
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "security"
```

### Intermediate Examples

```javascript
// Programmatic npm audit
const { execSync } = require('child_process');

function auditDependencies() {
  try {
    const output = execSync('npm audit --json', { encoding: 'utf8' });
    const audit = JSON.parse(output);
    
    if (audit.metadata.vulnerabilities.high > 0 || 
        audit.metadata.vulnerabilities.critical > 0) {
      console.error('Security vulnerabilities found!');
      console.error(`High: ${audit.metadata.vulnerabilities.high}`);
      console.error(`Critical: ${audit.metadata.vulnerabilities.critical}`);
      process.exit(1);
    }
    
    console.log('No high/critical vulnerabilities found.');
  } catch (err) {
    if (err.stdout) {
      // npm audit exits non-zero when vulnerabilities found
      const audit = JSON.parse(err.stdout);
      console.error('Vulnerabilities:', audit.metadata.vulnerabilities);
    }
    process.exit(1);
  }
}
```

### Advanced Examples

```javascript
// Custom dependency scanning with advisory database
const https = require('https');

class DependencyScanner {
  constructor() {
    this.advisoryDb = new Map();
    this.init();
  }
  
  async init() {
    // Fetch NPM advisory database
    // See: https://registry.npmjs.org/-/npm/v1/security/advisories
  }
  
  async scan(packageJson, lockFile) {
    const dependencies = this.flattenDependencies(lockFile);
    const vulnerabilities = [];
    
    for (const [name, version] of Object.entries(dependencies)) {
      const advisory = await this.checkAdvisory(name, version);
      if (advisory) {
        vulnerabilities.push({
          package: name,
          installed: version,
          advisory: advisory.title,
          severity: advisory.severity,
          cve: advisory.cve,
          patchedIn: advisory.patched_versions
        });
      }
    }
    
    return vulnerabilities;
  }
  
  flattenDependencies(lockFile) {
    const deps = {};
    for (const [pkg, data] of Object.entries(lockFile.packages || lockFile.dependencies || {})) {
      if (pkg) deps[pkg] = data.version;
    }
    return deps;
  }
  
  async checkAdvisory(name, version) {
    // Check against local or remote advisory database
    return null; // Simplified for example
  }
  
  generateReport(vulnerabilities) {
    return {
      scanned: new Date().toISOString(),
      total: vulnerabilities.length,
      bySeverity: {
        critical: vulnerabilities.filter(v => v.severity === 'critical').length,
        high: vulnerabilities.filter(v => v.severity === 'high').length,
        moderate: vulnerabilities.filter(v => v.severity === 'moderate').length,
        low: vulnerabilities.filter(v => v.severity === 'low').length
      },
      vulnerabilities: vulnerabilities.sort((a, b) => 
        ['critical','high','moderate','low'].indexOf(a.severity) - 
        ['critical','high','moderate','low'].indexOf(b.severity)
      )
    };
  }
}

// CI/CD integration
async function ciSecurityCheck() {
  const scanner = new DependencyScanner();
  const pkg = require('./package.json');
  const lock = require('./package-lock.json');
  
  const vulnerabilities = await scanner.scan(pkg, lock);
  
  if (vulnerabilities.some(v => ['critical', 'high'].includes(v.severity))) {
    console.error('❌ Security check failed');
    console.error(JSON.stringify(scanner.generateReport(vulnerabilities), null, 2));
    process.exit(1);
  }
  
  console.log('✅ Security check passed');
}
```

### Real-World Use Cases

CI/CD pipeline gates, pre-deployment checks, scheduled security audits, GitHub Dependabot alerts, Snyk continuous monitoring, OWASP Dependency-Check integration.

### Common Mistakes

Not scanning transitive (nested) dependencies, ignoring moderate/low severity advisories, not updating lock files, running audit only in production, not having automated alerts for new vulnerabilities.

### Best Practices

Run `npm audit` in CI/CD as a blocking gate, use Dependabot or Renovate for automated PRs, pin exact versions in lock file, monitor CVEs for all dependencies, remove unused dependencies, use tools like `depcheck` to find unused packages, consider using `--omit=dev` in production for dev-only packages.

### Performance Considerations

Scanning time depends on dependency count (typically 1-5 seconds). CI integration adds negligible overhead. Lock file analysis is fast local operation.

### Interview Questions

**Q: What is the difference between a direct dependency and a transitive dependency?** A: Direct dependencies are packages your code explicitly imports. Transitive (nested) dependencies are packages that your direct dependencies use. Both can introduce vulnerabilities. `npm audit` scans the entire dependency tree, including transitive dependencies.

**Q: How does `npm audit fix` work?** A: It reads advisory data, identifies vulnerable packages, and updates to the nearest safe version that doesn't break semver. If a breaking change is needed, it reports the issue but does not auto-fix. Use `npm audit fix --force` for breaking changes (with caution).

### Related Topics

- OWASP Top 10 vulnerability categories
- Input validation and sanitization frameworks
- TLS/SSL certificate management and ACME protocol
- Content Security Policy (CSP) as a mitigating control
- Subresource Integrity (SRI) for CDN assets
- HTTP security headers (HSTS, X-Frame-Options, X-Content-Type-Options)
- Secret management (environment variables, vault solutions)
- Security.txt standard for vulnerability disclosure
- Software Bill of Materials (SBOM) generation
- Supply chain security and SLSA framework
