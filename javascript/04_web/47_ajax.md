# AJAX - XMLHttpRequest, readyState, onreadystatechange, FormData

## Introduction

AJAX (Asynchronous JavaScript and XML) is a technique for creating asynchronous web applications. Despite its name, modern AJAX typically uses JSON rather than XML. The core API for AJAX is the XMLHttpRequest object, which enables JavaScript to make HTTP requests to the server without reloading the page. While the Fetch API has largely superseded XHR for new development, understanding XHR remains critical for maintaining legacy code, tracking upload progress, and working in environments where Fetch is unavailable.

## XMLHttpRequest object

### What It Is

XMLHttpRequest (XHR) is a browser API that provides client functionality for transferring data between a client and a server. It allows JavaScript to make HTTP and HTTPS requests and handle the server's response asynchronously through event-driven callbacks.

```javascript
const xhr = new XMLHttpRequest()
```

### Why It Is Important

XHR is the historical foundation of AJAX and remains widely used in production applications. It provides capabilities not available in the Fetch API, particularly upload progress tracking. Understanding XHR is essential for maintaining legacy codebases, working with older browsers, and scenarios requiring fine-grained control over the HTTP request lifecycle.

### How It Works Internally

When an XHR instance is created, it exists in the UNSENT state (readyState 0). Calling `open()` transitions to OPENED (1). `send()` starts the asynchronous operation, transitioning through HEADERS_RECEIVED (2), LOADING (3), and finally DONE (4). Each state transition triggers the `onreadystatechange` event. The browser handles the actual HTTP connection, manages the TCP socket, parses response headers, and streams the response body.

```javascript
// Internal lifecycle:
// UNSENT (0) -> open() -> OPENED (1) -> send() -> HEADERS_RECEIVED (2)
// -> LOADING (3) -> DONE (4)
```

### Syntax

```javascript
const xhr = new XMLHttpRequest()
xhr.open(method, url, async, user, password)
xhr.setRequestHeader(header, value)
xhr.send(body)
xhr.abort()

// Response properties
xhr.responseText     // Raw response as string
xhr.responseXML      // Response parsed as XML document
xhr.response         // Response based on responseType
xhr.status           // HTTP status code (e.g., 200)
xhr.statusText       // HTTP status text (e.g., "OK")
xhr.readyState       // Current state (0-4)
```

### Beginner Examples

```javascript
// Simple GET request
const xhr = new XMLHttpRequest()
xhr.open('GET', 'https://api.example.com/data', true)

xhr.onreadystatechange = function() {
  if (xhr.readyState === 4 && xhr.status === 200) {
    console.log(JSON.parse(xhr.responseText))
  }
}

xhr.send()

// Simple POST request with JSON
const xhr = new XMLHttpRequest()
xhr.open('POST', 'https://api.example.com/users', true)
xhr.setRequestHeader('Content-Type', 'application/json')

xhr.onload = function() {
  if (xhr.status >= 200 && xhr.status < 300) {
    console.log('Success:', xhr.responseText)
  } else {
    console.error('Error:', xhr.status, xhr.statusText)
  }
}

xhr.send(JSON.stringify({ name: 'John', email: 'john@example.com' }))
```

### Intermediate Examples

```javascript
// Upload with progress tracking
function uploadFile(url, file, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    const formData = new FormData()
    formData.append('file', file)

    xhr.open('POST', url, true)

    xhr.upload.onprogress = function(event) {
      if (event.lengthComputable) {
        const percent = Math.round((event.loaded / event.total) * 100)
        onProgress(percent)
      }
    }

    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(xhr.response)
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`))
      }
    }

    xhr.onerror = function() {
      reject(new Error('Network error'))
    }

    xhr.send(formData)
  })
}

// Download with progress tracking
function downloadFile(url, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.open('GET', url, true)
    xhr.responseType = 'blob'

    xhr.onprogress = function(event) {
      if (event.lengthComputable) {
        const percent = Math.round((event.loaded / event.total) * 100)
        onProgress(percent)
      }
    }

    xhr.onload = function() {
      if (xhr.status === 200) {
        resolve(xhr.response)
      } else {
        reject(new Error(`Download failed: ${xhr.status}`))
      }
    }

    xhr.onerror = () => reject(new Error('Network error'))
    xhr.send()
  })
}

// Request with timeout
function requestWithTimeout(url, options = {}) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.open(options.method || 'GET', url, true)

    if (options.headers) {
      Object.entries(options.headers).forEach(([key, value]) => {
        xhr.setRequestHeader(key, value)
      })
    }

    xhr.timeout = options.timeout || 10000

    xhr.ontimeout = function() {
      reject(new Error('Request timed out'))
    }

    xhr.onload = function() {
      resolve({
        status: xhr.status,
        statusText: xhr.statusText,
        data: xhr.responseText,
        headers: xhr.getAllResponseHeaders()
      })
    }

    xhr.onerror = function() {
      reject(new Error('Network error'))
    }

    xhr.send(options.body || null)
  })
}
```

### Advanced Examples

```javascript
// Promise-based XHR wrapper with full features
class XHRClient {
  constructor(baseURL = '') {
    this.baseURL = baseURL
  }

  request({ method, path, data, headers = {}, responseType = 'json', timeout = 0, onUploadProgress, onDownloadProgress }) {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest()
      const url = this.baseURL + path

      xhr.open(method, url, true)

      if (responseType) xhr.responseType = responseType
      if (timeout) xhr.timeout = timeout

      // Set default headers
      if (data && !(data instanceof FormData)) {
        headers['Content-Type'] = 'application/json'
      }

      Object.entries(headers).forEach(([key, value]) => {
        xhr.setRequestHeader(key, value)
      })

      // Upload progress
      if (onUploadProgress) {
        xhr.upload.onprogress = (e) => {
          if (e.lengthComputable) {
            onUploadProgress(e.loaded / e.total)
          }
        }
      }

      // Download progress
      if (onDownloadProgress) {
        xhr.onprogress = (e) => {
          if (e.lengthComputable) {
            onDownloadProgress(e.loaded / e.total)
          }
        }
      }

      xhr.onload = function() {
        const response = {
          status: xhr.status,
          statusText: xhr.statusText,
          headers: xhr.getAllResponseHeaders(),
          data: xhr.response
        }

        if (xhr.status >= 200 && xhr.status < 300) {
          resolve(response)
        } else {
          reject(new HttpError(response))
        }
      }

      xhr.onerror = () => reject(new Error('Network error'))
      xhr.ontimeout = () => reject(new Error('Request timeout'))

      const body = data instanceof FormData ? data : JSON.stringify(data)
      xhr.send(body)
    })
  }

  get(path, options = {}) {
    return this.request({ ...options, method: 'GET', path })
  }

  post(path, data, options = {}) {
    return this.request({ ...options, method: 'POST', path, data })
  }

  put(path, data, options = {}) {
    return this.request({ ...options, method: 'PUT', path, data })
  }

  delete(path, options = {}) {
    return this.request({ ...options, method: 'DELETE', path })
  }
}

// Queuing requests with XHR
class RequestQueue {
  constructor(concurrency = 3) {
    this.concurrency = concurrency
    this.queue = []
    this.active = 0
  }

  add(url, options = {}) {
    return new Promise((resolve, reject) => {
      this.queue.push({ url, options, resolve, reject })
      this.process()
    })
  }

  process() {
    while (this.active < this.concurrency && this.queue.length) {
      const { url, options, resolve, reject } = this.queue.shift()
      this.active++

      const xhr = new XMLHttpRequest()
      xhr.open(options.method || 'GET', url, true)

      xhr.onload = () => {
        this.active--
        resolve(xhr.responseText)
        this.process()
      }

      xhr.onerror = () => {
        this.active--
        reject(new Error(`Failed: ${url}`))
        this.process()
      }

      xhr.send(options.body || null)
    }
  }
}
```

### Real-World Use Cases

- **File upload with progress bars**: XHR remains the best choice for upload progress tracking
- **Legacy enterprise applications**: Many intranet applications still rely on XHR
- **Large file downloads**: Progress events allow showing download percentages
- **Polyfilling Fetch**: XHR is used to implement Fetch polyfills for older browsers
- **Form submissions without page reload**: Traditional AJAX form handling
- **Autocomplete/suggest**: Real-time search suggestions as user types

### Common Mistakes

```javascript
// Mistake: Forgetting to call open() before send()
const xhr = new XMLHttpRequest()
xhr.send() // Error: InvalidStateError

// Mistake: Setting headers after send()
const xhr = new XMLHttpRequest()
xhr.open('GET', '/api/data', true)
xhr.send()
xhr.setRequestHeader('Authorization', 'Bearer token') // Error

// Mistake: Not handling all error cases
xhr.onload = function() {
  // This fires for 4xx and 5xx too!
  // Always check xhr.status
}

// Mistake: Using synchronous XHR (third argument false)
// Blocks the UI thread completely - never do this
xhr.open('GET', '/api/data', false)

// Mistake: Trying to read response before DONE state
if (xhr.readyState === 2) {
  console.log(xhr.responseText) // May be empty
}
```

### Best Practices

```javascript
// Always use async XHR (true for third parameter)
const xhr = new XMLHttpRequest()
xhr.open('GET', url, true)

// Handle both onload and onerror
xhr.onload = handleSuccess
xhr.onerror = handleNetworkError
xhr.ontimeout = handleTimeout

// Always set appropriate responseType
xhr.responseType = 'json' // Automatic JSON parsing

// Set request timeout for reliability
xhr.timeout = 10000

// Abort requests on component unmount/navigation
function cleanup() {
  xhr.abort()
}

// Prefer event listeners over on* properties for multiple handlers
xhr.addEventListener('load', handler1)
xhr.addEventListener('load', handler2)
```

### Performance Considerations

- Limit concurrent XHR connections (browsers limit to 6-8 per origin)
- Use `xhr.responseType = 'blob'` for binary data to avoid string encoding overhead
- For large responses, process data incrementally during LOADING state
- Abort requests when navigating away to free up connection slots
- Consider using `withCredentials = true` sparingly as it impacts cookie handling
- XHR memory usage scales with response size; stream large responses when possible

### Interview Questions

**Q: What are the five readyState values of XMLHttpRequest?**

A: UNSENT (0) - client created but open() not called; OPENED (1) - open() called; HEADERS_RECEIVED (2) - headers received; LOADING (3) - body loading; DONE (4) - operation complete. Only DONE indicates the request is fully complete.

**Q: Why can't Fetch API track upload progress?**

A: The Fetch API's Request.body is a ReadableStream that doesn't expose progress events. The Streams API provides low-level control but lacks built-in progress reporting. XHR's upload object fires progress events natively, making it the only choice for upload progress without significant additional work.

**Q: When should you still use XHR instead of Fetch?**

A: When you need upload progress events, when working with legacy browser requirements (IE), when implementing request abort with progress tracking, and when maintaining large existing XHR-based codebases. Some enterprise environments also restrict Fetch usage in their Content Security Policy.

### Coding Challenges

```javascript
// Challenge 1: Implement a Fetch-like API using XHR
function fetchLike(url, options = {}) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.open(options.method || 'GET', url)

    if (options.headers) {
      Object.entries(options.headers).forEach(([k, v]) => xhr.setRequestHeader(k, v))
    }

    xhr.onload = () => {
      resolve({
        ok: xhr.status >= 200 && xhr.status < 300,
        status: xhr.status,
        statusText: xhr.statusText,
        headers: new Headers(xhr.getAllResponseHeaders().split('\r\n').filter(Boolean).map(h => h.split(': '))),
        json: () => Promise.resolve(JSON.parse(xhr.responseText)),
        text: () => Promise.resolve(xhr.responseText)
      })
    }

    xhr.onerror = () => reject(new TypeError('Network request failed'))
    xhr.send(options.body || null)
  })
}

// Challenge 2: Load multiple images with progress
function loadImages(urls) {
  let loaded = 0
  const total = urls.length

  return Promise.all(urls.map(url => {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest()
      xhr.responseType = 'blob'
      xhr.open('GET', url)

      xhr.onprogress = () => {
        loaded++
        console.log(`Progress: ${loaded}/${total}`)
      }

      xhr.onload = () => resolve(URL.createObjectURL(xhr.response))
      xhr.onerror = reject
      xhr.send()
    })
  }))
}

// Challenge 3: XHR with request cancellation
function createCancellableRequest(url) {
  const xhr = new XMLHttpRequest()
  const promise = new Promise((resolve, reject) => {
    xhr.open('GET', url)
    xhr.onload = () => resolve(xhr.responseText)
    xhr.onerror = reject
    xhr.send()
  })
  return { promise, cancel: () => xhr.abort() }
}
```

### Related Topics

- `Fetch API` - Modern replacement for XHR
- `FormData` - Form data serialization for XHR and Fetch
- `ProgressEvent` - Event type used by XHR progress events
- `AJAX patterns` - Common AJAX usage patterns and anti-patterns
- `Service Workers` - Can intercept XHR requests via fetch event
- `CORS` - Cross-origin constraints apply to XHR as well
- `JSON` - The primary data format for modern AJAX applications
