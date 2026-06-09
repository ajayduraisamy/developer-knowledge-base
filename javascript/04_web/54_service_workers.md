# Service Workers - ServiceWorker registration, lifecycle, events, caching strategies

## Introduction

Service Workers are JavaScript files that run in the background, separate from web pages, acting as programmable network proxies. They enable offline functionality, push notifications, background sync, and advanced caching strategies. As a core technology of Progressive Web Apps, Service Workers intercept network requests and determine how to respond, giving developers fine-grained control over the network-cache interaction. They operate on a separate thread and do not have access to the DOM.

## ServiceWorker registration

### What It Is

Registration is the process of instructing the browser to download, install, and activate a service worker for a given scope. The `navigator.serviceWorker.register()` method initiates this process.

```javascript
navigator.serviceWorker.register('/sw.js', { scope: '/' })
```

### Why It Is Important

Registration is the entry point that connects a web application with its service worker. Without registration, the service worker code never runs. The registration process determines the service worker's scope (which pages it controls) and initiates its lifecycle.

### How It Works Internally

When `register()` is called, the browser downloads the service worker file, parses it, and installs it. If successful, the service worker enters the installing state, then transitions to installed (waiting), and eventually to activated. The service worker is then available to control pages within its scope.

```javascript
// Registration flow:
// 1. Browser downloads sw.js
// 2. Parses and compiles the script
// 3. Fires install event
// 4. Fires activate event
// 5. Service worker takes control of pages
```

### Syntax

```javascript
// Basic registration
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(registration => {
      console.log('Service Worker registered:', registration.scope)
    })
    .catch(error => {
      console.error('Registration failed:', error)
    })
}

// Registration with scope
navigator.serviceWorker.register('/sw.js', {
  scope: '/admin/'
})

// Check if already registered
navigator.serviceWorker.getRegistration().then(registration => {
  console.log('Current registration:', registration)
})
```

### Beginner Examples

```javascript
// Complete registration with error handling
async function registerServiceWorker() {
  if (!('serviceWorker' in navigator)) {
    console.log('Service Workers not supported')
    return null
  }

  try {
    const registration = await navigator.serviceWorker.register('/sw.js')
    console.log('SW registered for scope:', registration.scope)

    // Check if there's a waiting worker to update
    if (registration.waiting) {
      console.log('New version waiting')
    }

    return registration
  } catch (error) {
    console.error('SW registration failed:', error)
    return null
  }
}

// Registration with update handling
async function registerWithUpdates() {
  const registration = await navigator.serviceWorker.register('/sw.js')

  // New worker installed but waiting
  registration.addEventListener('updatefound', () => {
    const newWorker = registration.installing
    console.log('New service worker installing')

    newWorker.addEventListener('statechange', () => {
      if (newWorker.state === 'installed') {
        if (navigator.serviceWorker.controller) {
          console.log('New content available - refresh to update')
        }
      }
    })
  })

  return registration
}

// Wait for service worker to be ready
navigator.serviceWorker.ready.then(registration => {
  console.log('Service Worker is ready')
})
```

### Intermediate Examples

```javascript
// Registration with version tracking and auto-update
class ServiceWorkerManager {
  constructor(swPath = '/sw.js', scope = '/') {
    this.swPath = swPath
    this.scope = scope
    this.registration = null
  }

  async init() {
    if (!('serviceWorker' in navigator)) {
      return { supported: false }
    }

    try {
      this.registration = await navigator.serviceWorker.register(this.swPath, {
        scope: this.scope
      })

      this.setupUpdateListeners()
      this.setupControllerChange()

      return {
        supported: true,
        scope: this.registration.scope,
        state: this.getCurrentState()
      }
    } catch (error) {
      return { supported: true, error: error.message }
    }
  }

  setupUpdateListeners() {
    this.registration.addEventListener('updatefound', () => {
      const newWorker = this.registration.installing

      newWorker.addEventListener('statechange', () => {
        switch (newWorker.state) {
          case 'installed':
            if (navigator.serviceWorker.controller) {
              this.showUpdatePrompt()
            }
            break
          case 'activated':
            console.log('New service worker activated')
            break
        }
      })
    })
  }

  setupControllerChange() {
    navigator.serviceWorker.addEventListener('controllerchange', () => {
      console.log('Controller changed - page will reload on next navigation')
    })
  }

  async showUpdatePrompt() {
    // Customize this to show a UI prompt
    console.log('New version available')
    if (confirm('A new version is available. Update now?')) {
      await this.skipWaiting()
      window.location.reload()
    }
  }

  async skipWaiting() {
    if (this.registration.waiting) {
      this.registration.waiting.postMessage({ type: 'SKIP_WAITING' })
    }
  }

  async update() {
    if (this.registration) {
      await this.registration.update()
    }
  }

  getCurrentState() {
    if (!this.registration) return 'none'
    if (this.registration.installing) return 'installing'
    if (this.registration.waiting) return 'waiting'
    if (this.registration.active) return 'active'
    return 'unknown'
  }

  async unregister() {
    if (this.registration) {
      const result = await this.registration.unregister()
      console.log('Unregistered:', result)
      return result
    }
    return false
  }
}

// Usage
const swManager = new ServiceWorkerManager()
swManager.init().then(result => {
  if (!result.supported) {
    console.log('SW not supported in this browser')
  } else if (result.state !== 'active') {
    console.log('SW state:', result.state)
  }
})
```

### Advanced Examples

```javascript
// Service worker with update notification and graceful migration
// File: sw.js (service worker code)
const CACHE_NAME = 'my-app-v2'
const ASSETS_TO_CACHE = [
  '/',
  '/index.html',
  '/styles/main.css',
  '/scripts/main.js',
  '/images/logo.png'
]

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => {
      return cache.addAll(ASSETS_TO_CACHE)
    })
  )
})

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(name => {
          if (name !== CACHE_NAME) {
            console.log('Deleting old cache:', name)
            return caches.delete(name)
          }
        })
      )
    }).then(() => {
      // Take control of all clients immediately
      return self.clients.claim()
    })
  )
})

self.addEventListener('message', (event) => {
  if (event.data.type === 'SKIP_WAITING') {
    self.skipWaiting()
  }

  if (event.data.type === 'CACHE_URL') {
    caches.open(CACHE_NAME).then(cache => {
      cache.add(event.data.url)
    })
  }

  if (event.data.type === 'GET_VERSION') {
    event.ports[0].postMessage({ version: CACHE_NAME })
  }
})

// Dynamic caching strategies
const CACHING_STRATEGIES = {
  // Cache-first: try cache, fall back to network
  cacheFirst: async (request) => {
    const cachedResponse = await caches.match(request)
    if (cachedResponse) {
      return cachedResponse
    }
    try {
      const networkResponse = await fetch(request)
      const cache = await caches.open(CACHE_NAME)
      cache.put(request, networkResponse.clone())
      return networkResponse
    } catch (error) {
      return new Response('Offline', { status: 503 })
    }
  },

  // Network-first: try network, fall back to cache
  networkFirst: async (request) => {
    try {
      const networkResponse = await fetch(request)
      const cache = await caches.open(CACHE_NAME)
      cache.put(request, networkResponse.clone())
      return networkResponse
    } catch (error) {
      const cachedResponse = await caches.match(request)
      if (cachedResponse) {
        return cachedResponse
      }
      return new Response('Offline', { status: 503 })
    }
  },

  // Stale-while-revalidate: serve cache, update in background
  staleWhileRevalidate: async (request) => {
    const cache = await caches.open(CACHE_NAME)
    const cachedResponse = await cache.match(request)

    const fetchPromise = fetch(request).then(networkResponse => {
      cache.put(request, networkResponse.clone())
      return networkResponse
    }).catch(() => cachedResponse)

    return cachedResponse || fetchPromise
  },

  // Network-only: always fetch from network
  networkOnly: async (request) => {
    return fetch(request)
  },

  // Cache-only: always serve from cache
  cacheOnly: async (request) => {
    const cachedResponse = await caches.match(request)
    if (!cachedResponse) {
      throw new Error('Not in cache')
    }
    return cachedResponse
  }
}

// Route-based strategy selection
self.addEventListener('fetch', (event) => {
  const { request } = event
  const url = new URL(request.url)

  // Only handle same-origin requests
  if (url.origin !== self.location.origin) {
    return
  }

  let strategy

  if (request.destination === 'document') {
    // HTML pages: network-first
    strategy = CACHING_STRATEGIES.networkFirst
  } else if (request.destination === 'style' || request.destination === 'script') {
    // Static assets: cache-first
    strategy = CACHING_STRATEGIES.cacheFirst
  } else if (request.destination === 'image') {
    // Images: stale-while-revalidate
    strategy = CACHING_STRATEGIES.staleWhileRevalidate
  } else if (url.pathname.startsWith('/api/')) {
    // API calls: network-first
    strategy = CACHING_STRATEGIES.networkFirst
  } else {
    // Everything else: network-only
    strategy = CACHING_STRATEGIES.networkOnly
  }

  event.respondWith(strategy(request))
})

// Background sync
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-posts') {
    event.waitUntil(syncPendingPosts())
  }
})

async function syncPendingPosts() {
  const db = await openSyncDB()
  const pendingPosts = await db.getAll('pending-posts')

  for (const post of pendingPosts) {
    try {
      await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(post.data)
      })
      await db.delete('pending-posts', post.id)
    } catch (error) {
      console.error('Sync failed for post:', post.id)
    }
  }
}

// Push notifications
self.addEventListener('push', (event) => {
  let data = {}

  if (event.data) {
    try {
      data = event.data.json()
    } catch {
      data = { title: 'New Notification', body: event.data.text() }
    }
  }

  const options = {
    body: data.body || 'You have a new notification',
    icon: '/images/icon-192.png',
    badge: '/images/badge-72.png',
    data: { url: data.url || '/' },
    actions: [
      { action: 'open', title: 'Open' },
      { action: 'dismiss', title: 'Dismiss' }
    ]
  }

  event.waitUntil(
    self.registration.showNotification(data.title || 'Update', options)
  )
})

self.addEventListener('notificationclick', (event) => {
  event.notification.close()

  if (event.action === 'dismiss') return

  event.waitUntil(
    clients.openWindow(event.notification.data.url)
  )
})
```

### Real-World Use Cases

- **Offline web applications**: Full offline functionality for PWAs
- **Push notifications**: Re-engage users with server-pushed notifications
- **Background sync**: Sync data when connectivity resumes
- **Custom cache strategies**: Optimized loading for static assets and API data
- **Performance optimization**: Pre-caching critical resources
- **Content prefetching**: Predictively load likely-to-be-needed pages
- **Analytics in background**: Send analytics data even when offline
- **Authentication token refresh**: Background token refresh without user interaction

### Common Mistakes

```javascript
// Mistake: Assuming Service Worker works over HTTP
// Service Workers require HTTPS (except localhost)

// Mistake: Not handling the waiting phase
// New service worker waits until all tabs close

// Mistake: Blocking the install event
self.addEventListener('install', (event) => {
  // waitUntil is required to extend install
  // Without it, install may complete before caching finishes
})

// Mistake: Not cleaning up old caches
// Old cache data accumulates and consumes storage

// Mistake: Caching dynamic API responses without versioning
// User sees stale data after API change

// Mistake: Too broad scope
// SW on /app/ won't control /pages/

// Mistake: Using importScripts incorrectly
// ImportScripts is synchronous and only works in SW context

// Mistake: Not calling event.respondWith() in fetch handler
// Without respondWith, browser handles request normally
```

### Best Practices

```javascript
// Always wait for install to complete
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(assets))
  )
})

// Clean up old caches on activate
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(keys => {
      return Promise.all(
        keys.filter(k => k !== CACHE_NAME).map(k => caches.delete(k))
      )
    }).then(() => self.clients.claim())
  )
})

// Use meaningful cache versioning
const CACHE_VERSION = 'v2'
const STATIC_CACHE = `static-${CACHE_VERSION}`
const DYNAMIC_CACHE = `dynamic-${CACHE_VERSION}`

// Implement a cache quota management
async function trimCache(cacheName, maxItems) {
  const cache = await caches.open(cacheName)
  const requests = await cache.keys()
  if (requests.length > maxItems) {
    await cache.delete(requests[0])
    return trimCache(cacheName, maxItems)
  }
}

// Handle fetch errors gracefully
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request).catch(() => {
      return caches.match(event.request).then(cached => {
        return cached || new Response('Offline', { status: 503 })
      })
    })
  )
})
```

### Performance Considerations

- Service workers add startup latency (cold start ~100-500ms)
- Pre-caching too many assets delays installation
- Cache storage is shared with other origins (quota managed per origin)
- Large caches slow down cache lookups
- IndexedDB within SW is asynchronous but adds complexity
- Background sync requests are batched by the browser
- Push notifications require user permission and can be blocked
- Use `caches.match()` with ignoreSearch for URL normalization
- Service worker updates trigger re-download of the entire script (compare byte-by-byte)

### Interview Questions

**Q: What is the service worker lifecycle?**

A: The lifecycle has three main phases: Install (fired once when SW is first downloaded, used for pre-caching), Waiting (SW is installed but not active because an older SW is still controlling pages), and Activate (fired when the SW takes control, used for cache cleanup). The SW transitions from install -> waiting -> active.

**Q: How do you update a service worker?**

A: The browser automatically checks for SW updates every 24 hours and on navigation. When a new SW is found (byte-different), it downloads and installs but waits in 'waiting' state until all pages using the old SW are closed. You can call `self.skipWaiting()` in the new SW to activate immediately, and `self.clients.claim()` to take control of all clients.

**Q: What caching strategies can a service worker implement?**

A: Cache-first (serve from cache, fallback to network - best for static assets), Network-first (try network, fallback to cache - best for dynamic content), Stale-while-revalidate (serve cached version, update cache in background - fastest UX with fresh data), Network-only (always go to network), and Cache-only (never go to network). Each strategy balances freshness vs speed.

### Coding Challenges

```javascript
// Challenge 1: Implement a network-first strategy with timeout
async function networkFirstWithTimeout(request, timeoutMs = 3000) {
  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Timeout')), timeoutMs)
  )

  try {
    const response = await Promise.race([
      fetch(request.clone()),
      timeoutPromise
    ])
    const cache = await caches.open('dynamic')
    cache.put(request, response.clone())
    return response
  } catch {
    const cached = await caches.match(request)
    return cached || new Response('Offline', { status: 503 })
  }
}

// Challenge 2: Implement cache purging based on least recently used
async function purgeLRU(cacheName, maxAgeMs) {
  const cache = await caches.open(cacheName)
  const requests = await cache.keys()

  const now = Date.now()
  const deletePromises = []

  for (const request of requests) {
    const response = await cache.match(request)
    const dateHeader = response.headers.get('date')
    if (dateHeader) {
      const age = now - new Date(dateHeader).getTime()
      if (age > maxAgeMs) {
        deletePromises.push(cache.delete(request))
      }
    }
  }

  return Promise.all(deletePromises)
}

// Challenge 3: Implement a messaging bridge between SW and page
// In service worker:
self.addEventListener('message', (event) => {
  if (event.data.type === 'PING') {
    event.ports[0].postMessage({ type: 'PONG', timestamp: Date.now() })
  }
})

// In page:
function sendMessageToSW(message) {
  return new Promise((resolve) => {
    const channel = new MessageChannel()
    channel.port1.onmessage = (event) => resolve(event.data)

    if (navigator.serviceWorker.controller) {
      navigator.serviceWorker.controller.postMessage(message, [channel.port2])
    } else {
      resolve(null)
    }
  })
}
```

### Related Topics

- `Progressive Web Apps` - Service workers are a core PWA technology
- `Cache API` - The storage mechanism service workers use for caching
- `Fetch API` - Intercepted by service worker fetch events
- `Web Workers` - Service workers are a specialized type of web worker
- `IndexedDB` - For complex offline data storage
- `Notifications API` - Push notifications delivered via service workers
- `Background Sync` - Sync data in the background after connectivity
- `Web App Manifest` - Configure PWA appearance alongside service workers
