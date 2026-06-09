# Fetch API - fetch() method, Request/Response, headers, error handling

## Introduction

The Fetch API provides a modern, promise-based interface for making HTTP requests in JavaScript. It replaces the older XMLHttpRequest with a more powerful and flexible approach, leveraging JavaScript Promises for asynchronous operations. Fetch is built into modern browsers and is also available in Node.js (since version 18). It offers a cleaner syntax, better error handling patterns, and integration with service workers and the Cache API.

## fetch() method

### What It Is

The `fetch()` method is the entry point of the Fetch API. It initiates an HTTP request and returns a Promise that resolves to a Response object. The method accepts a URL string or a Request object, and an optional options object to configure the request.

```javascript
fetch(resource, options)
```

### Why It Is Important

The `fetch()` method simplifies HTTP communication by using Promises, eliminating callback hell and making asynchronous code more readable. It handles the most common request patterns with minimal code, supports streaming, and integrates seamlessly with async/await syntax.

### How It Works Internally

When `fetch()` is called, the browser queues the request and returns a Promise immediately. The Promise resolves when the response headers are received (not when the full body is available). The response body is streamed, which means you can start processing data before it's fully downloaded. Under the hood, the browser creates a network request, manages connection pooling, handles redirects, and enforces the same-origin policy and CORS restrictions.

```javascript
// Internal flow
// 1. Create internal request object
// 2. Check cache (if cache mode set)
// 3. Apply CORS checks
// 4. Open network connection
// 5. Send request headers
// 6. Receive response headers -> Promise resolves
// 7. Stream response body -> available via body methods
```

### Syntax

```javascript
// Basic GET request
fetch('https://api.example.com/data')

// With options
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  body: JSON.stringify({ key: 'value' })
})

// Using async/await
async function getData() {
  const response = await fetch('https://api.example.com/data')
  const data = await response.json()
  return data
}
```

### Beginner Examples

```javascript
// Simple GET request with JSON response
fetch('https://jsonplaceholder.typicode.com/posts/1')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error))

// POST request with JSON body
fetch('https://jsonplaceholder.typicode.com/posts', {
  method: 'POST',
  body: JSON.stringify({
    title: 'foo',
    body: 'bar',
    userId: 1
  }),
  headers: {
    'Content-type': 'application/json; charset=UTF-8'
  }
})
  .then(response => response.json())
  .then(json => console.log(json))
```

### Intermediate Examples

```javascript
// AbortController for timeout handling
function fetchWithTimeout(url, timeoutMs = 5000) {
  const controller = new AbortController()
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs)

  return fetch(url, { signal: controller.signal })
    .then(response => {
      clearTimeout(timeoutId)
      return response
    })
    .catch(error => {
      clearTimeout(timeoutId)
      if (error.name === 'AbortError') {
        throw new Error('Request timed out')
      }
      throw error
    })
}

// Progress tracking (content-length based)
async function fetchWithProgress(url) {
  const response = await fetch(url)
  const contentLength = response.headers.get('content-length')
  const total = parseInt(contentLength, 10)
  let loaded = 0

  const reader = response.body.getReader()
  const chunks = []

  while (true) {
    const { done, value } = await reader.read()
    if (done) break
    chunks.push(value)
    loaded += value.length
    console.log(`Progress: ${((loaded / total) * 100).toFixed(1)}%`)
  }

  const blob = new Blob(chunks)
  return blob
}

// Handling different response types
async function handleResponse(url) {
  const response = await fetch(url)

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`)
  }

  const contentType = response.headers.get('content-type')

  if (contentType && contentType.includes('application/json')) {
    return response.json()
  } else if (contentType && contentType.includes('text/html')) {
    return response.text()
  } else if (contentType && contentType.includes('image')) {
    return response.blob()
  }

  return response.arrayBuffer()
}
```

### Advanced Examples

```javascript
// Streaming JSON parsing (NDJSON)
async function* streamJsonLines(url) {
  const response = await fetch(url)
  const reader = response.body.getReader()
  const decoder = new TextDecoder()
  let buffer = ''

  while (true) {
    const { done, value } = await reader.read()
    if (done) break

    buffer += decoder.decode(value, { stream: true })
    const lines = buffer.split('\n')
    buffer = lines.pop() || ''

    for (const line of lines) {
      if (line.trim()) {
        yield JSON.parse(line)
      }
    }
  }
}

// Retry with exponential backoff
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options)
      if (response.status >= 500 && attempt < maxRetries) {
        throw new Error(`Server error ${response.status}`)
      }
      return response
    } catch (error) {
      if (attempt === maxRetries) throw error
      const delay = Math.pow(2, attempt) * 1000
      await new Promise(resolve => setTimeout(resolve, delay))
    }
  }
}

// Multipart form upload with progress
async function uploadFile(url, file, onProgress) {
  const formData = new FormData()
  formData.append('file', file)

  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.upload.addEventListener('progress', (event) => {
      if (event.lengthComputable) {
        onProgress(event.loaded / event.total)
      }
    })
    xhr.addEventListener('load', () => {
      resolve(xhr.response)
    })
    xhr.addEventListener('error', reject)
    xhr.open('POST', url)
    xhr.send(formData)
  })
}

// Cache-first strategy with fetch
async function cacheFirstFetch(url) {
  const cache = await caches.open('api-cache-v1')
  const cachedResponse = await cache.match(url)

  if (cachedResponse) {
    return cachedResponse
  }

  const networkResponse = await fetch(url)
  cache.put(url, networkResponse.clone())
  return networkResponse
}
```

### Real-World Use Cases

- **API client libraries**: Fetch is the foundation for HTTP clients in modern JavaScript applications
- **Server-side rendering**: Next.js and other frameworks use fetch for data fetching during SSR
- **Progressive enhancement**: Service workers use fetch event listeners for caching strategies
- **File uploads**: Multipart form data uploads with progress tracking
- **Real-time dashboards**: Polling endpoints with controlled concurrency
- **Authentication flows**: Token refresh and retry logic for 401 responses

### Common Mistakes

```javascript
// Mistake: Not checking response.ok
fetch('/api/data')
  .then(response => response.json()) // 404 response also goes here!
  .catch(error => console.log('Not caught for 404'))

// Correct: Always check response.ok
fetch('/api/data')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`)
    }
    return response.json()
  })

// Mistake: Cannot read body multiple times
const response = await fetch('/api/data')
const data1 = await response.json()
const data2 = await response.json() // TypeError: Already read

// Correct: Clone if needed
const response = await fetch('/api/data')
const clone = response.clone()
const data1 = await response.json()
const data2 = await clone.json()

// Mistake: Forgetting to handle network errors separately
// fetch only rejects on network failure, not HTTP errors
```

### Best Practices

```javascript
// Always validate response status
async function safeFetch(url, options = {}) {
  const response = await fetch(url, options)
  if (!response.ok) {
    const errorBody = await response.text().catch(() => '')
    throw new ApiError(response.status, response.statusText, errorBody)
  }
  return response
}

// Use AbortController for cleanup
function createCancellableFetch() {
  const controller = new AbortController()
  const fetchPromise = fetch(url, { signal: controller.signal })
  return { promise: fetchPromise, cancel: () => controller.abort() }
}

// Set default options with a wrapper
const api = {
  baseUrl: 'https://api.example.com/v1',
  token: null,

  async request(endpoint, options = {}) {
    const url = `${this.baseUrl}${endpoint}`
    const headers = {
      'Content-Type': 'application/json',
      ...(this.token && { Authorization: `Bearer ${this.token}` }),
      ...options.headers
    }

    const response = await fetch(url, { ...options, headers })
    if (!response.ok) throw new ApiError(response)
    return response.json()
  }
}
```

### Performance Considerations

- Response body methods (json(), text(), blob()) consume memory proportional to payload size
- For large payloads, use streaming via `response.body.getReader()` to avoid buffering the entire response
- Connection pooling is automatic; limit concurrent requests to avoid saturating connections
- Use `keepalive: true` for analytics pings to ensure requests complete even after page unload
- Caching responses via Cache API reduces network latency
- Prefer `GET` over `POST` for idempotent requests to leverage browser caching
- Consider using HTTP/2 multiplexing for multiple concurrent requests to the same origin

### Interview Questions

**Q: Why does `fetch()` not reject on HTTP errors like 404 or 500?**

A: `fetch()` only rejects on network failures (DNS lookup failure, connection refused, etc.). HTTP error status codes (4xx, 5xx) are considered successful HTTP transactions. The response's `ok` property (boolean) and `status` property distinguish successful from error responses. This design gives developers control over how to handle different HTTP status codes.

**Q: How does `fetch()` differ from `XMLHttpRequest` in terms of request lifecycle?**

A: `fetch()` uses Promises and has a simpler lifecycle: create request -> receive response headers (Promise resolves) -> read body. XHR has multiple readyState events (0-4) with more granular control. `fetch()` cannot track upload progress natively and has no built-in timeout mechanism, requiring AbortController for both.

**Q: Can you read a `fetch()` response body multiple times?**

A: No. The response body is a ReadableStream and can only be consumed once. Calling `response.json()` or `response.text()` multiple times throws a TypeError. Use `response.clone()` to create copies for multiple reads, or cache the parsed result in a variable.

### Coding Challenges

```javascript
// Challenge 1: Implement a fetch function that automatically retries on failure
async function retryFetch(url, options, retries = 3, delay = 1000) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url, options)
      if (response.ok) return response
    } catch (e) {
      if (i === retries - 1) throw e
      await new Promise(r => setTimeout(r, delay * Math.pow(2, i)))
    }
  }
}

// Challenge 2: Fetch with concurrency limit
async function fetchAllWithLimit(urls, limit = 3) {
  const results = []
  const executing = new Set()

  for (const url of urls) {
    const promise = fetch(url).then(r => r.json())
    results.push(promise)
    executing.add(promise)
    promise.finally(() => executing.delete(promise))

    if (executing.size >= limit) {
      await Promise.race(executing)
    }
  }

  return Promise.all(results)
}

// Challenge 3: Implement fetch with request deduplication
const pendingRequests = new Map()
function deduplicatedFetch(url, options) {
  const key = `${url}-${JSON.stringify(options)}`

  if (pendingRequests.has(key)) {
    return pendingRequests.get(key)
  }

  const promise = fetch(url, options).finally(() => {
    pendingRequests.delete(key)
  })

  pendingRequests.set(key, promise)
  return promise
}
```

### Related Topics

- `XMLHttpRequest` - The legacy predecessor for HTTP requests
- `Service Workers` - Intercept fetch events for offline caching
- `Streams API` - Low-level stream handling for request/response bodies
- `AbortController` - Cancel requests and other async operations
- `Cache API` - Store and retrieve request/response pairs
- `CORS` - Cross-Origin Resource Sharing restrictions and headers
- `Web Workers` - Offload fetch processing to background threads
- `HTTP/2 Server Push` - Server-initiated requests (largely superseded by other techniques)
