# HTTP Requests - requests.get/post, sessions, timeouts, auth

## requests.get() and requests.post()

### What It Is
`requests.get()` and `requests.post()` are the primary functions in the `requests` library for making HTTP requests. `get()` retrieves data from a URL (HTTP GET method) and is used for read-only operations. `post()` submits data to a URL (HTTP POST method) and is used for creating or modifying resources. Both return a `Response` object encapsulating the server's status code, headers, and body.

### Why It Is Important
These two functions cover the vast majority of HTTP interactions in Python development — from consuming REST APIs and scraping web pages to submitting forms and uploading files. Their ergonomic API reduces the boilerplate that comes with Python's built-in `urllib` library, making HTTP code concise, readable, and maintainable. Understanding when and how to use each method is fundamental to building web clients.

### How It Works Internally
When `requests.get(url, **kwargs)` is called, the library creates an internal `Request` object from the parameters, then prepares it into a `PreparedRequest` containing the final URL, headers, query parameters, and body. This prepared request is dispatched through a transport adapter (defaulting to `urllib3`'s `HTTPAdapter`) which opens a TCP connection to the host, sends the HTTP request, and reads the response. The raw HTTP response is parsed and wrapped in a `Response` object with properties like `status_code`, `headers`, `text`, `content`, and convenience methods like `json()`. For POST requests, the data payload is encoded according to the `data`, `json`, `files`, or `params` arguments before being sent.

### Syntax
```python
import requests

# GET request
response = requests.get("https://api.github.com/users/octocat")
response = requests.get("https://api.example.com/search", params={"q": "python", "page": 1})

# POST request with JSON
response = requests.post("https://api.example.com/users", json={"name": "Alice", "role": "admin"})

# POST with form data
response = requests.post("https://httpbin.org/post", data={"username": "alice", "password": "s3cret"})

# POST with files
response = requests.post("https://httpbin.org/post", files={"file": open("report.pdf", "rb")})
```

### Beginner Examples
```python
import requests

response = requests.get("https://api.github.com")
print(response.status_code)        # 200
print(response.headers["content-type"])
data = response.json()
print(data["current_user_url"])

response = requests.post(
    "https://jsonplaceholder.typicode.com/posts",
    json={"title": "Hello", "body": "World", "userId": 1}
)
print(response.status_code)        # 201
print(response.json()["id"])       # 101
```

### Intermediate Examples
```python
import requests

response = requests.get(
    "https://api.github.com/search/repositories",
    params={"q": "requests library", "sort": "stars"},
    headers={"Accept": "application/vnd.github.v3+json"}
)
for repo in response.json()["items"][:5]:
    print(f"{repo['name']}: {repo['stargazers_count']} stars")

response = requests.post(
    "https://api.example.com/login",
    data={"username": "admin", "password": "pass123"},
    headers={"X-CSRF-Token": "abc123"},
    cookies={"session": "prev_session_id"}
)
print(response.history)  # Redirect chain if any
print(response.elapsed)  # Total request duration
```

### Advanced Examples
```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retries = Retry(total=3, backoff_factor=0.5, status_forcelist=[502, 503, 504])
session.mount("https://", HTTPAdapter(max_retries=retries))

def fetch_with_retry(url):
    response = session.get(url, timeout=10)
    response.raise_for_status()
    return response.json()

import concurrent.futures
urls = [f"https://jsonplaceholder.typicode.com/posts/{i}" for i in range(1, 51)]
with concurrent.futures.ThreadPoolExecutor(max_workers=10) as pool:
    results = list(pool.map(fetch_with_retry, urls))
print(f"Fetched {len(results)} posts")
```

### Real-World Use Cases
- Consuming REST APIs (GitHub, Stripe, OpenAI, Twilio).
- Submitting HTML forms in web scraping workflows.
- Uploading files to cloud storage (S3 presigned URLs, file hosting services).
- Triggering CI/CD pipelines via webhooks.
- Health-check polling for monitoring systems.

### Common Mistakes
```python
# Not checking status code
response = requests.get("https://api.example.com/users/99999")
# response body is "Not Found", but code proceeds

# Forgetting to close file handles in uploads
with open("data.txt", "rb") as f:
    requests.post("https://httpbin.org/post", files={"file": f})
# File auto-closes via context manager; using bare open() leaks handles

# Using data= instead of json= for dict payloads
# data= sends form-encoded; json= sends JSON with correct Content-Type

# Calling .json() on non-JSON response
response = requests.get("https://example.com")
data = response.json()  # raises ValueError
```

### Best Practices
- Always check `response.ok` or call `response.raise_for_status()`.
- Use the `json=` parameter instead of manually serializing with `json.dumps()`.
- Set a `User-Agent` header to identify your client.
- Close response content for large responses with `response.close()`.
- Use context managers (`with requests.get(...) as response`) when possible.

### Performance Considerations
- `json=` parameter is faster than `data=json.dumps()` because it sets the correct `Content-Type` header automatically.
- Streaming large responses with `stream=True` and `iter_content()` avoids loading the entire response into memory.
- Connection overhead dominates single requests; use Sessions for multiple requests.
- Thread pool parallelism greatly increases throughput for IO-bound workloads.

### Interview Questions
1. What is the difference between GET and POST in terms of idempotency and safety?
2. How does the `params` argument differ from including query string directly in the URL?
3. What happens when you pass both `data` and `json` to `requests.post()`?
4. How do you handle redirects with `requests`?
5. What does `raise_for_status()` do?

### Coding Challenges
```python
# Challenge: Build a paginated API client that handles cursor-based pagination
def fetch_all_pages(base_url, params=None):
    results = []
    params = params or {}
    while base_url:
        response = requests.get(base_url, params=params)
        response.raise_for_status()
        data = response.json()
        results.extend(data["items"])
        base_url = data.get("next_url")
    return results
```

### Related Topics
- HTTP Methods and Status Codes
- Sessions
- Timeouts
- Authentication

---

## Sessions

### What It Is
A `requests.Session()` object provides persistent settings across multiple HTTP requests. It maintains a pool of TCP connections (via `urllib3`), stores cookies, and holds default headers and authentication credentials. Sessions are the recommended way to make multiple requests to the same host.

### Why It Is Important
Without sessions, each request opens and closes a new TCP connection. For sequential requests to the same server, session reuse reduces latency by 2-10x through connection pooling. Sessions also automatically persist cookies set by the server, which is essential for login workflows, session-based auth, and CSRF token handling. They provide a single point to configure defaults (headers, auth, timeouts) for all outgoing requests.

### How It Works Internally
When a `Session` is created, it initializes an internal `urllib3.PoolManager` with configurable connection pools per host. Each `mount()` call associates a URL prefix with an adapter (default `HTTPAdapter`). When `session.get()` is called, the Session merges its own headers/cookies/auth with per-request overrides, prepares the request, selects the matching adapter based on the URL prefix, and dispatches through the adapter's connection pool. Responses pass back through the Session, which extracts and stores `Set-Cookie` headers for subsequent requests. The Session object maintains a `cookies` dict and a `headers` dict that persist across all requests made through it.

### Syntax
```python
import requests

session = requests.Session()

# Set defaults
session.headers.update({"User-Agent": "MyApp/1.0"})
session.auth = ("user", "pass")

# All requests share these defaults
response1 = session.get("https://api.example.com/users")
response2 = session.post("https://api.example.com/users", json={"name": "Bob"})

# Cookies persist automatically
session.get("https://httpbin.org/cookies/set/name/alice")
response = session.get("https://httpbin.org/cookies")
print(response.json())  # {"cookies": {"name": "alice"}}
```

### Beginner Examples
```python
import requests

session = requests.Session()
session.headers["Accept"] = "application/json"

login_data = {"username": "alice", "password": "hunter2"}
session.post("https://example.com/login", data=login_data)

profile = session.get("https://example.com/profile")
dashboard = session.get("https://example.com/dashboard")

print(f"Profile status: {profile.status_code}")
print(f"Dashboard status: {dashboard.status_code}")
```

### Intermediate Examples
```python
import requests
from requests.adapters import HTTPAdapter

session = requests.Session()

adapter = HTTPAdapter(
    pool_connections=10,
    pool_maxsize=100,
    max_retries=3
)
session.mount("https://", adapter)
session.mount("http://", adapter)

class BaseURLSession(requests.Session):
    def __init__(self, base_url=None):
        super().__init__()
        self.base_url = base_url

    def request(self, method, url, *args, **kwargs):
        if self.base_url and not url.startswith("http"):
            url = self.base_url.rstrip("/") + "/" + url.lstrip("/")
        return super().request(method, url, *args, **kwargs)

api = BaseURLSession("https://api.github.com")
api.headers.update({"Accept": "application/vnd.github.v3+json"})
user = api.get("/users/octocat").json()
print(user["login"])
```

### Advanced Examples
```python
import requests
from requests.adapters import HTTPAdapter
import time

session = requests.Session()

session.mount(
    "https://api.github.com",
    HTTPAdapter(pool_connections=5, pool_maxsize=20)
)
session.mount(
    "https://httpbin.org",
    HTTPAdapter(pool_connections=2, pool_maxsize=10)
)

# Rate-limited session wrapper
class RateLimitedSession:
    def __init__(self, requests_per_second=10):
        self.session = requests.Session()
        self.min_interval = 1.0 / requests_per_second
        self._last_request = 0.0

    def request(self, method, url, **kwargs):
        elapsed = time.time() - self._last_request
        if elapsed < self.min_interval:
            time.sleep(self.min_interval - elapsed)
        self._last_request = time.time()
        return self.session.request(method, url, **kwargs)

    def get(self, url, **kwargs):
        return self.request("GET", url, **kwargs)

rate_limited = RateLimitedSession(requests_per_second=5)
for i in range(20):
    resp = rate_limited.get(f"https://jsonplaceholder.typicode.com/posts/{i + 1}")
    print(f"Post {i + 1}: {resp.status_code}")
```

### Real-World Use Cases
- **API clients**: SDKs for GitHub, Stripe, Twilio use sessions internally to maintain auth and connection pools.
- **Web scraping login flows**: Login once, reuse authenticated session for subsequent pages.
- **Microservice gateways**: Internal services use sessions with mutual TLS and custom headers.
- **Batch processing jobs**: Thousands of requests to the same API benefit from connection reuse.

### Common Mistakes
```python
# Mistake: Creating a new session per request
for i in range(100):
    session = requests.Session()  # Wrong: defeats pooling
    session.get(f"https://api.example.com/item/{i}")

# Mistake: Not closing a session
session = requests.Session()
# ... use it
# session.close()  # Frees connection pool resources

# Mistake: Modifying session.headers after requests (race conditions in threads)
# Use a lock or create per-thread sessions

# Mistake: Sharing a session between unrelated API hosts
# Mount different adapters for different hosts
```

### Best Practices
- Reuse a single session for all requests to the same host.
- Close sessions when done (`session.close()` or use context manager `with requests.Session() as s:`).
- Configure adapters per host for fine-grained pooling control.
- Do not mutate session state (headers, cookies) concurrently.
- Use separate sessions for different API credentials.

### Performance Considerations
- Connection pooling eliminates TCP handshake and TLS negotiation overhead.
- Default pool size is 10 connections per host; increase with `HTTPAdapter(pool_maxsize=100)` for high throughput.
- Sessions are not thread-safe for mutation; use one session per thread or use locks.
- For very high throughput, consider `urllib3.PoolManager` directly or async clients like `httpx`.

### Interview Questions
1. How does a `Session` improve performance over individual requests?
2. Are `requests.Session` objects thread-safe?
3. How do you mount different adapters for different hosts?
4. What happens to cookies set by the server in a session?
5. How do you override a session-level default header for a single request?

### Coding Challenges
```python
# Challenge: Build a session pool manager
from queue import Queue
import threading

class SessionPool:
    def __init__(self, size=5):
        self._pool = Queue(maxsize=size)
        for _ in range(size):
            session = requests.Session()
            session.headers.update({"User-Agent": "PooledClient/1.0"})
            self._pool.put(session)

    def get_session(self):
        return self._pool.get()

    def return_session(self, session):
        session.cookies.clear()
        self._pool.put(session)

# pool = SessionPool(10)
# sess = pool.get_session()
# try:
#     resp = sess.get("https://api.example.com")
# finally:
#     pool.return_session(sess)
```

### Related Topics
- Connection Pooling
- HTTPAdapter
- Cookie Persistence
- requests.get() and requests.post()

---

## Timeouts

### What It Is
Timeouts are limits on how long `requests` will wait for a server response before raising an exception. `requests` supports two separate timeout values: the connection timeout (time to establish the TCP connection) and the read timeout (time to receive data once connected). Timeouts can be set as a single float (applied to both phases) or a tuple of two floats.

### Why It Is Important
Without timeouts, a request can block indefinitely if the server is unreachable, overloaded, or suffering from network issues. In production, hung requests consume threads, file descriptors, and memory, leading to cascading failures and resource exhaustion. Timeouts are an essential guardrail for building robust, resilient distributed systems.

### How It Works Internally
When `timeout=(3.05, 10)` is passed, `requests` splits this into two values. The connection timeout (3.05s) is passed to `urllib3`, which sets `socket.settimeout()` on the underlying socket during the TCP connect phase. If the connect call does not complete within this window, `urllib3` raises `ConnectTimeoutError`, which `requests` wraps as `ConnectTimeout`. After connection, the read timeout (10s) is set on the socket for each `recv()` call. If the server sends no data within this window, `urllib3` raises `ReadTimeoutError`, wrapped as `ReadTimeout`. With a single float `timeout=5`, both connect and read share the same value.

### Syntax
```python
import requests

# Same timeout for connect and read
response = requests.get("https://api.example.com", timeout=5)

# Separate connect and read timeouts (recommended)
response = requests.get("https://api.example.com", timeout=(3.05, 10))

# No timeout (block indefinitely — avoid in production)
response = requests.get("https://api.example.com", timeout=None)
```

### Beginner Examples
```python
import requests

try:
    response = requests.get("https://httpbin.org/delay/5", timeout=3)
except requests.exceptions.Timeout:
    print("Request timed out after 3 seconds")

try:
    response = requests.get(
        "https://httpbin.org/delay/2",
        timeout=(1, 3)  # 1s connect, 3s read
    )
except requests.exceptions.ConnectTimeout:
    print("Connection timed out")
except requests.exceptions.ReadTimeout:
    print("Read timed out")
```

### Intermediate Examples
```python
import requests
from requests.adapters import HTTPAdapter

session = requests.Session()

adapter = HTTPAdapter(max_retries=2)
session.mount("https://", adapter)
session.mount("http://", adapter)

urls = [
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/5",
    "https://httpbin.org/delay/2",
]

for url in urls:
    try:
        response = session.get(url, timeout=(2, 4))
        print(f"OK: {url} -> {response.elapsed.total_seconds():.2f}s")
    except requests.exceptions.Timeout:
        print(f"TIMEOUT: {url}")

# Retry with backoff on timeout
from time import sleep

def request_with_retry(url, timeout, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=timeout)
            response.raise_for_status()
            return response
        except requests.exceptions.Timeout:
            if attempt == max_retries - 1:
                raise
            sleep(2 ** attempt)  # exponential backoff
```

### Advanced Examples
```python
import requests
import socket
from requests.adapters import HTTPAdapter

# Custom adapter with per-host timeouts
class TimeoutHTTPAdapter(HTTPAdapter):
    def __init__(self, default_timeout=None, *args, **kwargs):
        self.default_timeout = default_timeout
        super().__init__(*args, **kwargs)

    def send(self, request, **kwargs):
        kwargs.setdefault("timeout", self.default_timeout)
        return super().send(request, **kwargs)

session = requests.Session()
adapter = TimeoutHTTPAdapter(default_timeout=(3, 15))
session.mount("https://", adapter)
session.mount("http://", adapter)

response = session.get("https://api.example.com")
# Uses default timeout (3, 15) unless overridden
response = session.get("https://api.example.com", timeout=30)
# Per-request timeout overrides the adapter default

# Global timeout via session property override
class TimedSession(requests.Session):
    def __init__(self, connect_timeout=3, read_timeout=10):
        super().__init__()
        self.connect_timeout = connect_timeout
        self.read_timeout = read_timeout

    def request(self, method, url, **kwargs):
        kwargs.setdefault("timeout", (self.connect_timeout, self.read_timeout))
        return super().request(method, url, **kwargs)

timed = TimedSession(connect_timeout=2, read_timeout=5)
timed.get("https://httpbin.org/delay/3")  # Uses (2, 5)
```

### Real-World Use Cases
- **Microservice health checks**: Short timeouts (1s connect, 2s read) to detect unhealthy services.
- **External API calls**: Moderate timeouts (5s connect, 30s read) for third-party SaaS APIs.
- **File upload/download**: Longer read timeouts (60-300s) for large payloads.
- **Batch processing pipelines**: Configurable timeouts per task to prevent single slow request from blocking the batch.

### Common Mistakes
```python
# Mistake: No timeout at all
requests.get("https://example.com")  # Could hang forever

# Mistake: Too short a timeout for realistic conditions
requests.get("https://slow-api.example.com", timeout=0.5)
# Times out even when server is working fine

# Mistake: Forgetting that the tuple order is (connect, read), not (read, connect)

# Mistake: Assuming timeout covers the entire request (DNS + connect + read + SSL handshake)
# Timeout only covers connect and read phases, not DNS resolution
```

### Best Practices
- Always set timeouts; never use `timeout=None` in production.
- Use separate connect and read timeouts: short connect (2-5s) detects unreachable hosts quickly; longer read (10-30s) accommodates slow processing.
- Set higher timeouts for uploads/large downloads.
- Combine timeouts with retry logic for transient failures.
- Monitor timeout exception rates to detect upstream service degradation.
- Adjust timeouts based on the service's documented SLA.

### Performance Considerations
- Shorter timeouts free up threads and connections faster during outages.
- Connection timeout should account for worst-case network latency (including TLS handshake overhead).
- Read timeout should account for the slowest legitimate response time plus network latency.
- Aggressive timeouts (too short) cause false positives and unnecessary retries.
- Each timed-out request still consumed resources (connection attempt, partial response).

### Interview Questions
1. What is the difference between connect timeout and read timeout?
2. How does `requests` handle timeouts at the socket level?
3. Why is it important to set timeouts even for localhost requests?
4. How would you implement a per-host default timeout?
5. What happens to the underlying socket when a timeout occurs?

### Coding Challenges
```python
# Challenge: Adaptive timeout system
import time
import statistics

class AdaptiveTimeout:
    def __init__(self, initial=5, min_timeout=1, max_timeout=30, multiplier=1.5):
        self.current = initial
        self.min_timeout = min_timeout
        self.max_timeout = max_timeout
        self.multiplier = multiplier
        self._history = []

    def measure(self, url):
        start = time.time()
        try:
            response = requests.get(url, timeout=self.current)
            elapsed = time.time() - start
            self._history.append(elapsed)
            if len(self._history) > 20:
                self._history.pop(0)
            avg = statistics.mean(self._history)
            std = statistics.stdev(self._history) if len(self._history) > 1 else 1
            self.current = min(max(avg + 3 * std, self.min_timeout), self.max_timeout)
            return response
        except requests.exceptions.Timeout:
            self.current = min(self.current * self.multiplier, self.max_timeout)
            raise

# adaptive = AdaptiveTimeout()
# try:
#     resp = adaptive.measure("https://httpbin.org/delay/3")
# except requests.exceptions.Timeout:
#     print(f"Timed out at {adaptive.current}s")
```

### Related Topics
- Retry Strategies
- Exponential Backoff
- Connection Pooling
- Error Handling

---

## Authentication

### What It Is
Authentication verifies the identity of the client making an HTTP request. The `requests` library supports multiple authentication mechanisms out of the box: HTTP Basic Auth, Digest Auth, Bearer tokens, and custom authentication via pluggable `AuthBase` subclasses. Authentication can be set per-request via the `auth` parameter or configured globally on a `Session`.

### Why It Is Important
Most production APIs require authentication to control access, enforce rate limits, and track usage. Without proper auth handling, requests to protected endpoints return `401 Unauthorized` or `403 Forbidden`. The `requests` library abstracts away the complexities of different auth schemes behind a unified `auth` parameter, making it straightforward to switch between methods or implement custom flows like HMAC signing or OAuth2.

### How It Works Internally
The `auth` parameter accepts a callable (like `HTTPBasicAuth`) or a tuple `(username, password)`. When a request is made, `requests` invokes `auth(request)` on the `PreparedRequest` before sending it. For `HTTPBasicAuth`, this callable base64-encodes `username:password` and sets an `Authorization: Basic <encoded>` header. For `HTTPDigestAuth`, it performs the MD5-based challenge-response handshake: it sends an unauthenticated request first, reads the `WWW-Authenticate` header for nonce/realm details, then resends with the computed digest. Custom `AuthBase` subclasses can implement arbitrary auth logic by overriding `__call__(self, request)` and modifying request headers, body, or URL.

### Syntax
```python
import requests
from requests.auth import HTTPBasicAuth, HTTPDigestAuth

# Basic auth (tuple form)
response = requests.get("https://api.example.com/secure", auth=("admin", "secret"))

# Basic auth (explicit class)
response = requests.get("https://api.example.com/secure", auth=HTTPBasicAuth("admin", "secret"))

# Digest auth
response = requests.get("https://api.example.com/digest", auth=HTTPDigestAuth("admin", "secret"))

# Bearer token via headers
response = requests.get(
    "https://api.example.com/protected",
    headers={"Authorization": "Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0"}
)

# Bearer token via auth (using requests >= 2.27.0)
from requests.auth import HTTPBasicAuth  # Note: no built-in Bearer class; use headers or custom

# Custom auth class
from requests.auth import AuthBase

class BearerAuth(AuthBase):
    def __init__(self, token):
        self.token = token

    def __call__(self, request):
        request.headers["Authorization"] = f"Bearer {self.token}"
        return request

response = requests.get("https://api.example.com/protected", auth=BearerAuth("mytoken"))
```

### Beginner Examples
```python
import requests

response = requests.get(
    "https://api.github.com/user",
    auth=("your_username", "your_token")
)
print(response.status_code)
if response.ok:
    print(response.json()["login"])

response = requests.get(
    "https://httpbin.org/basic-auth/user/pass",
    auth=("user", "pass")
)
print(response.json())  # {"authenticated": true, "user": "user"}
```

### Intermediate Examples
```python
import requests

session = requests.Session()
session.auth = ("admin", "s3cret")

response1 = session.get("https://httpbin.org/basic-auth/admin/s3cret")
print(response1.json())

response2 = session.get("https://httpbin.org/basic-auth/admin/s3cret")
# Auth is reused; no need to pass auth again

# Per-request override
response3 = session.get(
    "https://httpbin.org/basic-auth/other/different",
    auth=("other", "different")
)

# Digest authentication
from requests.auth import HTTPDigestAuth

response = requests.get(
    "https://httpbin.org/digest-auth/auth/user/pass",
    auth=HTTPDigestAuth("user", "pass")
)
print(response.status_code)  # 200
```

### Advanced Examples
```python
import requests
from requests.auth import AuthBase
import hashlib
import hmac
import time
import base64

class HMACAuth(AuthBase):
    def __init__(self, api_key, api_secret):
        self.api_key = api_key
        self.api_secret = api_secret

    def __call__(self, request):
        timestamp = str(int(time.time()))
        method = request.method
        path = request.path_url
        body = request.body or b""
        if isinstance(body, str):
            body = body.encode()
        body_hash = hashlib.sha256(body).hexdigest()

        message = f"{method}\n{path}\n{timestamp}\n{body_hash}"
        signature = hmac.new(
            self.api_secret.encode(),
            message.encode(),
            hashlib.sha256
        ).hexdigest()

        request.headers["X-API-Key"] = self.api_key
        request.headers["X-Timestamp"] = timestamp
        request.headers["X-Signature"] = signature
        return request

session = requests.Session()
session.auth = HMACAuth("ak_12345", "sk_abcdef")
response = session.post(
    "https://api.example.com/orders",
    json={"product": "widget", "quantity": 5}
)
print(response.status_code)

# OAuth2 client credentials flow with requests-oauthlib
from requests_oauthlib import OAuth2Session

client_id = "your_client_id"
client_secret = "your_client_secret"
token_url = "https://provider.com/oauth/token"

oauth = OAuth2Session(client_id)
token = oauth.fetch_token(
    token_url=token_url,
    client_secret=client_secret,
    grant_type="client_credentials"
)

response = oauth.get("https://api.provider.com/v1/users")
print(response.json())
```

### Real-World Use Cases
- **REST API clients**: GitHub (Basic or Bearer), Stripe (Bearer with secret key), OpenAI (Bearer).
- **Microservice mesh**: Service-to-service auth using JWT Bearer tokens with mutual TLS.
- **Legacy enterprise systems**: Still use HTTP Basic Auth or Digest Auth.
- **Financial APIs**: Often require HMAC-signed requests for non-repudiation.
- **OAuth2 flows**: Social login, delegated authorization (Google, Facebook, GitHub OAuth).

### Common Mistakes
```python
# Mistake: Hardcoding credentials in source code
auth = ("admin", "p@ssw0rd")  # Don't! Use env vars

# Mistake: Storing tokens insecurely
token = "ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"  # Leaked if committed

# Mistake: Passing auth as a tuple to Digest auth
# HTTPDigestAuth needs the explicit class, not a tuple

# Mistake: Using Basic Auth over plain HTTP (no TLS)
# Credentials are base64-encoded, not encrypted

# Mistake: Forgetting to refresh expired tokens
# OAuth2 access tokens expire; handle 401 by re-authenticating
```

### Best Practices
- Never hardcode credentials; use environment variables or a secrets manager.
- Always use HTTPS with authentication to prevent credential sniffing.
- Prefer Bearer tokens over Basic Auth for API authentication.
- Implement token refresh logic with automatic retry on 401.
- Use sessions to reuse auth across multiple requests.
- Log authentication failures but never log the credentials themselves.
- Rotate API keys and tokens regularly.

### Performance Considerations
- Basic Auth adds minimal overhead (one base64 encoding per request).
- Digest Auth requires two round trips per request (challenge + response).
- HMAC signing adds CPU overhead proportional to body size.
- Token-based auth (Bearer) is faster than challenge-response schemes.
- OAuth2 token refresh adds latency; batch refresh before expiry.
- Pre-compute signatures when possible to reduce per-request CPU cost.

### Interview Questions
1. What is the difference between Basic Auth and Digest Auth?
2. How would you implement Bearer token authentication with auto-refresh?
3. How does the `auth` parameter work internally — what does it expect?
4. What security considerations apply when using Basic Auth over HTTP?
5. How would you implement HMAC-based request signing?

### Coding Challenges
```python
# Challenge: Auto-refreshing auth for OAuth2 tokens
import time

class AutoRefreshAuth(AuthBase):
    def __init__(self, get_token_func, refresh_threshold=60):
        self.get_token = get_token_func
        self.refresh_threshold = refresh_threshold
        self._token = None
        self._expires_at = 0

    def _ensure_token(self):
        if time.time() > self._expires_at - self.refresh_threshold:
            self._token = self.get_token()
            self._expires_at = time.time() + 3600
        return self._token

    def __call__(self, request):
        token = self._ensure_token()
        request.headers["Authorization"] = f"Bearer {token}"
        return request

# Mock token provider
def fetch_token():
    print("Fetching new token...")
    return "new_access_token_xyz"

session = requests.Session()
session.auth = AutoRefreshAuth(fetch_token, refresh_threshold=120)
session.get("https://api.example.com/data")
# Token fetched on first request; re-fetched when within 120s of expiry
```

### Related Topics
- OAuth2
- JWT
- HMAC Signing
- Environment Variables and Secrets Management
- HTTPS and TLS
