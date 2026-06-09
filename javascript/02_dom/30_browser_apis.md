# Browser APIs - Geolocation, Canvas, History, Fetch, Notification APIs

## Introduction

Modern browsers provide a rich set of APIs that enable web applications to interact with device capabilities, render graphics, manage navigation, communicate with servers, and send notifications. These APIs bridge the gap between web applications and native applications, enabling features that were once only possible on desktop or mobile platforms.

Understanding these browser APIs allows developers to create more powerful, engaging, and user-friendly web applications. Each API follows the asynchronous, event-driven pattern common to JavaScript in the browser, and most require user permission for privacy-sensitive operations.

## Geolocation API

### What It Is

The Geolocation API allows web applications to access the user's geographical location information. It provides methods for one-time position requests and continuous position tracking. The API is available through the `navigator.geolocation` object and requires user permission before providing location data.

```javascript
// One-time position request
navigator.geolocation.getCurrentPosition(
  (position) => {
    console.log('Latitude:', position.coords.latitude);
    console.log('Longitude:', position.coords.longitude);
    console.log('Accuracy:', position.coords.accuracy, 'meters');
  },
  (error) => {
    console.error('Error:', error.message);
  }
);
```

### Why It Is Important

The Geolocation API enables location-aware web applications: mapping services, local search, weather apps, ride-sharing, fitness tracking, and location-based recommendations. It provides a standardized way to access device location without platform-specific code.

### How It Works Internally

The browser uses the device's available location sources: GPS (most accurate), Wi-Fi positioning, cellular tower triangulation, and IP address (least accurate). The API abstracts these sources and provides a unified interface. The user's browser prompts for permission, and the user can grant, deny, or persist their choice.

```javascript
// Typical location sources (accuracy decreasing):
// GPS: ~5-10 meters (outdoor, clear sky)
// Wi-Fi: ~10-100 meters
// Cellular: ~100-1000 meters
// IP: ~1000-10000 meters
```

### Syntax

```javascript
// Check availability
if ('geolocation' in navigator) {
  // Geolocation is available
} else {
  // Fallback
}

// Get current position (one-time)
navigator.geolocation.getCurrentPosition(successCallback, errorCallback, options);

// Watch position (continuous)
const watchId = navigator.geolocation.watchPosition(successCallback, errorCallback, options);

// Stop watching
navigator.geolocation.clearWatch(watchId);

// Options
const options = {
  enableHighAccuracy: true,  // Use GPS if available (more battery)
  timeout: 10000,            // Max wait time in ms
  maximumAge: 60000          // Accept cached position up to 60 seconds old
};
```

### Beginner Examples

```javascript
// Example 1: Basic location request
const status = document.querySelector('#location-status');
const coords = document.querySelector('#location-coords');

function getLocation() {
  if (!navigator.geolocation) {
    status.textContent = 'Geolocation is not supported';
    return;
  }

  status.textContent = 'Getting location...';

  navigator.geolocation.getCurrentPosition(
    (position) => {
      status.textContent = 'Location found';
      coords.textContent = `
        Latitude: ${position.coords.latitude.toFixed(4)}
        Longitude: ${position.coords.longitude.toFixed(4)}
        Accuracy: ${position.coords.accuracy.toFixed(1)} meters
      `;
    },
    (error) => {
      status.textContent = 'Error getting location';
      coords.textContent = getErrorMessage(error);
    },
    { enableHighAccuracy: true, timeout: 10000 }
  );
}

function getErrorMessage(error) {
  switch (error.code) {
    case error.PERMISSION_DENIED: return 'User denied the request';
    case error.POSITION_UNAVAILABLE: return 'Location unavailable';
    case error.TIMEOUT: return 'Request timed out';
    default: return 'Unknown error';
  }
}

document.querySelector('#get-location-btn').addEventListener('click', getLocation);

// Example 2: Show location on map
function showOnMap(latitude, longitude) {
  const mapUrl = `https://maps.google.com/maps?q=${latitude},${longitude}&z=15`;
  window.open(mapUrl, '_blank');
}

navigator.geolocation.getCurrentPosition(
  (position) => {
    showOnMap(position.coords.latitude, position.coords.longitude);
  }
);

// Example 3: Distance calculation
function calculateDistance(lat1, lon1, lat2, lon2) {
  const R = 6371000; // Earth's radius in meters
  const toRad = (deg) => deg * Math.PI / 180;

  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);
  const a = Math.sin(dLat / 2) ** 2 +
    Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon / 2) ** 2;
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
}

navigator.geolocation.getCurrentPosition((pos) => {
  const distance = calculateDistance(
    pos.coords.latitude, pos.coords.longitude,
    40.7128, -74.0060 // New York City
  );
  console.log(`Distance to NYC: ${(distance / 1000).toFixed(1)} km`);
});
```

### Intermediate Examples

```javascript
// Example 1: Location tracker with watchPosition
class LocationTracker {
  constructor() {
    this.watchId = null;
    this.locations = [];
    this.listeners = new Set();
  }

  start() {
    if (!navigator.geolocation) {
      throw new Error('Geolocation not supported');
    }

    this.watchId = navigator.geolocation.watchPosition(
      (position) => {
        const loc = {
          lat: position.coords.latitude,
          lng: position.coords.longitude,
          accuracy: position.coords.accuracy,
          altitude: position.coords.altitude,
          speed: position.coords.speed,
          timestamp: position.timestamp
        };

        this.locations.push(loc);
        this.notify(loc);

        if (loc.speed !== null) {
          console.log(`Speed: ${(loc.speed * 3.6).toFixed(1)} km/h`);
        }
      },
      (error) => console.error('Tracking error:', error.message),
      { enableHighAccuracy: true, maximumAge: 5000 }
    );
  }

  stop() {
    if (this.watchId !== null) {
      navigator.geolocation.clearWatch(this.watchId);
      this.watchId = null;
    }
  }

  onLocation(callback) {
    this.listeners.add(callback);
    return () => this.listeners.delete(callback);
  }

  notify(location) {
    for (const listener of this.listeners) {
      listener(location);
    }
  }

  getPath() {
    return [...this.locations];
  }

  getTotalDistance() {
    let total = 0;
    for (let i = 1; i < this.locations.length; i++) {
      total += calculateDistance(
        this.locations[i - 1].lat, this.locations[i - 1].lng,
        this.locations[i].lat, this.locations[i].lng
      );
    }
    return total;
  }
}

const tracker = new LocationTracker();
tracker.onLocation((loc) => {
  console.log(`Location updated: ${loc.lat}, ${loc.lng}`);
});
tracker.start();

// Example 2: Reverse geocoding (requires external API)
async function reverseGeocode(lat, lng) {
  try {
    const response = await fetch(
      `https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lng}`
    );
    const data = await response.json();
    return data.display_name;
  } catch {
    return 'Location unavailable';
  }
}

navigator.geolocation.getCurrentPosition(async (pos) => {
  const address = await reverseGeocode(pos.coords.latitude, pos.coords.longitude);
  console.log('You are at:', address);
});

// Example 3: Location-based content
async function getLocalContent() {
  try {
    const position = await new Promise((resolve, reject) => {
      navigator.geolocation.getCurrentPosition(resolve, reject, { timeout: 5000 });
    });

    const { latitude, longitude } = position.coords;
    const response = await fetch(`/api/nearby?lat=${latitude}&lng=${longitude}`);
    const places = await response.json();

    displayPlaces(places);
  } catch {
    // Location failed, show default content
    showDefaultContent();
  }
}
```

### Advanced Examples

```javascript
// Example 1: Geofencing with watchPosition
class Geofence {
  constructor(center, radius) {
    this.center = center; // { lat, lng }
    this.radius = radius; // meters
    this.inside = false;
    this.callbacks = { enter: [], exit: [] };
  }

  check(position) {
    const distance = calculateDistance(
      this.center.lat, this.center.lng,
      position.lat, position.lng
    );

    const wasInside = this.inside;
    this.inside = distance <= this.radius;

    if (this.inside && !wasInside) {
      this.callbacks.enter.forEach(cb => cb(position));
    } else if (!this.inside && wasInside) {
      this.callbacks.exit.forEach(cb => cb(position));
    }
  }

  onEnter(callback) { this.callbacks.enter.push(callback); }
  onExit(callback) { this.callbacks.exit.push(callback); }
}

const office = new Geofence({ lat: 40.7128, lng: -74.0060 }, 500);

office.onEnter(() => {
  showNotification('Welcome to the office!');
  checkIn();
});

office.onExit(() => {
  showNotification('You left the office area');
  checkOut();
});

const tracker = new LocationTracker();
tracker.onLocation((loc) => office.check(loc));
tracker.start();

// Example 2: Location permission management
class LocationPermissionManager {
  static async checkStatus() {
    if (!navigator.permissions) return 'unknown';
    try {
      const result = await navigator.permissions.query({ name: 'geolocation' });
      return result.state; // 'granted', 'denied', 'prompt'
    } catch {
      return 'unknown';
    }
  }

  static async request() {
    return new Promise((resolve) => {
      navigator.geolocation.getCurrentPosition(
        (position) => resolve({ granted: true, position }),
        (error) => resolve({ granted: false, error: error.code }),
        { timeout: 10000 }
      );
    });
  }
}

async function initLocation() {
  const status = await LocationPermissionManager.checkStatus();

  switch (status) {
    case 'granted':
      console.log('Location already permitted');
      return LocationPermissionManager.request();
    case 'denied':
      console.warn('Location permission denied');
      showPermissionPrompt();
      return null;
    case 'prompt':
    case 'unknown':
      console.log('Requesting location permission...');
      return LocationPermissionManager.request();
  }
}

// Example 3: Battery-efficient location polling
class EfficientLocationPoller {
  constructor(options = {}) {
    this.interval = options.interval || 60000; // 1 minute
    this.minAccuracy = options.minAccuracy || 100; // meters
    this.lastPosition = null;
    this.timer = null;
  }

  start() {
    // Initial high-accuracy fix
    navigator.geolocation.getCurrentPosition(
      (pos) => this.handlePosition(pos),
      null,
      { enableHighAccuracy: true, timeout: 15000 }
    );

    // Then poll at lower accuracy
    this.timer = setInterval(() => {
      navigator.geolocation.getCurrentPosition(
        (pos) => this.handlePosition(pos),
        null,
        {
          enableHighAccuracy: false,
          timeout: 10000,
          maximumAge: this.interval
        }
      );
    }, this.interval);
  }

  handlePosition(position) {
    const { latitude, longitude, accuracy } = position.coords;

    if (accuracy > this.minAccuracy) {
      console.warn(`Low accuracy (${accuracy}m), skipping update`);
      return;
    }

    const significantChange = this.lastPosition &&
      calculateDistance(
        this.lastPosition.lat, this.lastPosition.lng,
        latitude, longitude
      ) > 50; // 50 meters threshold

    if (significantChange || !this.lastPosition) {
      this.lastPosition = { lat: latitude, lng: longitude, accuracy };
      this.onUpdate?.(this.lastPosition);
    }
  }

  onUpdate(callback) {
    this.onUpdate = callback;
  }

  stop() {
    clearInterval(this.timer);
    this.timer = null;
  }
}
```

### Real-World Use Cases

- **Mapping and Navigation**: Google Maps, Waze, Apple Maps
- **Fitness Tracking**: Strava, Runkeeper, MapMyRun
- **Ride Sharing**: Uber, Lyft pickup location
- **Weather Apps**: Local weather based on current position
- **Social Media**: Geotagging posts, check-ins
- **E-commerce**: Nearby stores, local inventory
- **Emergency Services**: Location-aware emergency calls
- **Field Service**: Technician location tracking
- **Geofencing**: Location-based triggers and notifications

### Common Mistakes

```javascript
// Mistake 1: Not handling permission denial
navigator.geolocation.getCurrentPosition(
  (pos) => { /* position handler */ }
  // No error handler! User denial will fail silently
);

// Mistake 2: Forgetting to clear watches
const id = navigator.geolocation.watchPosition(handler);
// Later:
// navigator.geolocation.clearWatch(id); // Don't forget!

// Mistake 3: Not setting a timeout
navigator.geolocation.getCurrentPosition(handler);
// Could wait indefinitely if GPS fix takes too long

// Mistake 4: High accuracy mode when not needed
// enableHighAccuracy: true uses more battery
// Use false for most cases, true only when precise location is essential
```

### Best Practices

- Always provide error callbacks for location requests
- Use `enableHighAccuracy: false` by default for battery conservation
- Set reasonable timeouts (5-15 seconds)
- Clear watches when no longer needed
- Explain why location is needed before requesting permission
- Cache location data when appropriate (use `maximumAge`)
- Fall back gracefully when location is unavailable or denied
- Consider user privacy—only request location when necessary

### Performance Considerations

GPS location requests are battery-intensive, especially with `enableHighAccuracy: true`. Use `watchPosition` instead of polling `getCurrentPosition` for continuous tracking, as it allows the browser to optimize hardware usage. Set `maximumAge` to reuse cached positions and reduce hardware requests.

### Interview Questions

1. What is the difference between `getCurrentPosition` and `watchPosition`?
2. What error codes can the Geolocation API return?
3. How does the browser determine location?
4. How do you check geolocation permission status?
5. What is the `enableHighAccuracy` option?
6. How do you stop watching position changes?
7. What is `maximumAge` and when would you use it?
8. How does the Geolocation API handle user privacy?

### Coding Challenges

1. **Location Tracker**: Build a real-time location tracker with distance calculation
2. **Geofence Alert**: Implement geofence enter/exit detection
3. **Permission Manager**: Build a location permission request flow with explanations
4. **Location History**: Create a location history viewer with path visualization

### Related Topics

- [Permissions API](https://developer.mozilla.org/en-US/docs/Web/API/Permissions_API)
- [Geolocation Sensor API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_Sensor_API)
- [PositionOptions](https://developer.mozilla.org/en-US/docs/Web/API/PositionOptions)

## Canvas API

### What It Is

The Canvas API provides a way to draw graphics programmatically using JavaScript. It operates on a `<canvas>` HTML element and provides a 2D rendering context with methods for drawing shapes, text, images, and animations. The Canvas API is pixel-based (raster) and is ideal for real-time graphics, games, data visualization, and image processing.

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

ctx.fillStyle = 'blue';
ctx.fillRect(10, 10, 100, 50);

ctx.strokeStyle = 'red';
ctx.lineWidth = 2;
ctx.strokeRect(10, 10, 100, 50);
```

### Why It Is Important

The Canvas API enables rich, interactive graphics without plugins. It is the foundation for data visualization libraries (Chart.js, D3.js), games (Phaser, Pixi.js), image editors, and creative applications. Unlike SVG (vector), Canvas is pixel-based and better suited for frequent redrawing and pixel manipulation.

### How It Works Internally

The `<canvas>` element has a fixed-size drawing surface with a 2D or 3D (WebGL) rendering context. When drawing operations are called, they are stored in an internal command buffer. The buffer is flushed and rasterized to the canvas bitmap when `ctx.draw()` operations occur or at the next animation frame.

```javascript
// Canvas coordinate system:
// (0, 0) ----> x (width)
//   |
//   |  Drawing area
//   |
//   v y (height)
```

### Syntax

```javascript
// Setup
const canvas = document.getElementById('canvas');
canvas.width = 800;
canvas.height = 600;
const ctx = canvas.getContext('2d');

// Drawing primitives
ctx.fillRect(x, y, width, height);
ctx.strokeRect(x, y, width, height);
ctx.clearRect(x, y, width, height);

// Paths
ctx.beginPath();
ctx.moveTo(x, y);
ctx.lineTo(x, y);
ctx.arc(x, y, radius, startAngle, endAngle);
ctx.rect(x, y, width, height);
ctx.closePath();
ctx.fill();
ctx.stroke();

// Text
ctx.fillText(text, x, y);
ctx.strokeText(text, x, y);
ctx.font = '16px Arial';
ctx.textAlign = 'center';

// Images
ctx.drawImage(image, x, y);
ctx.drawImage(image, sx, sy, sw, sh, dx, dy, dw, dh);

// Styles
ctx.fillStyle = 'red'; // Color string
ctx.strokeStyle = '#ff0000';
ctx.lineWidth = 2;
ctx.globalAlpha = 0.5;
ctx.shadowBlur = 10;
```

### Beginner Examples

```javascript
// Example 1: Basic shapes
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

// Rectangle
ctx.fillStyle = '#3498db';
ctx.fillRect(50, 50, 200, 100);

// Circle
ctx.beginPath();
ctx.arc(400, 100, 50, 0, Math.PI * 2);
ctx.fillStyle = '#e74c3c';
ctx.fill();

// Line
ctx.beginPath();
ctx.moveTo(50, 250);
ctx.lineTo(550, 250);
ctx.strokeStyle = '#2ecc71';
ctx.lineWidth = 3;
ctx.stroke();

// Text
ctx.font = '24px Arial';
ctx.fillStyle = '#333';
ctx.textAlign = 'center';
ctx.fillText('Hello Canvas', 300, 350);

// Example 2: Drawing chart
function drawBarChart(data, canvas) {
  const ctx = canvas.getContext('2d');
  const width = canvas.width;
  const height = canvas.height;
  const barWidth = width / data.length;
  const maxVal = Math.max(...data);
  const padding = 40;

  ctx.clearRect(0, 0, width, height);

  // Draw bars
  data.forEach((value, index) => {
    const barHeight = (value / maxVal) * (height - padding * 2);
    const x = index * barWidth + barWidth * 0.1;
    const y = height - padding - barHeight;

    ctx.fillStyle = `hsl(${index * 40}, 70%, 50%)`;
    ctx.fillRect(x, y, barWidth * 0.8, barHeight);

    // Label
    ctx.fillStyle = '#333';
    ctx.font = '12px Arial';
    ctx.textAlign = 'center';
    ctx.fillText(value, x + barWidth * 0.4, height - padding + 20);
  });
}

drawBarChart([23, 45, 67, 12, 89, 34], document.getElementById('chart'));

// Example 3: Simple animation
let x = 0;
function animate() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  ctx.fillStyle = '#e74c3c';
  ctx.beginPath();
  ctx.arc(x, 300, 30, 0, Math.PI * 2);
  ctx.fill();

  x += 2;
  if (x > canvas.width) x = 0;

  requestAnimationFrame(animate);
}
animate();
```

### Intermediate Examples

```javascript
// Example 1: Interactive drawing app
class DrawingApp {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.isDrawing = false;
    this.lastX = 0;
    this.lastY = 0;
    this.setup();
  }

  setup() {
    this.ctx.lineWidth = 5;
    this.ctx.lineCap = 'round';
    this.ctx.lineJoin = 'round';
    this.ctx.strokeStyle = '#333';

    this.canvas.addEventListener('mousedown', (e) => this.startDrawing(e));
    this.canvas.addEventListener('mousemove', (e) => this.draw(e));
    this.canvas.addEventListener('mouseup', () => this.stopDrawing());
    this.canvas.addEventListener('mouseleave', () => this.stopDrawing());

    // Touch support
    this.canvas.addEventListener('touchstart', (e) => {
      e.preventDefault();
      this.startDrawing(e.touches[0]);
    });
    this.canvas.addEventListener('touchmove', (e) => {
      e.preventDefault();
      this.draw(e.touches[0]);
    });
    this.canvas.addEventListener('touchend', () => this.stopDrawing());
  }

  startDrawing(e) {
    this.isDrawing = true;
    const rect = this.canvas.getBoundingClientRect();
    this.lastX = e.clientX - rect.left;
    this.lastY = e.clientY - rect.top;
  }

  draw(e) {
    if (!this.isDrawing) return;

    const rect = this.canvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;

    this.ctx.beginPath();
    this.ctx.moveTo(this.lastX, this.lastY);
    this.ctx.lineTo(x, y);
    this.ctx.stroke();

    this.lastX = x;
    this.lastY = y;
  }

  stopDrawing() {
    this.isDrawing = false;
  }

  setColor(color) {
    this.ctx.strokeStyle = color;
  }

  setLineWidth(width) {
    this.ctx.lineWidth = width;
  }

  clear() {
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
  }

  getDataURL() {
    return this.canvas.toDataURL('image/png');
  }
}

const app = new DrawingApp(document.getElementById('draw-canvas'));

// Example 2: Image pixel manipulation
function applyFilter(canvas, filterType) {
  const ctx = canvas.getContext('2d');
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  const data = imageData.data;

  for (let i = 0; i < data.length; i += 4) {
    const r = data[i];
    const g = data[i + 1];
    const b = data[i + 2];

    switch (filterType) {
      case 'grayscale':
        const gray = 0.299 * r + 0.587 * g + 0.114 * b;
        data[i] = data[i + 1] = data[i + 2] = gray;
        break;
      case 'invert':
        data[i] = 255 - r;
        data[i + 1] = 255 - g;
        data[i + 2] = 255 - b;
        break;
      case 'sepia':
        data[i] = Math.min(255, r * 0.393 + g * 0.769 + b * 0.189);
        data[i + 1] = Math.min(255, r * 0.349 + g * 0.686 + b * 0.168);
        data[i + 2] = Math.min(255, r * 0.272 + g * 0.534 + b * 0.131);
        break;
    }
  }

  ctx.putImageData(imageData, 0, 0);
}

// Example 3: Particle system
class Particle {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.vx = (Math.random() - 0.5) * 4;
    this.vy = (Math.random() - 0.5) * 4;
    this.life = 1;
    this.decay = Math.random() * 0.02 + 0.005;
  }

  update() {
    this.x += this.vx;
    this.y += this.vy;
    this.life -= this.decay;
  }

  draw(ctx) {
    ctx.globalAlpha = this.life;
    ctx.fillStyle = `hsl(${this.life * 360}, 100%, 50%)`;
    ctx.beginPath();
    ctx.arc(this.x, this.y, 3, 0, Math.PI * 2);
    ctx.fill();
    ctx.globalAlpha = 1;
  }
}

class ParticleSystem {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.particles = [];
    this.animate = this.animate.bind(this);
  }

  emit(x, y, count = 10) {
    for (let i = 0; i < count; i++) {
      this.particles.push(new Particle(x, y));
    }
  }

  animate() {
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

    this.particles.forEach(p => {
      p.update();
      p.draw(this.ctx);
    });

    this.particles = this.particles.filter(p => p.life > 0);
    requestAnimationFrame(this.animate);
  }

  start() {
    this.animate();
  }
}
```

### Advanced Examples

```javascript
// Example 1: Canvas-based data URL generation
class CanvasCapture {
  constructor(canvas) {
    this.canvas = canvas;
  }

  toBlob(type = 'image/png', quality = 0.92) {
    return new Promise((resolve) => {
      this.canvas.toBlob((blob) => resolve(blob), type, quality);
    });
  }

  toDataURL(type = 'image/png') {
    return this.canvas.toDataURL(type);
  }

  async toFile(filename = 'canvas.png', type = 'image/png') {
    const blob = await this.toBlob(type);
    return new File([blob], filename, { type });
  }

  async download(filename = 'canvas.png') {
    const blob = await this.toBlob();
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.click();
    URL.revokeObjectURL(url);
  }
}

// Example 2: Canvas compositing and layering
class CanvasCompositor {
  constructor(width, height) {
    this.width = width;
    this.height = height;
    this.layers = [];
  }

  addLayer(name, drawFn, opacity = 1) {
    const canvas = document.createElement('canvas');
    canvas.width = this.width;
    canvas.height = this.height;
    const ctx = canvas.getContext('2d');

    drawFn(ctx, this.width, this.height);

    this.layers.push({ name, canvas, opacity });
  }

  render(targetCanvas) {
    const ctx = targetCanvas.getContext('2d');
    ctx.clearRect(0, 0, this.width, this.height);

    for (const layer of this.layers) {
      ctx.globalAlpha = layer.opacity;
      ctx.drawImage(layer.canvas, 0, 0);
    }

    ctx.globalAlpha = 1;
  }
}

const compositor = new CanvasCompositor(800, 600);

compositor.addLayer('background', (ctx, w, h) => {
  const gradient = ctx.createLinearGradient(0, 0, 0, h);
  gradient.addColorStop(0, '#87CEEB');
  gradient.addColorStop(1, '#228B22');
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, w, h);
});

compositor.addLayer('grid', (ctx, w, h) => {
  ctx.strokeStyle = 'rgba(255,255,255,0.3)';
  ctx.lineWidth = 1;
  for (let x = 0; x < w; x += 50) {
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, h);
    ctx.stroke();
  }
  for (let y = 0; y < h; y += 50) {
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(w, y);
    ctx.stroke();
  }
});

// Example 3: Canvas animation loop with FPS control
class AnimationLoop {
  constructor(canvas, fps = 60) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.fps = fps;
    this.frameInterval = 1000 / fps;
    this.lastTime = 0;
    this.running = false;
    this.frame = 0;
  }

  start() {
    this.running = true;
    this.lastTime = performance.now();
    this.loop(this.lastTime);
  }

  stop() {
    this.running = false;
  }

  loop(currentTime) {
    if (!this.running) return;

    const deltaTime = currentTime - this.lastTime;

    if (deltaTime >= this.frameInterval) {
      this.lastTime = currentTime - (deltaTime % this.frameInterval);
      this.frame++;
      this.update(deltaTime);
      this.render();
    }

    requestAnimationFrame((time) => this.loop(time));
  }

  update(deltaTime) {
    // Override in subclass
  }

  render() {
    // Override in subclass
  }
}

class BouncingBalls extends AnimationLoop {
  constructor(canvas) {
    super(canvas, 60);
    this.balls = Array.from({ length: 10 }, () => ({
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      vx: (Math.random() - 0.5) * 4,
      vy: (Math.random() - 0.5) * 4,
      radius: Math.random() * 20 + 10,
      color: `hsl(${Math.random() * 360}, 70%, 50%)`
    }));
  }

  update(dt) {
    const factor = dt / 16; // Normalize to ~60fps

    for (const ball of this.balls) {
      ball.x += ball.vx * factor;
      ball.y += ball.vy * factor;

      if (ball.x < ball.radius || ball.x > this.canvas.width - ball.radius) {
        ball.vx *= -1;
      }
      if (ball.y < ball.radius || ball.y > this.canvas.height - ball.radius) {
        ball.vy *= -1;
      }
    }
  }

  render() {
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

    for (const ball of this.balls) {
      this.ctx.beginPath();
      this.ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
      this.ctx.fillStyle = ball.color;
      this.ctx.fill();
    }
  }
}

// const balls = new BouncingBalls(document.getElementById('canvas'));
// balls.start();
```

### Real-World Use Cases

- **Data Visualization**: Chart.js, D3.js canvas renderer, custom dashboards
- **Games**: Browser-based 2D games, game editors
- **Image Processing**: Photo filters, cropping, resizing tools
- **Drawing Applications**: Whiteboards, paint apps, annotation tools
- **Animation**: Banners, loading animations, particle effects
- **Video Processing**: Frame-by-frame analysis, video filters
- **PDF Rendering**: Page-by-page canvas rendering
- **Captcha Generation**: Dynamic image generation
- **Graphical Editors**: Flowchart tools, diagram editors

### Common Mistakes

```javascript
// Mistake 1: Forgetting to call beginPath() before drawing paths
ctx.moveTo(10, 10);
ctx.lineTo(100, 100);
ctx.stroke(); // Re-strokes previous path too!

// Mistake 2: Not clearing canvas before redrawing
ctx.fillRect(0, 0, 100, 100);
// Next frame without clearRect:
ctx.fillRect(50, 50, 100, 100); // Both rectangles visible!

// Mistake 3: Canvas size vs CSS display size mismatch
canvas.style.width = '800px'; // CSS size
canvas.width = 800; // Drawing surface size (default 300x150!)
// They need to match or drawing will be distorted

// Mistake 4: Heavy operations in animation loop
function animate() {
  // Don't do heavy synchronous operations here
  for (let i = 0; i < 10000; i++) { /* heavy loop */ }
  drawFrame();
  requestAnimationFrame(animate);
}
```

### Best Practices

- Match canvas dimensions to CSS dimensions for crisp rendering
- Use `requestAnimationFrame` for smooth animations (not `setTimeout`)
- Clear canvas (`clearRect`) before each animation frame
- Batch drawing operations to minimize state changes
- Use `beginPath()` before each new path to avoid accumulated paths
- Consider using off-screen canvas for complex pre-rendering
- Avoid `setInterval`/`setTimeout` for animations
- Use `canvas.toBlob()` for generating image data (async, non-blocking)
- Handle HiDPI/Retina displays by multiplying canvas dimensions by `devicePixelRatio`

### Performance Considerations

Canvas is generally faster than SVG for complex animations but slower for static graphics. Each frame must be fully redrawn, unlike SVG where the browser handles incremental updates. Use off-screen canvases for pre-rendering static elements. Limit the number of drawing operations per frame. Use `requestAnimationFrame` which pauses when the tab is not visible.

### Interview Questions

1. What is the difference between Canvas and SVG?
2. How do you handle Retina displays with Canvas?
3. What is `requestAnimationFrame` and why is it better than `setTimeout`?
4. How do you save and restore canvas state?
5. What is `getImageData` and `putImageData` used for?
6. How do you draw text on a canvas?
7. What is `toDataURL` and `toBlob`?
8. How do you handle click events on specific canvas elements?
9. What is the canvas coordinate system?
10. How do you optimize canvas performance?

### Coding Challenges

1. **Drawing App**: Build a canvas-based drawing application with colors, brush sizes, and undo
2. **Particle System**: Create a fireworks or confetti particle system
3. **Image Filter**: Implement grayscale, sepia, blur, and edge detection filters
4. **Chart Renderer**: Build a bar chart, line chart, and pie chart renderer
5. **Game**: Create a simple breakout or snake game using canvas

### Related Topics

- [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API)
- [OffscreenCanvas](https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvas)
- [CanvasRenderingContext2D](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D)
- [ImageData](https://developer.mozilla.org/en-US/docs/Web/API/ImageData)

## History API

### What It Is

The History API provides access to the browser's session history (the stack of pages visited in the current tab). It allows JavaScript to navigate through the history, modify the current URL without a page reload, and respond to history navigation events. It is the foundation for client-side routing in single-page applications (SPAs).

```javascript
// Navigate through history
history.back();
history.forward();
history.go(-2); // Go back 2 pages

// Modify history (push a new entry)
history.pushState({ page: 1 }, 'Page 1', '/page1');

// Replace current entry (no new entry created)
history.replaceState({ page: 2 }, 'Page 2', '/page2');
```

### Why It Is Important

The History API enables single-page applications with proper URL management and browser navigation support. Without it, SPAs would break the back/forward buttons and could not deep-link to specific states. It allows web applications to behave like native applications while maintaining standard web navigation patterns.

### How It Works Internally

The browser maintains a stack of history entries. Each entry has an associated state object, title, and URL. When the user navigates (clicks back/forward), the browser fires a `popstate` event. `pushState` adds a new entry to the stack. `replaceState` replaces the current entry. The URL is updated in the address bar but the page does not reload.

```javascript
// History stack (conceptual):
// Stack top (current):
//   { state: { page: 3 }, url: '/page3' }  <- current
//   { state: { page: 2 }, url: '/page2' }
//   { state: null, url: '/page1' }
// Stack bottom (initial):
```

### Syntax

```javascript
// Navigation
history.back();              // Same as history.go(-1)
history.forward();           // Same as history.go(1)
history.go(n);               // Go n steps (negative = back, positive = forward)
history.length;              // Number of entries in history

// Manipulation
history.pushState(state, title, url);
// state: any serializable data associated with the entry
// title: string (mostly ignored by browsers)
// url: string (new URL, must be same-origin)

history.replaceState(state, title, url);
// Replaces the current entry instead of adding a new one

// Events
window.addEventListener('popstate', (event) => {
  console.log('State:', event.state);
  // event.state is the state object from pushState/replaceState
});
```

### Beginner Examples

```javascript
// Example 1: Simple SPA navigation
const pages = {
  home: '<h1>Home</h1><p>Welcome to my site</p>',
  about: '<h1>About</h1><p>Learn about us</p>',
  contact: '<h1>Contact</h1><p>Get in touch</p>'
};

function navigate(page) {
  const content = document.getElementById('content');
  content.innerHTML = pages[page] || '<h1>404</h1>';

  // Update URL without page reload
  history.pushState({ page }, `Page: ${page}`, `/${page}`);

  // Update active nav link
  document.querySelectorAll('nav a').forEach(a => a.classList.remove('active'));
  document.querySelector(`[data-page="${page}"]`)?.classList.add('active');
}

// Handle back/forward
window.addEventListener('popstate', (event) => {
  const page = event.state?.page || 'home';
  document.getElementById('content').innerHTML = pages[page];
});

// Intercept link clicks
document.addEventListener('click', (e) => {
  const link = e.target.closest('[data-page]');
  if (link) {
    e.preventDefault();
    navigate(link.dataset.page);
  }
});

// Example 2: Tab state preservation
function saveTabState() {
  const activeTab = document.querySelector('.tab.active')?.dataset.tab || 'overview';
  const searchQuery = document.querySelector('#search')?.value || '';

  history.replaceState(
    { tab: activeTab, search: searchQuery },
    '',
    `?tab=${activeTab}&q=${encodeURIComponent(searchQuery)}`
  );
}

// Example 3: Modal state in URL
function openModal(modalId) {
  document.getElementById(modalId).classList.add('open');
  history.pushState({ modal: modalId }, '', `?modal=${modalId}`);
}

function closeModal() {
  document.querySelector('.modal.open')?.classList.remove('open');
  history.back();
}

window.addEventListener('popstate', (event) => {
  if (event.state?.modal) {
    document.getElementById(event.state.modal).classList.add('open');
  } else {
    document.querySelector('.modal.open')?.classList.remove('open');
  }
});
```

### Intermediate Examples

```javascript
// Example 1: SPA Router with History API
class SPARouter {
  constructor(routes) {
    this.routes = routes;
    this.setup();
  }

  setup() {
    window.addEventListener('popstate', (event) => {
      this.handleRoute(window.location.pathname, event.state);
    });

    document.addEventListener('click', (e) => {
      const link = e.target.closest('a[data-router]');
      if (link) {
        e.preventDefault();
        this.navigate(link.getAttribute('href'));
      }
    });

    // Handle initial route
    this.handleRoute(window.location.pathname);
  }

  navigate(path) {
    const route = this.matchRoute(path);
    if (route) {
      history.pushState({ path }, '', path);
      this.handleRoute(path, { path });
    }
  }

  handleRoute(path, state) {
    const route = this.matchRoute(path);
    if (route) {
      route.handler(state);
    } else {
      this.handleNotFound(path);
    }
  }

  matchRoute(path) {
    for (const [pattern, handler] of this.routes) {
      const match = path.match(pattern);
      if (match) {
        return { handler, params: match.slice(1) };
      }
    }
    return null;
  }

  handleNotFound(path) {
    console.warn(`No route for: ${path}`);
    this.render('404 - Page Not Found');
  }

  render(html) {
    document.getElementById('app').innerHTML = html;
  }
}

const router = new SPARouter([
  [/^\/$/, () => '<h1>Home</h1>'],
  [/^\/users\/(\d+)$/, (state, params) => `<h1>User ${params[0]}</h1>`],
  [/^\/settings$/, () => '<h1>Settings</h1>']
]);

// Example 2: History state with undo
class HistoryStateManager {
  constructor(maxStates = 50) {
    this.maxStates = maxStates;
    this.undoStack = [];
    this.redoStack = [];
  }

  pushState(state, url) {
    // Store previous state before pushing
    const current = {
      state: history.state,
      url: window.location.pathname
    };

    this.undoStack.push(current);
    this.redoStack = []; // Clear redo on new action

    if (this.undoStack.length > this.maxStates) {
      this.undoStack.shift();
    }

    history.pushState(state, '', url);
  }

  undo() {
    if (this.undoStack.length === 0) return false;

    const current = {
      state: history.state,
      url: window.location.pathname
    };

    const previous = this.undoStack.pop();
    this.redoStack.push(current);

    history.replaceState(previous.state, '', previous.url);
    window.dispatchEvent(new PopStateEvent('popstate', { state: previous.state }));

    return true;
  }

  redo() {
    if (this.redoStack.length === 0) return false;

    const current = {
      state: history.state,
      url: window.location.pathname
    };

    const next = this.redoStack.pop();
    this.undoStack.push(current);

    history.replaceState(next.state, '', next.url);
    window.dispatchEvent(new PopStateEvent('popstate', { state: next.state }));

    return true;
  }
}

// Example 3: History-aware forms
class HistoryAwareForm {
  constructor(form) {
    this.form = form;
    this.initialState = this.serialize();
    this.setup();
  }

  setup() {
    this.form.addEventListener('input', () => {
      const currentState = this.serialize();
      if (currentState !== this.initialState) {
        history.replaceState(
          { formData: currentState },
          '',
          window.location.pathname + (currentState ? '?draft=true' : '')
        );
      }
    });

    window.addEventListener('popstate', (event) => {
      if (event.state?.formData) {
        this.deserialize(event.state.formData);
      }
    });
  }

  serialize() {
    return new URLSearchParams(new FormData(this.form)).toString();
  }

  deserialize(data) {
    const params = new URLSearchParams(data);
    for (const [key, value] of params) {
      const field = this.form.querySelector(`[name="${key}"]`);
      if (field) field.value = value;
    }
  }
}
```

### Advanced Examples

```javascript
// Example 1: Full-featured SPA router
class AdvancedRouter {
  constructor(options = {}) {
    this.routes = [];
    this.middleware = [];
    this.container = options.container || document.getElementById('app');
    this.beforeHooks = [];
    this.afterHooks = [];
    this.setup();
  }

  setup() {
    window.addEventListener('popstate', (event) => {
      this.resolveRoute(window.location.pathname + window.location.search, event.state);
    });

    document.addEventListener('click', (e) => {
      const link = e.target.closest('a:not([target])');
      if (link && this.isInternalLink(link)) {
        e.preventDefault();
        this.navigate(link.pathname + link.search);
      }
    });
  }

  isInternalLink(link) {
    return link.origin === window.location.origin ||
      (!link.origin || link.origin === '');
  }

  addRoute(pattern, handler, options = {}) {
    this.routes.push({
      pattern: typeof pattern === 'string'
        ? new RegExp('^' + pattern.replace(/:\w+/g, '([^/]+)') + '$')
        : pattern,
      handler,
      options
    });
    return this;
  }

  use(middleware) {
    this.middleware.push(middleware);
    return this;
  }

  beforeEach(hook) {
    this.beforeHooks.push(hook);
    return this;
  }

  afterEach(hook) {
    this.afterHooks.push(hook);
    return this;
  }

  async navigate(path, replace = false) {
    const fullPath = path.startsWith('/') ? path : '/' + path;

    // Run before hooks
    for (const hook of this.beforeHooks) {
      const result = await hook(fullPath);
      if (result === false) return;
    }

    if (replace) {
      history.replaceState({ path: fullPath }, '', fullPath);
    } else {
      history.pushState({ path: fullPath }, '', fullPath);
    }

    await this.resolveRoute(fullPath, { path: fullPath });
  }

  async resolveRoute(path, state) {
    // Run middleware
    const context = { path, state, params: {} };
    for (const mw of this.middleware) {
      await mw(context);
      if (context.cancelled) return;
    }

    // Find matching route
    for (const route of this.routes) {
      const match = path.match(route.pattern);
      if (match) {
        const paramNames = route.pattern.source.match(/:(\w+)/g) || [];
        const params = {};
        match.slice(1).forEach((value, i) => {
          params[paramNames[i]?.slice(1) || i] = value;
        });

        context.params = params;
        context.route = route;

        const html = await route.handler(context);
        this.render(html);

        for (const hook of this.afterHooks) {
          await hook(context);
        }
        return;
      }
    }

    // 404
    this.render('<h1>404 - Page Not Found</h1>');
  }

  render(html) {
    if (typeof html === 'string') {
      this.container.innerHTML = html;
    } else if (html instanceof Element || html instanceof DocumentFragment) {
      this.container.innerHTML = '';
      this.container.appendChild(html);
    }
  }

  getCurrentPath() {
    return window.location.pathname + window.location.search;
  }

  getQueryParams() {
    const params = new URLSearchParams(window.location.search);
    const result = {};
    for (const [key, value] of params) {
      result[key] = value;
    }
    return result;
  }

  navigateBack() {
    window.history.back();
  }

  navigateForward() {
    window.history.forward();
  }
}

// Usage
const router = new AdvancedRouter({ container: document.getElementById('app') });

router.use(async (ctx) => {
  console.log(`Navigating to: ${ctx.path}`);
});

router.beforeEach(async (path) => {
  if (path.startsWith('/admin') && !isAuthenticated()) {
    router.navigate('/login');
    return false;
  }
});

router.addRoute('/', () => '<h1>Home</h1>');
router.addRoute('/users/:id', (ctx) => `<h1>User ${ctx.params.id}</h1>`);
router.addRoute('/products/:category/:productId', (ctx) =>
  `<h1>${ctx.params.productId} in ${ctx.params.category}</h1>`
);

// Example 2: History state compression (for complex state)
class CompressedHistoryState {
  constructor() {
    this.maxSize = 64000; // Most browsers limit state to 64KB
  }

  pushState(state, title, url) {
    // Compress state if needed
    const serialized = JSON.stringify(state);
    if (serialized.length > this.maxSize) {
      state = this.compress(state);
    }
    history.pushState(state, title, url);
  }

  compress(state) {
    // Remove unnecessary fields
    const compressed = {};
    for (const [key, value] of Object.entries(state)) {
      if (value !== null && value !== undefined && value !== '') {
        if (typeof value === 'object' && !Array.isArray(value)) {
          const nested = this.compress(value);
          if (Object.keys(nested).length > 0) {
            compressed[key] = nested;
          }
        } else if (Array.isArray(value)) {
          compressed[key] = value.slice(0, 10); // Limit array size
        } else {
          compressed[key] = value;
        }
      }
    }
    return compressed;
  }
}

// Example 3: History state with scroll position restoration
class ScrollHistory {
  constructor() {
    this.setup();
  }

  setup() {
    // Save scroll position before navigation
    document.addEventListener('click', (e) => {
      const link = e.target.closest('a');
      if (link && link.pathname !== window.location.pathname) {
        this.saveScrollPosition();
      }
    });

    // Restore scroll position on popstate
    window.addEventListener('popstate', (event) => {
      if (event.state?.scrollY !== undefined) {
        requestAnimationFrame(() => {
          window.scrollTo(0, event.state.scrollY);
        });
      }
    });

    // Save on page unload
    window.addEventListener('beforeunload', () => {
      this.saveScrollPosition();
    });
  }

  saveScrollPosition() {
    const currentState = history.state || {};
    currentState.scrollY = window.scrollY;
    history.replaceState(currentState, '', window.location.href);
  }
}

new ScrollHistory();
```

### Real-World Use Cases

- **Single Page Applications**: React Router, Vue Router, Angular Router
- **Tab Navigation**: URL-based tab state preservation
- **Modal State**: Shareable modal URLs
- **Search Filters**: URL-encoded search parameters that survive navigation
- **Form Wizards**: Step state in URL for deep linking
- **Image Galleries**: Image view state in URL for sharing
- **Data Tables**: Sort, filter, page state in URL
- **Content Pagination**: Page number in URL for direct access

### Common Mistakes

```javascript
// Mistake 1: Forgetting to handle popstate
history.pushState({}, '', '/new-url');
// User clicks back - nothing happens because popstate isn't handled

// Mistake 2: State objects that are too large
history.pushState({ hugeData: Array(100000) }, '', '/page');
// 64KB limit on most browsers

// Mistake 3: Not intercepting link clicks
// Regular <a href="/page"> links will cause full page reload
// Must intercept with e.preventDefault() and use pushState

// Mistake 4: Forgetting to handle initial page load
// The History API doesn't fire popstate for the initial page
// Must manually handle the initial route
```

### Best Practices

- Always handle the `popstate` event for back/forward navigation
- Keep state objects small (< 64KB)
- Intercept all internal link clicks for SPA navigation
- Handle initial page load separately from `popstate`
- Use `replaceState` for non-critical navigation (like query param updates)
- Preserve scroll position when navigating with state
- Provide fallback for browsers without History API support

### Performance Considerations

`pushState` and `replaceState` are synchronous and fast. The `popstate` event fires synchronously. State objects are serialized and stored, so large state objects use memory. Each history entry persists until the tab is closed.

### Interview Questions

1. What is the difference between `pushState` and `replaceState`?
2. How does the History API enable SPAs?
3. What is the `popstate` event and when does it fire?
4. What is the data limit for `pushState` state objects?
5. How do you handle the browser back button in an SPA?
6. What happens if you call `pushState` with a cross-origin URL?
7. How does `history.length` work?
8. How do you deep-link to a specific state in an SPA?

### Coding Challenges

1. **SPA Router**: Build a client-side router with nested routes and params
2. **History-Aware Tabs**: Implement tabs that update the URL and survive navigation
3. **Undo/Redo Navigation**: Implement undo/redo using the History API
4. **Scroll Restoration**: Build a system that saves and restores scroll positions with history entries

### Related Topics

- [Location API](https://developer.mozilla.org/en-US/docs/Web/API/Location)
- [URL API](https://developer.mozilla.org/en-US/docs/Web/API/URL)
- [PopStateEvent](https://developer.mozilla.org/en-US/docs/Web/API/PopStateEvent)
- [Single Page Application Architecture](https://developer.mozilla.org/en-US/docs/Glossary/SPA)

## Notification API

### What It Is

The Notification API allows web applications to send system notifications to the user's desktop or mobile device. Notifications appear outside the browser window, similar to native application notifications. They can include titles, text, icons, and action buttons. The API requires user permission before showing notifications.

```javascript
// Request permission
Notification.requestPermission().then(permission => {
  if (permission === 'granted') {
    new Notification('Hello!', {
      body: 'This is a notification',
      icon: '/icon.png'
    });
  }
});
```

### Why It Is Important

Notifications enable web applications to engage users even when they are not on the page. This is essential for messaging apps, calendar reminders, email clients, and any application that needs to alert users about important events. They bridge the gap between web and native app capabilities.

### How It Works Internally

The browser manages notification permissions and display through the operating system's native notification system. When a notification is created, the browser passes the notification data to the OS, which displays it in the system's notification center. Clicking a notification can focus the browser tab that created it, or navigate to a specific URL.

```javascript
// Notification lifecycle:
// 1. Request permission (user must grant)
// 2. Create notification: new Notification(title, options)
// 3. OS displays notification
// 4. User can click (fires 'click' event) or dismiss
// 5. Notification can auto-close (timeout) or remain until dismissed
```

### Syntax

```javascript
// Check support
if ('Notification' in window) {
  // Notifications are supported
}

// Request permission
Notification.requestPermission(); // Returns a promise
Notification.requestPermission(callback); // Legacy callback style

// Permission states
Notification.permission; // 'granted', 'denied', 'default'

// Create notification
const notification = new Notification(title, {
  body: 'Notification body text',
  icon: '/icon.png',        // Image URL
  image: '/image.png',      // Larger image
  badge: '/badge.png',      // Badge icon (mobile)
  tag: 'message-group',     // Group tag replaces existing notifications with same tag
  data: { url: '/messages' }, // Custom data for the notification
  requireInteraction: true, // Stay visible until user interacts
  silent: false,            // No sound
  vibrate: [200, 100, 200], // Vibration pattern (mobile)
  actions: [                // Action buttons
    { action: 'reply', title: 'Reply' },
    { action: 'mark-read', title: 'Mark as Read' }
  ],
  timestamp: Date.now(),
  renotify: true            // Notify even if same tag exists
});

// Notification events
notification.onshow = () => {};
notification.onclick = (event) => {};
notification.onclose = () => {};
notification.onerror = () => {};

// Close notification
notification.close();

// Notification action event (service worker)
// self.addEventListener('notificationclick', (event) => {
//   event.notification.close();
//   if (event.action === 'reply') { /* handle reply */ }
// });
```

### Beginner Examples

```javascript
// Example 1: Basic notification
async function sendNotification(title, body) {
  if (!('Notification' in window)) {
    console.warn('Notifications not supported');
    return;
  }

  const permission = await Notification.requestPermission();
  if (permission === 'granted') {
    new Notification(title, { body });
  }
}

document.querySelector('#notify-btn').addEventListener('click', () => {
  sendNotification('Hello!', 'This is a test notification');
});

// Example 2: Permission request with explanation
async function requestNotificationPermission() {
  if (!('Notification' in window)) {
    alert('Notifications are not supported in your browser');
    return false;
  }

  if (Notification.permission === 'granted') {
    return true;
  }

  if (Notification.permission === 'denied') {
    alert('Notifications are blocked. Please enable them in your browser settings.');
    return false;
  }

  const permission = await Notification.requestPermission();
  return permission === 'granted';
}

// Example 3: Reminder notification
function setReminder(text, delayMs) {
  setTimeout(async () => {
    if (await requestNotificationPermission()) {
      new Notification('Reminder', {
        body: text,
        icon: '/reminder-icon.png',
        tag: 'reminder'
      });
    }
  }, delayMs);
}

setReminder('Time to take a break!', 300000); // 5 minutes

// Example 4: Counting notifications
const notificationCount = new URLSearchParams(window.location.search).get('notifications') || 0;
if (parseInt(notificationCount) > 0) {
  new Notification(`You have ${notificationCount} new messages`, {
    body: 'Check your inbox',
    icon: '/mail-icon.png'
  });
}
```

### Intermediate Examples

```javascript
// Example 1: Notification manager
class NotificationManager {
  constructor() {
    this.notifications = new Map();
    this.defaultIcon = '/icon-192.png';
  }

  async requestPermission() {
    if (!('Notification' in window)) return false;

    if (Notification.permission === 'granted') return true;
    if (Notification.permission === 'denied') return false;

    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }

  async show(title, options = {}) {
    const hasPermission = await this.requestPermission();
    if (!hasPermission) return null;

    const notification = new Notification(title, {
      icon: this.defaultIcon,
      ...options,
      data: { id: Date.now(), ...options.data }
    });

    this.notifications.set(notification.data.id, notification);

    notification.onclick = (event) => {
      this.onClick(event, notification);
    };

    notification.onclose = () => {
      this.notifications.delete(notification.data.id);
    };

    return notification;
  }

  onClick(event, notification) {
    const data = notification.data || {};
    if (data.url) {
      window.focus();
      if (window.location.pathname !== data.url) {
        window.location.href = data.url;
      }
    }
    notification.close();
  }

  close(id) {
    const notification = this.notifications.get(id);
    if (notification) {
      notification.close();
      this.notifications.delete(id);
    }
  }

  closeAll() {
    for (const [id, notification] of this.notifications) {
      notification.close();
      this.notifications.delete(id);
    }
  }

  closeByTag(tag) {
    for (const [id, notification] of this.notifications) {
      if (notification.tag === tag) {
        notification.close();
        this.notifications.delete(id);
      }
    }
  }
}

const notifier = new NotificationManager();
notifier.show('New message', {
  body: 'You have a new message from John',
  tag: 'chat-message',
  data: { url: '/messages/123' },
  requireInteraction: true
});

// Example 2: Scheduled notifications
class NotificationScheduler {
  constructor() {
    this.timers = new Map();
    this.notifier = new NotificationManager();
  }

  schedule(id, title, options, delayMs) {
    this.cancel(id);

    const timer = setTimeout(async () => {
      await this.notifier.show(title, options);
      this.timers.delete(id);
    }, delayMs);

    this.timers.set(id, timer);
  }

  scheduleAt(id, title, options, date) {
    const delay = date.getTime() - Date.now();
    if (delay > 0) {
      this.schedule(id, title, options, delay);
    }
  }

  cancel(id) {
    const timer = this.timers.get(id);
    if (timer) {
      clearTimeout(timer);
      this.timers.delete(id);
    }
  }

  cancelAll() {
    for (const [id] of this.timers) {
      this.cancel(id);
    }
  }
}

const scheduler = new NotificationScheduler();

// Schedule a meeting reminder
const meetingTime = new Date();
meetingTime.setHours(14, 0, 0); // 2:00 PM
scheduler.scheduleAt('meeting', 'Meeting Reminder', {
  body: 'Team standup in 5 minutes',
  tag: 'meeting',
  requireInteraction: true
}, new Date(meetingTime.getTime() - 5 * 60000));

// Example 3: Notification with actions
async function showActionNotification() {
  if (!('Notification' in window)) return;

  const permission = await Notification.requestPermission();
  if (permission !== 'granted') return;

  const notification = new Notification('Incoming Call', {
    body: 'John Doe is calling...',
    icon: '/call-icon.png',
    tag: 'call',
    requireInteraction: true,
    actions: [
      { action: 'answer', title: 'Answer' },
      { action: 'decline', title: 'Decline' },
      { action: 'message', title: 'Message' }
    ]
  });

  notification.onclick = (event) => {
    event.preventDefault(); // Prevent default focus behavior
    window.focus();
    handleAnswerCall();
    notification.close();
  };
}

// For action handling, a Service Worker is needed:
// self.addEventListener('notificationclick', (event) => {
//   event.notification.close();
//   if (event.action === 'answer') {
//     clients.openWindow('/call/answer');
//   } else if (event.action === 'decline') {
//     clients.openWindow('/call/declined');
//   }
// });
```

### Advanced Examples

```javascript
// Example 1: Notification queue with throttling
class NotificationQueue {
  constructor(options = {}) {
    this.maxPerMinute = options.maxPerMinute || 5;
    this.queue = [];
    this.sentTimestamps = [];
    this.notifier = new NotificationManager();
  }

  async enqueue(title, options = {}) {
    this.queue.push({ title, options, timestamp: Date.now() });
    this.process();
  }

  async process() {
    if (this.queue.length === 0) return;

    // Clean old timestamps
    const oneMinuteAgo = Date.now() - 60000;
    this.sentTimestamps = this.sentTimestamps.filter(t => t > oneMinuteAgo);

    if (this.sentTimestamps.length >= this.maxPerMinute) {
      // Too many notifications, wait
      const oldestTimestamp = this.sentTimestamps[0];
      const waitTime = 60000 - (Date.now() - oldestTimestamp);
      setTimeout(() => this.process(), waitTime);
      return;
    }

    const item = this.queue.shift();
    if (item) {
      await this.notifier.show(item.title, item.options);
      this.sentTimestamps.push(Date.now());
      this.process();
    }
  }

  get queueLength() {
    return this.queue.length;
  }

  clear() {
    this.queue = [];
  }
}

const notificationQueue = new NotificationQueue({ maxPerMinute: 3 });

// Batch notifications will be throttled
for (let i = 0; i < 10; i++) {
  notificationQueue.enqueue(`Notification ${i}`, { body: `Body ${i}` });
}

// Example 2: Rich notifications with images
class RichNotification {
  constructor() {
    this.notifier = new NotificationManager();
  }

  async showArticleNotification(article) {
    return this.notifier.show(article.title, {
      body: article.summary,
      image: article.imageUrl,
      icon: article.authorAvatar,
      tag: 'article-' + article.id,
      data: { url: article.url, id: article.id },
      actions: [
        { action: 'read-later', title: 'Read Later' },
        { action: 'share', title: 'Share' }
      ]
    });
  }

  async showProgressNotification(id, title, progress) {
    return this.notifier.show(title, {
      body: `${Math.round(progress * 100)}% complete`,
      tag: `progress-${id}`,
      data: { id }
    });
  }
}

// Example 3: Notification analytics tracker
class NotificationAnalytics {
  constructor() {
    this.events = [];
    this.notifier = new NotificationManager();
    this.setup();
  }

  setup() {
    // Intercept Notification creation to track events
    const originalNotification = window.Notification;

    window.Notification = function (title, options) {
      const id = Date.now() + Math.random();
      const notification = new originalNotification(title, options);

      notification.addEventListener('show', () => {
        this.track('shown', { title, id, tag: options.tag });
      });

      notification.addEventListener('click', () => {
        this.track('clicked', { title, id, tag: options.tag });
      });

      notification.addEventListener('close', () => {
        this.track('closed', { title, id, tag: options.tag });
      });

      return notification;
    };

    window.Notification.permission = originalNotification.permission;
    window.Notification.requestPermission = originalNotification.requestPermission.bind(originalNotification);
  }

  track(type, data) {
    this.events.push({
      type,
      data,
      timestamp: Date.now()
    });

    console.log(`[Notification Analytics] ${type}:`, data);

    // Send to server
    this.sendToAnalytics(type, data);
  }

  async sendToAnalytics(type, data) {
    try {
      await fetch('/api/analytics/notification', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ event: type, data, timestamp: Date.now() }),
        keepalive: true
      });
    } catch {
      // Analytics failure is non-critical
    }
  }

  getStats() {
    const stats = { shown: 0, clicked: 0, closed: 0 };
    for (const event of this.events) {
      stats[event.type] = (stats[event.type] || 0) + 1;
    }
    stats.clickRate = stats.shown > 0 ? (stats.clicked / stats.shown) * 100 : 0;
    return stats;
  }
}

// const analytics = new NotificationAnalytics();

// Example 4: Service worker notification handling
// In service-worker.js:
// self.addEventListener('push', (event) => {
//   const data = event.data.json();
//   const options = {
//     body: data.body,
//     icon: data.icon,
//     badge: '/badge.png',
//     vibrate: [200, 100, 200],
//     data: { url: data.url },
//     actions: [
//       { action: 'view', title: 'View' },
//       { action: 'dismiss', title: 'Dismiss' }
//     ]
//   };
//
//   event.waitUntil(
//     self.registration.showNotification(data.title, options)
//   );
// });
//
// self.addEventListener('notificationclick', (event) => {
//   event.notification.close();
//
//   if (event.action === 'dismiss') return;
//
//   event.waitUntil(
//     clients.matchAll({ type: 'window', includeUncontrolled: true })
//       .then(windowClients => {
//         // Focus existing window or open new one
//         for (const client of windowClients) {
//           if (client.url === event.notification.data.url && 'focus' in client) {
//             return client.focus();
//           }
//         }
//         if (clients.openWindow) {
//           return clients.openWindow(event.notification.data.url);
//         }
//       })
//   );
// });
```

### Real-World Use Cases

- **Messaging Apps**: New message notifications (WhatsApp Web, Slack, Telegram)
- **Email Clients**: New email alerts (Gmail, Outlook)
- **Calendar**: Meeting reminders, event notifications
- **Project Management**: Task assignments, deadline reminders (Asana, Trello)
- **Social Media**: Likes, comments, follower notifications
- **E-commerce**: Order confirmations, shipping updates
- **News**: Breaking news alerts
- **Monitoring**: Server alerts, error notifications
- **Fitness**: Workout reminders, goal achievements
- **Collaboration**: Document edits, comment notifications

### Common Mistakes

```javascript
// Mistake 1: Requesting permission without user gesture
// Must be called from user interaction (click, touch)
button.addEventListener('click', () => {
  Notification.requestPermission(); // OK
});
// Calling on page load: will be blocked

// Mistake 2: Not checking permission before creating notification
if (Notification.permission === 'granted') {
  new Notification('Test'); // OK
}
new Notification('Test'); // Might throw if permission denied

// Mistake 3: Not handling the promise from requestPermission
Notification.requestPermission(); // Returns a promise
// Handle it:
const permission = await Notification.requestPermission();

// Mistake 4: Too many notifications (user will block them)
// Rate-limit notifications and respect user's attention

// Mistake 5: Forgetting about notification data on click
new Notification('Test', { data: { url: '/page' } });
notification.onclick = (event) => {
  // event.target.data.url or event.currentTarget.data.url
  const url = event.currentTarget?.data?.url;
  if (url) window.location.href = url;
};
```

### Best Practices

- Always request permission from a user gesture (click/touch)
- Explain why notifications are needed before requesting
- Respect the user's permission choice (don't re-request if denied)
- Rate-limit notifications to avoid spamming the user
- Use `tag` to replace old notifications with updated ones
- Provide action buttons for common responses
- Use `requireInteraction` sparingly (for important alerts only)
- Handle notification click events to navigate to relevant content
- Use `service workers` for push notifications from the server

### Performance Considerations

Notifications have minimal performance impact since the OS handles display. However, creating too many notifications can overwhelm the user. Use `tag` to consolidate related notifications. The `Notification` constructor is synchronous and lightweight.

### Interview Questions

1. What is the Notification API and when would you use it?
2. How do you request notification permission?
3. What are the possible permission states?
4. How do you handle notification click events?
5. What is the `tag` property used for?
6. How do you add action buttons to a notification?
7. What is `requireInteraction`?
8. How do service workers relate to notifications?
9. What is the difference between `Notification` and `Push API`?
10. How do you close a notification programmatically?

### Coding Challenges

1. **Notification Manager**: Build a notification manager with permission handling and rate limiting
2. **Scheduler with Reminders**: Create a scheduling app that sends notifications at specified times
3. **Notification Queue**: Implement a notification queue with throttling to prevent spam
4. **Analytics Tracker**: Build a notification analytics system that tracks show/click/close rates

### Related Topics

- [Push API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)
- [Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Permissions API](https://developer.mozilla.org/en-US/docs/Web/API/Permissions_API)
- [Web Notifications (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)
