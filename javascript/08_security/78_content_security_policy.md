# Content Security Policy - CSP headers, directives, report-uri, nonce/hash

## Introduction

Content Security Policy (CSP) is a browser security mechanism that provides a declarative allowlist of approved content sources for a web page. Using HTTP headers or meta tags, CSP tells the browser which origins and types of resources are permitted, effectively preventing a wide range of attacks including Cross-Site Scripting (XSS), clickjacking, and data injection. CSP acts as a defense-in-depth layer that can mitigate the impact of vulnerabilities even when other security measures fail. For JavaScript developers, understanding CSP is essential for building secure web applications, especially when integrating third-party scripts, fonts, or other external resources.

## CSP Headers (Content-Security-Policy)

### What It Is

The `Content-Security-Policy` HTTP response header instructs the browser about which dynamic resources are allowed to load on a page. It defines a set of directives that control connections, script execution, styling, font loading, form submissions, and more. When a browser receives a page with a CSP header, it enforces the policy by blocking or reporting violations. Policies can be delivered via the `Content-Security-Policy` header (enforcement) or `Content-Security-Policy-Report-Only` header (monitoring only).

### Why It Is Important

CSP is one of the most effective defenses against XSS attacks because it can block inline scripts and restrict which external scripts can execute, even if an attacker manages to inject a `<script>` tag. CSP also mitigates other attacks: it prevents clickjacking (`frame-ancestors`), restricts form submissions (`form-action`), blocks mixed content (`block-all-mixed-content`), and prevents data exfiltration via outbound connections (`connect-src`). Implementing CSP is a security best practice recommended by OWASP, Google, and Mozilla.

### How It Works Internally

When the browser receives a page with a CSP header, it parses the policy string into a set of directive-source pairs. For each resource type (script, style, image, connect, font, etc.), the browser builds an internal allowlist. Whenever the page attempts to load or execute a resource, the browser checks it against the corresponding directive. If the resource's source matches an entry in the allowlist, the resource is loaded; otherwise, it's blocked and a violation report is optionally sent to the `report-uri` endpoint. The browser evaluates CSP before any JavaScript execution, making it a robust security boundary.

### Syntax

```http
Content-Security-Policy: directive1 source1 source2; directive2 source3 source4
```

```javascript
// Setting CSP via Express middleware
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' https://apis.google.com; style-src 'self' 'unsafe-inline'"
  );
  next();
});

// Setting CSP via HTML meta tag
// <meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```

### Beginner Examples

```javascript
// Basic CSP: Only allow resources from same origin
const helmet = require('helmet');
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'"],
    imgSrc: ["'self'"]
  }
}));

// Equivalent header:
// Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self'
```

### Intermediate Examples

```javascript
// Intermediate: Allowing external services
app.use(helmet.contentSecurityPolicy({
  useDefaults: true,
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "https://analytics.example.com"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", "https://images.example.com", "data:"],
    connectSrc: ["'self'", "https://api.example.com"],
    fontSrc: ["'self'", "https://fonts.gstatic.com"],
    objectSrc: ["'none'"],
    upgradeInsecureRequests: []
  }
}));
```

### Advanced Examples

```javascript
// Advanced: Strict dynamic CSP with nonce for inline scripts
const crypto = require('crypto');

function cspMiddleware(req, res, next) {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.nonce = nonce;
  
  res.setHeader(
    'Content-Security-Policy',
    [
      `default-src 'self'`,
      `script-src 'nonce-${nonce}' 'strict-dynamic' 'unsafe-inline' https: http:`,
      `object-src 'none'`,
      `base-uri 'none'`,
      `report-uri /csp-report`
    ].join('; ')
  );
  
  next();
}

// Usage in template
// <script nonce="<%= nonce %>">console.log('safe')</script>
```

### Real-World Use Cases

- Banking and financial applications requiring strict security
- E-commerce platforms with third-party payment widgets
- Social media sites embedding user-generated content
- SaaS applications with custom domains
- Government and healthcare sites under compliance requirements

### Common Mistakes

- Using `'unsafe-inline'` for scripts (defeats CSP's XSS protection)
- Forgetting to include all necessary origins for third-party services
- Being overly restrictive and breaking legitimate functionality
- Using CSP meta tag instead of HTTP header (meta tag doesn't enforce for some directives)
- Not testing CSP in report-only mode before enforcement
- Not including reporting endpoints for violation monitoring
- Using wildcard `*` sources (too permissive)

### Best Practices

- Start with a restrictive policy and add sources as needed
- Use `'strict-dynamic'` for modern CSP that works with nonces/hashes
- Test policies in `Content-Security-Policy-Report-Only` mode first
- Use `'nonce-{random}'` or `'hash-{algorithm}-{value}'` instead of `'unsafe-inline'`
- Include `'unsafe-inline'` as a fallback for `strict-dynamic` in older browsers
- Use `report-uri` or `report-to` to monitor violations
- Review violations regularly to identify needed policy adjustments
- Use `form-action 'self'` to prevent form hijacking

### Performance Considerations

- CSP header parsing is done once per page load by the browser
- The header size adds to response overhead (typically 200-500 bytes)
- Nonce generation adds negligible server-side cost
- Violation reports generate POST requests to the reporting endpoint
- CSP evaluation does not significantly impact page load performance
- `'strict-dynamic'` can reduce header size by eliminating long source lists

### Interview Questions

**Q: What is the difference between Content-Security-Policy and Content-Security-Policy-Report-Only?**
A: `Content-Security-Policy` enforces the policy and blocks violations. `Content-Security-Policy-Report-Only` only reports violations without blocking, allowing developers to test policies before enforcement. Report-Only is used during development and policy refinement.

**Q: How does a nonce-based CSP work?**
A: The server generates a unique random value (nonce) per request and includes it in both the CSP header (`script-src 'nonce-abc123'`) and the script tag (`<script nonce="abc123">`). The browser executes only inline scripts whose nonce matches the CSP nonce. Attackers cannot guess the nonce, so injected scripts without the correct nonce are blocked.

### Coding Challenges

**Challenge 1:** Implement CSP middleware that allows Google Analytics, a CDN, and inline scripts via nonce.

```javascript
function cspMiddleware(req, res, next) {
  // Your implementation
}
```

**Solution:**

```javascript
function cspMiddleware(req, res, next) {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.nonce = nonce;
  
  res.setHeader(
    'Content-Security-Policy',
    [
      "default-src 'self'",
      `script-src 'nonce-${nonce}' 'strict-dynamic' https://www.googletagmanager.com https://www.google-analytics.com`,
      "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
      "img-src 'self' https://www.google-analytics.com https://cdn.example.com data:",
      "font-src 'self' https://fonts.gstatic.com",
      "connect-src 'self' https://www.google-analytics.com",
      "object-src 'none'",
      "base-uri 'self'",
      "report-uri /csp-violation"
    ].join('; ')
  );
  
  next();
}
```

## Directives (default-src, script-src, style-src)

### What It Is

CSP directives define the restrictions for specific resource categories. Each directive takes one or more source expressions that the browser uses to determine whether a resource should load. Source expressions can be hostnames, schemes, keywords (like `'self'`, `'none'`, `'unsafe-inline'`), nonces, or hashes. Understanding each directive is critical for building a policy that is both secure and functional.

### Why It Is Important

Directives provide granular control over resource loading. Misconfiguring a single directive can either break the application (too restrictive) or create vulnerabilities (too permissive). `default-src` serves as the fallback for all other directives that are not explicitly set, making it the foundation of the policy. `script-src` is the most critical directive for XSS prevention. `style-src` controls CSS injection risks. Each directive addresses specific attack vectors.

### How It Works Internally

When a resource load is triggered (e.g., `<script src="...">`), the browser checks the directive corresponding to the resource type (`script-src` for scripts, `style-src` for stylesheets, `img-src` for images, etc.). If the directive is explicitly defined, the browser uses its source list. If not defined, the browser falls back to `default-src`. If `default-src` is also not defined, the browser allows the resource (no restriction). The source list can include:

- `'self'`: Same origin as the document
- `'none'`: Block all sources
- `'unsafe-inline'`: Allow inline content (scripts or styles)
- `'unsafe-eval'`: Allow `eval()` and similar functions
- `'strict-dynamic'`: Propagate trust from nonced/hashed scripts to dynamically loaded scripts
- `host-src`: `https://example.com`, `*.example.com`
- `scheme-src`: `https:`, `data:`
- `'nonce-{value}'`: Allow specific inline content by nonce
- `'hash-{algorithm}-{value}'`: Allow specific inline content by hash

### Syntax

```http
Content-Security-Policy: 
  default-src 'none';
  script-src 'self' 'nonce-random123' 'strict-dynamic';
  style-src 'self' 'unsafe-inline';
  img-src 'self' https: data:;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://api.example.com wss://ws.example.com;
  media-src 'none';
  object-src 'none';
  frame-src 'self' https://www.youtube.com;
  frame-ancestors 'none';
  form-action 'self';
  base-uri 'self';
  manifest-src 'self';
  worker-src 'self';
```

### Beginner Examples

```javascript
// Beginner: Common directives setup
const cspDirectives = {
  // Fallback for all unspecified directives
  defaultSrc: ["'self'"],
  
  // Scripts: same origin only
  scriptSrc: ["'self'"],
  
  // Styles: same origin + inline styles 
  styleSrc: ["'self'", "'unsafe-inline'"],
  
  // Images: same origin + data URIs
  imgSrc: ["'self'", "data:"],
  
  // Fonts: same origin only
  fontSrc: ["'self'"],
  
  // AJAX/WebSocket connections
  connectSrc: ["'self'"],
  
  // Forms: same origin only
  formAction: ["'self'"],
  
  // Frames: none (clickjacking protection)
  frameAncestors: ["'none'"],
  
  // Objects: none (block Flash, Java)
  objectSrc: ["'none'"],
  
  // Base URL: same origin (prevents base URI injection)
  baseUri: ["'self'"]
};
```

### Intermediate Examples

```javascript
// Intermediate: Handling third-party integrations
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    
    // Google Analytics needs:
    scriptSrc: [
      "'self'",
      "https://www.googletagmanager.com",
      "https://www.google-analytics.com",
      "'unsafe-inline'"  // Fallback for GA inline scripts
    ],
    
    // GA uses pixel tracking
    imgSrc: [
      "'self'",
      "https://www.google-analytics.com",
      "https://stats.g.doubleclick.net",
      "data:"
    ],
    
    // GA needs to connect
    connectSrc: [
      "'self'",
      "https://www.google-analytics.com",
      "https://analytics.google.com"
    ],
    
    // Stripe.js for payments
    scriptSrc: [
      "'self'",
      "https://js.stripe.com"
    ],
    
    frameSrc: [
      "'self'",
      "https://js.stripe.com"
    ],
    
    // Allowed font origins
    fontSrc: [
      "'self'",
      "https://fonts.gstatic.com"
    ],
    
    // Block everything not explicitly needed
    objectSrc: ["'none'"],
    mediaSrc: ["'none'"],
    workerSrc: ["'none'"]
  }
}));
```

### Advanced Examples

```javascript
// Advanced: Dynamic CSP with strict-dynamic and fallbacks
const crypto = require('crypto');

function strictCSP(req, res, next) {
  const nonce = crypto.randomBytes(16).toString('base64');
  const hash = crypto.createHash('sha256')
    .update('console.log("app init")')
    .digest('base64');
  
  res.setHeader(
    'Content-Security-Policy',
    [
      "default-src 'self'",
      
      // Modern: nonce + strict-dynamic
      // strict-dynamic propagates trust to dynamically loaded scripts
      // 'unsafe-inline' as fallback for browsers without strict-dynamic
      `script-src 'nonce-${nonce}' 'strict-dynamic' 'unsafe-inline' https: http:`,
      
      // Hash for specific inline script
      `script-src 'sha256-${hash}'`,
      
      // Styles: nonce or hash preferred over unsafe-inline
      `style-src 'self' 'unsafe-inline'`,
      
      // Only allow specific CDN for images
      "img-src 'self' https://cdn.example.com data: blob:",
      
      // Restrict XMLHttpRequest, fetch, WebSocket
      "connect-src 'self' https://api.example.com",
      
      // Restrict forms
      "form-action 'self'",
      
      // Prevent framing of this page
      "frame-ancestors 'none'",
      
      // Block plugins
      "object-src 'none'",
      "embed-src 'none'",
      
      // Upgrade HTTP to HTTPS
      "upgrade-insecure-requests"
    ].join('; ')
  );
  
  res.locals.nonce = nonce;
  next();
}

// Advanced: CSP per route
app.get('/admin/*', (req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self'; frame-ancestors 'none'"
  );
  next();
});

app.get('/embed/*', (req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; frame-ancestors *"
  );
  next();
});
```

### Real-World Use Cases

- E-commerce: Stripe, PayPal, Google Analytics, Facebook Pixel
- SaaS: Intercom, Mixpanel, Sentry, Hotjar
- Media: YouTube embeds, Vimeo, SoundCloud
- Developer tools: Google Fonts, Font Awesome, CDN assets
- Authentication: Auth0, Firebase, Google Sign-In

### Common Mistakes

- Using `'unsafe-inline'` for scripts instead of nonces or hashes
- Not setting `object-src 'none'` (plugin-based XSS vector)
- Forgetting `base-uri 'self'` (base URI injection bypasses CSP)
- Using `https:` as a source (allows any HTTPS origin)
- Not accounting for `form-action` (allows form submission to any origin)
- Setting `default-src` too permissively (undermines all other directives)
- Not including `blob:` or `data:` when needed (can break PDF viewers, charts, etc.)

### Best Practices

- Use `default-src 'none'` as the strictest base, then explicitly allow what's needed
- Prefer nonce-based or hash-based script policies over `'unsafe-inline'`
- Use `'strict-dynamic'` for modern browsers to simplify script source management
- Set `object-src 'none'` and `frame-ancestors 'none'` as baseline security
- Use `base-uri 'self'` to prevent base element injection
- Use `form-action 'self'` to restrict form submission targets
- Use `upgrade-insecure-requests` to enforce HTTPS
- Test each directive in report-only mode before enforcement

### Performance Considerations

- More specific directives reduce the browser's matching effort
- `'strict-dynamic'` can reduce header size by eliminating long source lists
- Hash-based policies require computing hashes during build time
- The header size increases with each source expression; consider bundling
- Nonce generation uses `crypto.randomBytes`, which is fast but not free
- CSP evaluation is synchronous in the browser's resource loading pipeline

### Interview Questions

**Q: What is the difference between script-src and default-src?**
A: `script-src` specifically controls JavaScript sources. `default-src` serves as the fallback for any directive not explicitly set. If only `default-src` is set without `script-src`, then `default-src`'s value applies to scripts. Both can be set simultaneously, with `script-src` taking precedence over `default-src` for script resources.

**Q: Why is `'unsafe-inline'` dangerous in script-src?**
A: `'unsafe-inline'` allows all inline scripts to execute, including attacker-injected ones. This effectively nullifies CSP's XSS protection because any injected `<script>` tag or event handler will execute. Using nonces or hashes instead ensures that only specific, pre-approved inline scripts can execute.

### Coding Challenges

**Challenge 1:** Create a CSP configuration that allows Font Awesome CDN, Google Fonts, and a custom analytics endpoint while blocking everything else.

```javascript
function createCSP() {
  // Your implementation
}
```

**Solution:**

```javascript
function createCSP() {
  return {
    defaultSrc: ["'none'"],
    scriptSrc: ["'self'", "https://use.fontawesome.com", "https://kit.fontawesome.com"],
    styleSrc: ["'self'", "https://use.fontawesome.com", "https://fonts.googleapis.com", "'unsafe-inline'"],
    fontSrc: ["'self'", "https://fonts.gstatic.com", "https://use.fontawesome.com"],
    imgSrc: ["'self'", "data:"],
    connectSrc: ["'self'", "https://analytics.example.com"],
    formAction: ["'self'"],
    frameAncestors: ["'none'"],
    objectSrc: ["'none'"],
    baseUri: ["'self'"]
  };
}
```

## Report-Uri and Reporting

### What It Is

CSP violation reporting allows developers to receive notifications when the browser blocks resources due to policy violations. The `report-uri` directive (deprecated but still supported) specifies an endpoint where violation reports are sent via POST. The newer `report-to` directive uses the Reporting API to send reports to a group endpoint. Reports contain information about the violated directive, the blocked resource, and the page where the violation occurred.

### Why It Is Important

Violation reports are essential for CSP maintenance. They reveal resources that are being blocked, allowing developers to adjust the policy or fix issues. Without reporting, developers are blind to CSP breakage. Reports also serve as intrusion detection CSRF mechanisms, alerting administrators to potential XSS or injection attempts when unexpected violations occur.

### How It Works Internally

When the browser blocks a resource due to CSP, it creates a JSON report object and sends it to the configured endpoint. The report contains: `document-uri` (the page where the violation occurred), `referrer`, `blocked-uri` (the resource that was blocked), `violated-directive` (the directive that was violated), `effective-directive`, `original-policy`, `disposition` (enforce or report), `script-sample` (first 40 characters of inline script that was blocked), `status-code`, and `source-file`.

The browser uses HTTP POST to send reports, with `Content-Type: application/csp-report` (old format) or `application/reports+json` (Reporting API format). Browsers batch reports and send them periodically, typically every few seconds up to a maximum of 5-10 reports.

### Syntax

```http
# Deprecated report-uri
Content-Security-Policy: default-src 'self'; report-uri /csp-reports

# Modern report-to (requires Report-To header)
Report-To: {"group":"default","max_age":10886400,"endpoints":[{"url":"https://report.example.com/csp"}]}
Content-Security-Policy: default-src 'self'; report-to default
```

```javascript
// Server-side CSP report endpoint
app.post('/csp-report', (req, res) => {
  const report = req.body;
  
  // Handle both old and new report formats
  const violation = report['csp-report'] || report;
  
  console.warn('CSP Violation:', {
    documentUri: violation['document-uri'] || violation.documentURI,
    blockedUri: violation['blocked-uri'] || violation.blockedURI,
    violatedDirective: violation['violated-directive'] || violation.violatedDirective,
    originalPolicy: violation['original-policy'] || violation.originalPolicy,
    userAgent: req.headers['user-agent'],
    ip: req.ip,
    timestamp: new Date().toISOString()
  });
  
  // Store for analysis
  // saveViolation(violation);
  
  res.status(204).end();
});
```

### Beginner Examples

```javascript
// Beginner: Simple CSP reporting middleware
const helmet = require('helmet');

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    reportUri: '/csp-violation'
  },
  reportOnly: false // Set to true for testing
}));

// Simple report receiver
app.post('/csp-violation', express.json({ type: ['application/json', 'application/csp-report'] }), (req, res) => {
  console.log('CSP Report:', JSON.stringify(req.body, null, 2));
  res.status(204).end();
});
```

### Intermediate Examples

```javascript
// Intermediate: Comprehensive CSP reporting with analysis
const rateLimit = require('express-rate-limit');

class CSPReportService {
  constructor() {
    this.violations = [];
    this.alertThreshold = 100; // Alert after 100 similar violations
  }
  
  async handleReport(report) {
    const violation = this.normalizeReport(report);
    this.violations.push(violation);
    
    // Pattern analysis
    const key = `${violation.violatedDirective}:${violation.blockedUri}`;
    this.patternCounts = this.patternCounts || {};
    this.patternCounts[key] = (this.patternCounts[key] || 0) + 1;
    
    // Alert on unusual patterns
    if (this.patternCounts[key] === this.alertThreshold) {
      await this.sendAlert({
        type: 'csp_pattern_alert',
        directive: violation.violatedDirective,
        blockedUri: violation.blockedUri,
        count: this.patternCounts[key]
      });
    }
    
    return violation;
  }
  
  normalizeReport(report) {
    const cspReport = report['csp-report'] || report;
    return {
      documentUri: cspReport['document-uri'] || cspReport.documentURI,
      blockedUri: cspReport['blocked-uri'] || cspReport.blockedURI,
      violatedDirective: cspReport['violated-directive'] || cspReport.violatedDirective,
      effectiveDirective: cspReport['effective-directive'] || cspReport.effectiveDirective,
      originalPolicy: cspReport['original-policy'] || cspReport.originalPolicy,
      disposition: cspReport['disposition'] || cspReport.disposition,
      scriptSample: cspReport['script-sample'] || cspReport.scriptSample,
      sourceFile: cspReport['source-file'] || cspReport.sourceFile,
      lineNumber: cspReport['line-number'] || cspReport.lineNumber,
      columnNumber: cspReport['column-number'] || cspReport.columnNumber,
      timestamp: new Date().toISOString()
    };
  }
  
  async sendAlert(data) {
    // Send to alerting system (PagerDuty, Slack, etc.)
    console.warn('CSP Alert:', data);
  }
  
  getStatistics() {
    return {
      totalViolations: this.violations.length,
      patternCounts: this.patternCounts,
      recentViolations: this.violations.slice(-10)
    };
  }
}

const cspService = new CSPReportService();

// Rate-limited report endpoint
const reportLimiter = rateLimit({
  windowMs: 60000,
  max: 100,
  message: 'Too many reports'
});

app.post('/csp/report', reportLimiter, express.json({
  type: ['application/json', 'application/csp-report', 'application/reports+json']
}), (req, res) => {
  const report = req.body;
  
  if (Array.isArray(report)) {
    // Reporting API sends arrays
    report.forEach(r => cspService.handleReport(r));
  } else {
    cspService.handleReport(report);
  }
  
  res.status(204).end();
});

// Dashboard endpoint for viewing violations
app.get('/csp/stats', (req, res) => {
  res.json(cspService.getStatistics());
});
```

### Advanced Examples

```javascript
// Advanced: CSP reporting with Reporting API and deduplication
const crypto = require('crypto');

class CSPReportCollector {
  constructor(options = {}) {
    this.maxReports = options.maxReports || 10000;
    this.batchInterval = options.batchInterval || 30000;
    this.reports = new Map();
    this.endpoints = options.endpoints || [];
    this.setupBatchProcessing();
  }
  
  setupBatchProcessing() {
    setInterval(() => {
      this.flushReports();
    }, this.batchInterval);
  }
  
  ingestReport(report) {
    const normalized = this.normalize(report);
    
    // Deduplicate by creating a hash of key fields
    const hash = crypto.createHash('md5')
      .update(`${normalized.documentUri}:${normalized.blockedUri}:${normalized.violatedDirective}`)
      .digest('hex');
    
    if (this.reports.has(hash)) {
      const existing = this.reports.get(hash);
      existing.count++;
      existing.lastSeen = new Date().toISOString();
    } else {
      this.reports.set(hash, {
        ...normalized,
        count: 1,
        firstSeen: new Date().toISOString(),
        lastSeen: new Date().toISOString()
      });
      
      // Prune if too many
      if (this.reports.size > this.maxReports) {
        const oldest = Array.from(this.reports.entries())
          .sort((a, b) => new Date(a[1].firstSeen) - new Date(b[1].firstSeen))[0];
        this.reports.delete(oldest[0]);
      }
    }
  }
  
  normalize(report) {
    const csp = report['csp-report'] || report;
    return {
      documentUri: csp['document-uri'] || csp.documentURI,
      referrer: csp['referrer'],
      blockedUri: csp['blocked-uri'] || csp.blockedURI,
      violatedDirective: csp['violated-directive'] || csp.violatedDirective,
      effectiveDirective: csp['effective-directive'] || csp.effectiveDirective,
      originalPolicy: csp['original-policy'] || csp.originalPolicy,
      disposition: csp['disposition'] || csp.disposition,
      scriptSample: csp['script-sample'] || csp.scriptSample,
      lineNumber: csp['line-number'] || csp.lineNumber,
      sourceFile: csp['source-file'] || csp.sourceFile
    };
  }
  
  async flushReports() {
    if (this.reports.size === 0) return;
    
    const batch = Array.from(this.reports.values());
    this.reports.clear();
    
    // Forward to external reporting services
    await Promise.allSettled(
      this.endpoints.map(endpoint =>
        fetch(endpoint, {
          method: 'POST',
          headers: { 'Content-Type': 'application/reports+json' },
          body: JSON.stringify(batch)
        }).catch(err => console.error('Failed to forward CSP report:', err))
      )
    );
    
    console.log(`Flushed ${batch.length} aggregated CSP reports`);
  }
}

// Usage
const collector = new CSPReportCollector({
  maxReports: 5000,
  batchInterval: 60000,
  endpoints: ['https://csp-reports.example.com/collect']
});

app.post('/csp/report', (req, res) => {
  const reports = Array.isArray(req.body) ? req.body : [req.body];
  reports.forEach(r => collector.ingestReport(r));
  res.status(204).end();
});
```

### Real-World Use Cases

- Production CSP monitoring dashboards (Sentry, Report URI)
- Security operations center (SOC) alerting on CSP violations
- CI/CD pipeline policy validation (reject builds with unexpected violations)
- Attack detection (sudden spike in violations indicates XSS probing)
- Third-party script inventory management
- Regulatory compliance reporting

### Common Mistakes

- Not implementing a report endpoint and being blind to violations
- Using `report-uri` without also using `report-to` for modern browsers
- Not rate-limiting the report endpoint (can be abused for DDoS)
- Ignoring CSP reports in production (every report indicates a potential issue)
- Not filtering or deduplicating reports (can be overwhelming)
- Using `report-uri` with a relative path that resolves incorrectly

### Best Practices

- Always include a `report-uri` or `report-to` directive
- Use both `report-uri` (legacy) and `report-to` (modern) for maximum compatibility
- Implement rate limiting on the report endpoint
- Set up alerting for unusual CSP violation patterns
- Review CSP reports regularly and update the policy accordingly
- Use report-only mode for initial policy deployment
- Deduplicate and aggregate reports to reduce noise
- Log CSP violations in a structured format for analysis

### Performance Considerations

- Report generation is asynchronous in the browser, no impact on page rendering
- Reports are sent asynchronously after blocking, no impact on user experience
- Rate limiting prevents report endpoint overload
- Deduplication reduces storage and processing requirements
- Network cost per report is minimal (small JSON payload)
- Aggregation reduces the number of POST requests

### Interview Questions

**Q: What information is included in a CSP violation report?**
A: The report includes: `document-uri` (the page where the violation occurred), `blocked-uri` (the resource that was blocked), `violated-directive` (the CSP directive that was violated), `effective-directive` (the directive that was in effect, after fallbacks), `original-policy` (the full CSP string), `disposition` (whether it was enforced or report-only), `script-sample` (for inline script violations, the first 40 characters), `line-number` and `source-file` (where the violation originated), and `referrer`.

**Q: Why should you use both report-uri and report-to?**
A: `report-uri` is the older CSP spec and is supported in all CSP-supporting browsers. `report-to` is part of the newer Reporting API and allows multiple report types (CSP, network errors, deprecations) to use the same endpoint. Using both ensures forward compatibility: browsers that support `report-to` ignore `report-uri`; older browsers use `report-uri`. This provides the widest coverage.

### Coding Challenges

**Challenge 1:** Implement a CSP report aggregator that deduplicates reports and provides a summary endpoint.

```javascript
class CSPAggregator {
  constructor() {
    // Your implementation
  }
  
  addReport(report) { }
  getSummary() { }
}
```

**Solution:**

```javascript
class CSPAggregator {
  constructor() {
    this.reports = new Map();
    this.directiveCount = new Map();
  }
  
  addReport(report) {
    const csp = report['csp-report'] || report;
    const key = `${csp['violated-directive']}:${csp['blocked-uri']}`;
    
    if (this.reports.has(key)) {
      this.reports.get(key).count++;
    } else {
      this.reports.set(key, {
        violatedDirective: csp['violated-directive'],
        blockedUri: csp['blocked-uri'],
        documentUri: csp['document-uri'],
        count: 1,
        firstSeen: new Date().toISOString()
      });
    }
    
    const dir = csp['violated-directive'] || 'unknown';
    this.directiveCount.set(dir, (this.directiveCount.get(dir) || 0) + 1);
  }
  
  getSummary() {
    const topViolations = Array.from(this.reports.values())
      .sort((a, b) => b.count - a.count)
      .slice(0, 20);
    
    const directiveBreakdown = Object.fromEntries(this.directiveCount);
    
    return {
      totalUniqueViolations: this.reports.size,
      totalIncidents: Array.from(this.reports.values()).reduce((s, r) => s + r.count, 0),
      topViolations,
      directiveBreakdown
    };
  }
}
```

## Nonce and Hash Sources

### What It Is

Nonce and hash sources are CSP mechanisms that allow specific inline scripts or styles to execute without using the permissive `'unsafe-inline'` keyword. A nonce is a random value generated per request, included in both the CSP header and the HTML tag. A hash is the cryptographic digest of the script content, specified in the CSP header. Both provide cryptographically-enforced allowlisting of inline content.

### Why It Is Important

Nonce and hash sources are the recommended alternatives to `'unsafe-inline'` for modern CSP. They enable inline scripts (necessary for many frameworks and analytics) while maintaining strong XSS protection. An attacker who injects arbitrary `<script>` tags cannot know the correct nonce or match the required hash, so injected scripts are blocked. `'strict-dynamic'` combined with nonces provides the most secure and practical CSP for modern web applications.

### How It Works Internally

**Nonce**: The server generates a cryptographically random value per HTTP response. This value is included in both the CSP header (`script-src 'nonce-abc123'`) and the `nonce` attribute of trusted `<script>` tags (`<script nonce="abc123">`). The browser matches the nonce in the header with the nonce on each script tag. Only scripts with matching nonces execute. Nonces must be unique per request and unpredictable.

**Hash**: The developer computes the hash (SHA-256, SHA-384, or SHA-512) of the inline script content and includes it in the CSP header (`script-src 'sha256-base64hash'`). When the browser encounters a `<script>` tag, it computes the hash of its content and compares it to the allowed hashes in the header. Hashes are computed at build time and remain valid as long as the script content doesn't change.

### Syntax

```http
# Nonce-based CSP
Content-Security-Policy: script-src 'nonce-random123' 'strict-dynamic' 'unsafe-inline' https: http:

# Hash-based CSP
Content-Security-Policy: script-src 'sha256-ABC123...' 'sha384-DEF456...'
```

```javascript
// Server generates nonce
const nonce = crypto.randomBytes(16).toString('base64');

// HTML template uses nonce
const html = `<script nonce="${nonce}">
  initializeApp();
</script>
<script nonce="${nonce}" src="app.js"></script>`;
```

### Beginner Examples

```javascript
// Beginner: Nonce implementation in Express
const crypto = require('crypto');
const express = require('express');
const app = express();

app.use((req, res, next) => {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.nonce = nonce;
  
  res.setHeader(
    'Content-Security-Policy',
    `script-src 'nonce-${nonce}' 'strict-dynamic' 'unsafe-inline' https: http:`
  );
  
  next();
});

// In template (EJS):
// <script nonce="<%= nonce %>">
//   console.log('This inline script is allowed');
// </script>
```

### Intermediate Examples

```javascript
// Intermediate: Hash-based CSP for static inline scripts
const crypto = require('crypto');

// Build-time script hashing
function hashScript(content, algorithm = 'sha256') {
  const hash = crypto.createHash(algorithm).update(content).digest('base64');
  return `'${algorithm}-${hash}'`;
}

const inlineScript = 'console.log("app initialized");';
const scriptHash = hashScript(inlineScript);
// Result: 'sha256-uK7F6m1Y++XYZ...'

// CSP header
app.use(helmet.contentSecurityPolicy({
  directives: {
    scriptSrc: [
      "'self'",
      `'sha256-${crypto.createHash('sha256').update(inlineScript).digest('base64')}'`
    ]
  }
}));

// In HTML:
// <script>console.log("app initialized");</script>
// The browser hashes this and matches against CSP
```

### Advanced Examples

```javascript
// Advanced: Combined nonce + hash + strict-dynamic for comprehensive coverage
const crypto = require('crypto');

function strictCSPMiddleware(req, res, next) {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.nonce = nonce;
  
  // Pre-computed hashes for known inline scripts
  const scriptHashes = [
    'sha256-47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=', // empty script
    'sha256-2mBQhsfQ1mOe8M5XKQVrGoXRiR/kzR5AhT3Ko/JvOhM=', // specific GA snippet
  ].join(' ');
  
  res.setHeader(
    'Content-Security-Policy',
    [
      "default-src 'self'",
      
      // Primary: nonce-based with strict-dynamic
      `script-src 'nonce-${nonce}' 'strict-dynamic' ${scriptHashes}`,
      
      // Fallback for older browsers (ignored with strict-dynamic in modern browsers)
      "'unsafe-inline' https: http:",
      
      // Additional directives
      "object-src 'none'",
      "base-uri 'self'",
      "report-uri /csp-report"
    ].join('; ')
  );
  
  next();
}

// Advanced: Automatic nonce injection for inline scripts via HTML rewriting
class NonceInjector {
  constructor(nonce) {
    this.nonce = nonce;
    this.scriptRegex = /<script(?![^>]*nonce)([^>]*)>/gi;
  }
  
  processHtml(html) {
    return html.replace(this.scriptRegex, (match, attrs) => {
      // Skip if already has nonce
      if (attrs.includes('nonce=')) return match;
      
      // Add nonce to inline scripts and external scripts
      return `<script${attrs} nonce="${this.nonce}">`;
    });
  }
}

// Usage in middleware
app.use((req, res, next) => {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.nonce = nonce;
  res.locals.nonceInjector = new NonceInjector(nonce);
  
  res.setHeader(
    'Content-Security-Policy',
    `default-src 'self'; script-src 'nonce-${nonce}' 'strict-dynamic'`
  );
  
  next();
});

// Override res.send to inject nonces
const originalSend = res.send.bind(res);
res.send = function(body) {
  if (typeof body === 'string' && body.includes('<script')) {
    body = res.locals.nonceInjector.processHtml(body);
  }
  return originalSend(body);
};
```

### Real-World Use Cases

- Google Analytics (uses inline scripts that need nonces or hashes)
- Single-page applications with dynamic script loading
- A/B testing tools that inject inline scripts
- Error monitoring services (Sentry, Rollbar) that use inline configuration
- Custom analytics and tracking scripts
- Micro-frontends with dynamic script injection

### Common Mistakes

- Using the same nonce for multiple requests (nonces must be unique per response)
- Hardcoding nonces in source code (defeats the purpose)
- Including `'unsafe-inline'` alongside nonces (modern browsers ignore nonce if unsafe-inline is present)
- Not updating hashes when script content changes
- Using hashes for dynamic inline scripts (hashes work only for fixed content)
- Forgetting to include `'strict-dynamic'` for proper dynamic script propagation

### Best Practices

- Generate nonces using cryptographically secure random values (`crypto.randomBytes`)
- Use `'strict-dynamic'` to propagate trust to dynamically loaded scripts
- For static inline scripts, prefer hashes over nonces (no server-side generation needed)
- For dynamic inline scripts, use nonces (hashes can't account for dynamic content)
- Never use the same nonce for different script tags on the same page (they can be copied)
- Regenerate nonces on every HTTP response
- Include `'unsafe-inline'` as a fallback for older browsers when using `'strict-dynamic'`
- Combine hashes and nonces for comprehensive coverage

### Performance Considerations

- Nonce generation adds minimal overhead (16 bytes of random data per request)
- Hash computation at build time has zero runtime cost
- Hash computation at runtime (client-side) is negligible for small scripts
- `'strict-dynamic'` reduces CSP header size by eliminating source lists
- Nonce injection via HTML rewriting adds string processing overhead
- Both mechanisms are cached by the browser after initial policy evaluation

### Interview Questions

**Q: What's the difference between nonce-based and hash-based CSP for scripts?**
A: Nonce-based CSP uses a random value generated per request that matches between the CSP header and the script tag's nonce attribute. Hash-based CSP uses the cryptographic hash of the script content itself. Nonces work for dynamic script content; hashes are for static, known content. Nonces must be generated server-side per request; hashes can be computed at build time.

**Q: When would you choose a hash over a nonce?**
A: Hashes are preferred when the inline script content is fixed and known at build time, such as a bootstrap script or configuration object that never changes. Hashes don't require server-side generation, making them suitable for static sites or CDN-served pages where dynamic nonce generation isn't possible. Hashes also work for service workers, which can't use nonces.

### Coding Challenges

**Challenge 1:** Implement a CSP middleware that generates a nonce, applies it to scripts, and sets the appropriate policy.

```javascript
function nonceCSP(req, res, next) {
  // Your implementation
}
```

**Solution:**

```javascript
function nonceCSP(req, res, next) {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.scriptNonce = nonce;
  
  res.setHeader(
    'Content-Security-Policy',
    [
      "default-src 'self'",
      `script-src 'nonce-${nonce}' 'strict-dynamic' 'unsafe-inline' https: http:`,
      "object-src 'none'",
      "base-uri 'self'",
      "report-uri /csp-report"
    ].join('; ')
  );
  
  // Optionally inject nonce into response HTML
  const originalSend = res.send.bind(res);
  res.send = function(body) {
    if (typeof body === 'string' && body.includes('<script')) {
      body = body.replace(/<script(?![^>]*nonce)([^>]*)>/gi, 
        (match, attrs) => `<script${attrs} nonce="${nonce}">`);
    }
    return originalSend(body);
  };
  
  next();
}
```

### Related Topics

- XSS prevention (detailed cross-reference)
- CSP violation reporting
- Strict dynamic CSP
- Trusted Types API
- Subresource Integrity (SRI)
- Security headers (HSTS, X-Frame-Options, X-Content-Type-Options)
- OWASP CSP Cheat Sheet
- Mozilla CSP documentation and playground
- Google's CSP Evaluator tool
