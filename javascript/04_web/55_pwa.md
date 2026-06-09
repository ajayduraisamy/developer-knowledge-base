# Progressive Web Apps - Manifest, service workers, offline support, app shell

## Introduction

Progressive Web Apps (PWAs) are web applications that use modern web capabilities to deliver app-like experiences to users. They combine the reach of the web with the capabilities of native apps, including offline support, push notifications, home screen installation, and access to device features. PWAs are built on three core technologies: the Web App Manifest for installation, Service Workers for offline and caching, and HTTPS for security.

## Web App Manifest

### What It Is

The Web App Manifest is a JSON file that provides metadata about a web application, enabling it to be installed on a user's device home screen. It defines the app's name, icons, start URL, display mode, orientation, and theme colors.

```json
{
  "name": "My App",
  "short_name": "App",
  "description": "A great progressive web app",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#1976d2",
  "icons": [
    {
      "src": "/images/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/images/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "orientation": "portrait",
  "scope": "/",
  "lang": "en-US",
  "categories": ["productivity", "utilities"],
  "iarc_rating_id": "e84b072d-71b3-4d3e-86ae-31a8ce4e53b7"
}
```

### Why It Is Important

The manifest enables PWA installation, making the web app feel native. It controls how the app appears when launched from the home screen, including the splash screen, status bar color, and full-screen experience. Without a manifest, a PWA cannot be installed.

### How It Works Internally

When a user visits a site with a manifest link, the browser detects the manifest and uses it to determine installability. On mobile, the browser may prompt the user to install. When installed, the OS creates a shortcut that launches the browser in the specified display mode, pointing to the start URL. The manifest icons are used for the home screen icon and splash screen.

```javascript
// Link in HTML head
<link rel="manifest" href="/manifest.json">
```

### Syntax

```json
{
  "name": "Full application name",
  "short_name": "Short name for home screen",
  "description": "App description",
  "start_url": "/?source=pwa",
  "display": "standalone | fullscreen | minimal-ui | browser",
  "background_color": "#ffffff",
  "theme_color": "#1976d2",
  "icons": [{ "src": "path", "sizes": "192x192", "type": "image/png" }],
  "orientation": "any | natural | landscape | portrait",
  "scope": "/",
  "lang": "en-US",
  "dir": "auto | ltr | rtl",
  "related_applications": [{ "platform": "play", "url": "..." }],
  "prefer_related_applications": false
}
```

### Beginner Examples

```javascript
// Check if app can be installed
let deferredPrompt

window.addEventListener('beforeinstallprompt', (event) => {
  event.preventDefault()
  deferredPrompt = event

  // Show install button
  installButton.style.display = 'block'
})

// Handle install button click
installButton.addEventListener('click', async () => {
  if (deferredPrompt) {
    deferredPrompt.prompt()
    const result = await deferredPrompt.userChoice
    console.log('User choice:', result.outcome)
    deferredPrompt = null
    installButton.style.display = 'none'
  }
})

// Check if app is running in standalone mode
const isStandalone = window.matchMedia('(display-mode: standalone)').matches
  || window.navigator.standalone === true

if (isStandalone) {
  console.log('App is running as installed PWA')
}
```

### Intermediate Examples

```javascript
// Complete PWA install experience
class PWAInstallManager {
  constructor() {
    this.deferredPrompt = null
    this.isInstalled = false
    this.installButton = document.getElementById('install-button')
    this.installBanner = document.getElementById('install-banner')

    this.init()
  }

  init() {
    // Check if already installed
    this.isInstalled = window.matchMedia('(display-mode: standalone)').matches

    if (this.isInstalled) {
      this.hideInstallPrompts()
      return
    }

    // Listen for install prompt
    window.addEventListener('beforeinstallprompt', (event) => {
      event.preventDefault()
      this.deferredPrompt = event
      this.showInstallPrompts()
    })

    // Listen for successful installation
    window.addEventListener('appinstalled', () => {
      this.isInstalled = true
      this.hideInstallPrompts()
      console.log('App installed successfully')
      this.trackInstall()
    })

    // Listen for display mode changes
    window.matchMedia('(display-mode: standalone)').addEventListener('change', (e) => {
      this.isInstalled = e.matches
      if (e.matches) {
        this.hideInstallPrompts()
      }
    })
  }

  async install() {
    if (!this.deferredPrompt) return

    this.deferredPrompt.prompt()
    const result = await this.deferredPrompt.userChoice

    if (result.outcome === 'accepted') {
      this.trackInstall('accepted')
    } else {
      this.trackInstall('dismissed')
    }

    this.deferredPrompt = null
    this.hideInstallPrompts()
  }

  showInstallPrompts() {
    if (this.installButton) {
      this.installButton.style.display = 'block'
      this.installButton.addEventListener('click', () => this.install())
    }
    if (this.installBanner) {
      this.installBanner.style.display = 'flex'
    }
  }

  hideInstallPrompts() {
    if (this.installButton) this.installButton.style.display = 'none'
    if (this.installBanner) this.installBanner.style.display = 'none'
  }

  trackInstall(source = 'prompt') {
    if (typeof gtag !== 'undefined') {
      gtag('event', 'pwa_install', { source })
    }
  }
}

// Dynamic manifest generation
async function generateManifest() {
  const config = await fetch('/api/pwa-config').then(r => r.json())

  const manifest = {
    name: config.appName,
    short_name: config.shortName,
    description: config.description,
    start_url: '/',
    display: 'standalone',
    background_color: config.backgroundColor || '#ffffff',
    theme_color: config.themeColor || '#1976d2',
    icons: [
      ...config.icons.map(size => ({
        src: `/icons/icon-${size}.png`,
        sizes: `${size}x${size}`,
        type: 'image/png',
        purpose: 'any maskable'
      }))
    ],
    categories: config.categories,
    screenshots: config.screenshots?.map(s => ({
      src: s.src,
      sizes: s.sizes,
      type: 'image/png',
      form_factor: s.formFactor || 'wide'
    }))
  }

  // Serve via a Blob URL for dynamic injection
  const blob = new Blob([JSON.stringify(manifest)], { type: 'application/json' })
  const manifestURL = URL.createObjectURL(blob)

  const link = document.querySelector('link[rel="manifest"]')
  if (link) {
    link.href = manifestURL
  }
}
```

### Advanced Examples

```javascript
// Complete PWA setup with offline support
// manifest.json
const MANIFEST = {
  name: 'Task Manager Pro',
  short_name: 'Tasks',
  description: 'Professional task management application',
  start_url: '/dashboard?source=pwa',
  display: 'standalone',
  background_color: '#f5f5f5',
  theme_color: '#1976d2',
  orientation: 'any',
  scope: '/',
  icons: [
    { src: '/icons/72.png', sizes: '72x72', type: 'image/png' },
    { src: '/icons/96.png', sizes: '96x96', type: 'image/png' },
    { src: '/icons/128.png', sizes: '128x128', type: 'image/png' },
    { src: '/icons/144.png', sizes: '144x144', type: 'image/png' },
    { src: '/icons/152.png', sizes: '152x152', type: 'image/png' },
    { src: '/icons/192.png', sizes: '192x192', type: 'image/png', purpose: 'any maskable' },
    { src: '/icons/384.png', sizes: '384x384', type: 'image/png' },
    { src: '/icons/512.png', sizes: '512x512', type: 'image/png' }
  ],
  screenshots: [
    {
      src: '/screenshots/desktop.png',
      sizes: '1280x720',
      type: 'image/png',
      form_factor: 'wide'
    }
  ],
  categories: ['productivity', 'utilities']
}

// sw.js - Complete PWA service worker
const CACHE_VERSION = 'v1'
const STATIC_CACHE = `static-${CACHE_VERSION}`
const DYNAMIC_CACHE = `dynamic-${CACHE_VERSION}`
const API_CACHE = `api-${CACHE_VERSION}`

const STATIC_ASSETS = [
  '/',
  '/offline.html',
  '/css/app.css',
  '/js/app.js',
  '/js/vendor.js',
  '/images/logo.svg',
  '/images/offline.svg',
  '/fonts/roboto.woff2',
  '/manifest.json'
]

// Install: Cache static assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(STATIC_CACHE).then(cache => {
      return cache.addAll(STATIC_ASSETS)
    })
  )
})

// Activate: Clean old caches and take control
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(keys => {
      return Promise.all(
        keys.filter(key => {
          return key !== STATIC_CACHE &&
                 key !== DYNAMIC_CACHE &&
                 key !== API_CACHE
        }).map(key => caches.delete(key))
      )
    }).then(() => self.clients.claim())
  )
})

// Fetch: App shell + API strategies
self.addEventListener('fetch', (event) => {
  const { request } = event
  const url = new URL(request.url)

  // Skip non-GET requests
  if (request.method !== 'GET') return

  // Skip chrome-extension and other non-http requests
  if (!url.protocol.startsWith('http')) return

  // API requests: Network first, fallback to cache
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirstWithFallback(request, API_CACHE))
    return
  }

  // Navigation requests: Serve app shell from cache
  if (request.mode === 'navigate') {
    event.respondWith(networkFirstWithFallback(request, STATIC_CACHE, '/offline.html'))
    return
  }

  // Static assets: Cache first
  if (STATIC_ASSETS.includes(url.pathname)) {
    event.respondWith(cacheFirst(request, STATIC_CACHE))
    return
  }

  // Other assets: Stale while revalidate
  event.respondWith(staleWhileRevalidate(request, DYNAMIC_CACHE))
})

async function cacheFirst(request, cacheName) {
  const cached = await caches.match(request)
  if (cached) return cached

  try {
    const response = await fetch(request)
    const cache = await caches.open(cacheName)
    cache.put(request, response.clone())
    return response
  } catch {
    return caches.match('/offline.html')
  }
}

async function networkFirstWithFallback(request, cacheName, fallbackUrl) {
  try {
    const response = await fetch(request)
    const cache = await caches.open(cacheName)

    // Only cache successful responses
    if (response.status === 200) {
      cache.put(request, response.clone())
    }

    return response
  } catch {
    const cached = await caches.match(request)
    if (cached) return cached

    if (fallbackUrl) {
      return caches.match(fallbackUrl)
    }

    return new Response('Offline', { status: 503 })
  }
}

async function staleWhileRevalidate(request, cacheName) {
  const cache = await caches.open(cacheName)
  const cached = await cache.match(request)

  const fetchPromise = fetch(request).then(response => {
    if (response.status === 200) {
      cache.put(request, response.clone())
    }
    return response
  }).catch(() => cached)

  return cached || fetchPromise
}

// Background sync for offline actions
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-tasks') {
    event.waitUntil(syncPendingTasks())
  }
})

async function syncPendingTasks() {
  const db = await openDatabase()
  const pending = await db.getAll('sync-queue')

  for (const item of pending) {
    try {
      await fetch(item.url, {
        method: item.method,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(item.data)
      })
      await db.delete('sync-queue', item.id)
    } catch {
      console.error('Sync failed:', item.id)
    }
  }
}

// Push notifications
self.addEventListener('push', (event) => {
  let data
  try {
    data = event.data.json()
  } catch {
    data = { title: 'Task Reminder', body: 'You have pending tasks' }
  }

  const options = {
    body: data.body,
    icon: '/icons/192.png',
    badge: '/icons/72.png',
    data: { url: data.url || '/' },
    vibrate: [200, 100, 200],
    actions: [
      { action: 'view', title: 'View Tasks' },
      { action: 'dismiss', title: 'Dismiss' }
    ]
  }

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  )
})

self.addEventListener('notificationclick', (event) => {
  event.notification.close()

  if (event.action === 'dismiss') return

  event.waitUntil(
    clients.matchAll({ type: 'window' }).then(clientList => {
      for (const client of clientList) {
        if (client.url === event.notification.data.url && 'focus' in client) {
          return client.focus()
        }
      }
      if (clients.openWindow) {
        return clients.openWindow(event.notification.data.url)
      }
    })
  )
})

// Client-side database for offline data
function openDatabase() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('TaskManagerDB', 1)
    request.onupgradeneeded = (event) => {
      const db = event.target.result
      if (!db.objectStoreNames.contains('sync-queue')) {
        db.createObjectStore('sync-queue', { keyPath: 'id', autoIncrement: true })
      }
    }
    request.onsuccess = () => resolve(request.result)
    request.onerror = () => reject(request.error)
  })
}
```

### Real-World Use Cases

- **E-commerce apps**: Offline browsing, add to cart without network
- **News/Media apps**: Offline article reading with background updates
- **Productivity tools**: Task management, note-taking with offline sync
- **Travel apps**: Offline maps, itineraries, boarding passes
- **Social media**: Offline feed with sync when online
- **Banking apps**: Transaction history offline, push payment notifications
- **Streaming services**: Offline playlists, background audio
- **Healthcare apps**: Appointment management, prescription refills

### Common Mistakes

```javascript
// Mistake: Missing maskable icon
// On Android, icons are masked into a white circle
// Without purpose: "maskable", icon may be cropped badly

// Mistake: Incorrect scope
// If scope is /app/, navigating to / will not be controlled

// Mistake: Not handling offline gracefully
// Without an offline page, users see browser error page

// Mistake: Missing start_url
// Browser may not prompt for installation

// Mistake: No HTTPS
// Service workers (and therefore offline support) require HTTPS

// Mistake: Not testing on real devices
// PWA behavior varies significantly between browsers and OS

// Mistake: Ignoring the beforeinstallprompt event
// Default prompt varies by browser; custom UX is often better
```

### Best Practices

```javascript
// Always include maskable icons
{
  "icons": [{
    "src": "/icons/maskable-192.png",
    "sizes": "192x192",
    "type": "image/png",
    "purpose": "any maskable"
  }]
}

// Set appropriate display mode
// standalone for most apps, fullscreen for games/gallery apps

// Use a meaningful start_url
"start_url": "/?utm_source=pwa"

// Cache the offline fallback page
self.addEventListener('install', (event) => {
  event.waitUntil(caches.open(CACHE).then(cache => {
    return cache.add('/offline.html')
  }))
})

// Provide feedback during installation
installButton.addEventListener('click', async () => {
  installButton.disabled = true
  installButton.textContent = 'Installing...'
})

// Test with Lighthouse
// Run Chrome DevTools > Lighthouse > PWA audit

// Update the manifest version when app changes
// Use cache-busting query params for static assets
```

### Performance Considerations

- App shell caching dramatically improves repeat-visit load times
- Service worker cold start adds ~100ms latency on first load
- Cache size is limited (per-origin quota varies by browser)
- Pre-cache only critical assets; lazy-cache the rest
- Dynamic caches should have size limits and LRU eviction
- Background sync operations are batched and throttled
- Push notifications require user opt-in
- IndexedDB operations within service worker are async but slow
- Consider using Workbox library for production caching strategies

### Interview Questions

**Q: What are the three core technologies of a PWA?**

A: HTTPS (security requirement for service workers), Web App Manifest (JSON for installable metadata), and Service Workers (background script for offline, caching, push). Together these enable installability, offline functionality, and app-like experience.

**Q: How does the app shell architecture work?**

A: The app shell is the minimal HTML, CSS, and JavaScript required to render the application frame (header, navigation, footer). It is cached on first visit by the service worker. Subsequent navigations load the shell from cache and populate content dynamically (from cache or network). This provides instant loading on repeat visits and works offline.

**Q: How do you measure PWA quality?**

A: Use Lighthouse PWA audit for automated checks. Key metrics include: installability requirements met, service worker registered and active, offline support, HTTPS, manifest present with required properties, splash screen configured, fast page loads, and responsive design. Chrome DevTools' Application panel shows PWA installation status.

### Coding Challenges

```javascript
// Challenge 1: Implement an offline queue for failed API requests
class OfflineQueue {
  constructor() {
    this.queue = []
    this.processing = false
  }

  async add(request) {
    this.queue.push(request)
    await this.saveToIndexedDB()
    this.process()
  }

  async process() {
    if (this.processing || !navigator.onLine) return
    this.processing = true

    while (this.queue.length) {
      const request = this.queue[0]
      try {
        await fetch(request.url, request.options)
        this.queue.shift()
      } catch {
        break
      }
    }

    this.processing = false
    await this.saveToIndexedDB()
  }

  async saveToIndexedDB() {
    const db = await openDB('offline-queue', 1)
    const tx = db.transaction('requests', 'readwrite')
    tx.objectStore('requests').put({
      id: 'queue',
      data: this.queue
    })
    await tx.done
  }
}

// Challenge 2: Check if all PWA requirements are met
async function checkPWARequirements() {
  const results = {
    https: window.location.protocol === 'https:' || window.location.hostname === 'localhost',
    serviceWorker: 'serviceWorker' in navigator,
    manifest: !!document.querySelector('link[rel="manifest"]'),
    indexedDB: !!window.indexedDB,
    notifications: 'Notification' in window
  }

  if (results.serviceWorker) {
    const reg = await navigator.serviceWorker.getRegistration()
    results.swRegistered = !!reg
    results.swActive = reg?.active ? true : false
  }

  results.passed = Object.values(results).every(v => v === true)
  return results
}

// Challenge 3: Cache-busting service worker
function createCacheBustingURL(url) {
  const separator = url.includes('?') ? '&' : '?'
  return `${url}${separator}_=${Date.now()}`
}

// In service worker fetch handler
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/')) {
    // Bust cache for API calls with network-first strategy
    event.respondWith(networkFirst(event.request))
  }
})
```

### Related Topics

- `Service Workers` - Backbone of PWA offline support
- `Web App Manifest` - PWA metadata for installation
- `Cache API` - Caching mechanism for offline functionality
- `IndexedDB` - Structured offline data storage
- `Notifications API` - Push notification integration
- `Background Sync` - Sync data after connectivity resumes
- `Lighthouse` - PWA auditing and quality measurement
- `Workbox` - Google's service worker library for PWA caching
