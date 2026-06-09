# HTTP Requests - requests.get/post, sessions, timeouts, auth

## Introduction

Hypertext Transfer Protocol (HTTP) requests are the foundation of data communication on the World Wide Web. In Python, developers can send HTTP requests to interact with web servers, APIs, and web services using built-in libraries like `urllib.request` or third-party libraries like `requests`. The `requests` library is the most popular choice due to its simpler, more intuitive API. HTTP requests enable fetching web pages, consuming REST APIs, submitting form data, uploading files, and much more.

## Why It Is Important

Understanding HTTP requests is fundamental for modern Python development. Almost every application communicates over the web — whether it's a data science script pulling from an API, a web application making backend calls, or a microservice architecture. Mastery of HTTP requests allows developers to integrate third-party services, automate web interactions, scrape data, build API clients, and create robust networked applications. Proper handling of headers, timeouts, retries, and authentication ensures reliability, security, and performance.

## Syntax

### Using `urllib.request` (built-in)

```python
import urllib.request
import urllib.parse

# GET request
response = urllib.request.urlopen("https://api.example.com/data")
data = response.read().decode("utf-8")

# POST request with data
data = urllib.parse.urlencode({"key": "value"}).encode("utf-8")
req = urllib.request.Request("https://api.example.com/submit", data=data, method="POST")
response = urllib.request.urlopen(req)
```

### Using `requests` library (third-party)

```python
import requests

# GET request
response = requests.get("https://api.example.com/data")

# POST request
response = requests.post("https://api.example.com/submit", json={"key": "value"})

# PUT request
response = requests.put("https://api.example.com/update/1", json={"key": "new_value"})

# DELETE request
response = requests.delete("https://api.example.com/delete/1")
```

## Examples

### Basic GET Request

```python
import requests

response = requests.get("https://jsonplaceholder.typicode.com/posts/1")
print(response.status_code)
print(response.json())
```

### GET with Query Parameters

```python
import requests

params = {"userId": 1, "completed": True}
response = requests.get("https://jsonplaceholder.typicode.com/todos", params=params)
print(response.url)
print(response.json())
```

### POST with JSON Data

```python
import requests

payload = {"title": "foo", "body": "bar", "userId": 1}
response = requests.post("https://jsonplaceholder.typicode.com/posts", json=payload)
print(response.status_code)
print(response.json())
```

### PUT Request

```python
import requests

payload = {"id": 1, "title": "updated", "body": "updated body", "userId": 1}
response = requests.put("https://jsonplaceholder.typicode.com/posts/1", json=payload)
print(response.status_code)
print(response.json())
```

### DELETE Request

```python
import requests

response = requests.delete("https://jsonplaceholder.typicode.com/posts/1")
print(response.status_code)
```

## Beginner Examples

### 1. Fetch a Web Page

```python
import requests

url = "https://example.com"
response = requests.get(url)

if response.status_code == 200:
    print("Page fetched successfully!")
    print(response.text[:500])
else:
    print(f"Failed with status code: {response.status_code}")
```

### 2. Check Response Status

```python
import requests

response = requests.get("https://httpbin.org/status/200")
print(f"Status: {response.status_code}")
print(f"OK: {response.ok}")

response = requests.get("https://httpbin.org/status/404")
print(f"Status: {response.status_code}")
print(f"OK: {response.ok}")
```

### 3. Send Custom Headers

```python
import requests

headers = {
    "User-Agent": "MyApp/1.0",
    "Accept": "application/json",
    "Authorization": "Bearer my_token_123"
}

response = requests.get("https://httpbin.org/headers", headers=headers)
print(response.json())
```

### 4. Download and Save a File

```python
import requests

url = "https://httpbin.org/image/png"
response = requests.get(url)

if response.status_code == 200:
    with open("image.png", "wb") as f:
        f.write(response.content)
    print("Image downloaded successfully!")
else:
    print("Download failed")
```

### 5. Handle Query Strings

```python
import requests

base_url = "https://api.example.com/search"
params = {
    "q": "python requests",
    "page": 1,
    "limit": 10,
    "sort": "relevance"
}

response = requests.get(base_url, params=params)
print(f"Request URL: {response.url}")
print(f"Response: {response.status_code}")
```

## Intermediate Examples

### 1. Session with Persistent Headers

```python
import requests

session = requests.Session()
session.headers.update({"User-Agent": "MyApp/1.0", "Accept": "application/json"})

# First request
response1 = session.get("https://httpbin.org/headers")
print("First request:", response1.json())

# Second request (same session, same headers)
response2 = session.get("https://httpbin.org/anything")
print("Second request:", response2.json())

session.close()
```

### 2. Timeout Handling

```python
import requests
from requests.exceptions import Timeout, ConnectionError

try:
    response = requests.get(
        "https://httpbin.org/delay/5",
        timeout=3
    )
    print(response.json())
except Timeout:
    print("Request timed out!")
except ConnectionError:
    print("Connection error occurred!")
```

### 3. Retry on Failure

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()

retry_strategy = Retry(
    total=3,
    backoff_factor=1,
    status_forcelist=[500, 502, 503, 504],
    allowed_methods=["GET", "POST"]
)

adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("https://", adapter)
session.mount("http://", adapter)

try:
    response = session.get("https://httpbin.org/status/503", timeout=10)
    print(f"Status: {response.status_code}")
except Exception as e:
    print(f"Failed after retries: {e}")
finally:
    session.close()
```

### 4. Authentication (Basic Auth)

```python
import requests
from requests.auth import HTTPBasicAuth

response = requests.get(
    "https://httpbin.org/basic-auth/user/pass",
    auth=HTTPBasicAuth("user", "pass")
)
print(f"Status: {response.status_code}")
print(f"Authenticated: {response.json().get('authenticated')}")
```

### 5. Proxy Configuration

```python
import requests

proxies = {
    "http": "http://10.10.1.10:3128",
    "https": "http://10.10.1.10:1080"
}

try:
    response = requests.get(
        "https://httpbin.org/ip",
        proxies=proxies,
        timeout=10
    )
    print(response.json())
except Exception as e:
    print(f"Proxy error: {e}")
```

### 6. Upload Files

```python
import requests

url = "https://httpbin.org/post"
files = {
    "file": ("report.txt", b"Hello, this is file content!", "text/plain")
}

response = requests.post(url, files=files)
print(response.json())
```

### 7. Cookies Handling

```python
import requests

# Get cookies
response = requests.get("https://httpbin.org/cookies/set?name=value")
print("Cookies:", dict(response.cookies))

# Send cookies
cookies = {"session_id": "abc123"}
response = requests.get("https://httpbin.org/cookies", cookies=cookies)
print(response.json())
```

### 8. Streaming Response

```python
import requests

url = "https://httpbin.org/stream/10"
response = requests.get(url, stream=True)

for line in response.iter_lines():
    if line:
        print(line.decode("utf-8"))
```

### 9. Handle Redirects

```python
import requests

# Follow redirects (default)
response = requests.get("https://httpbin.org/redirect/3", allow_redirects=True)
print(f"Final URL: {response.url}")
print(f"Redirect history: {[r.url for r in response.history]}")

# Don't follow redirects
response = requests.get("https://httpbin.org/redirect/3", allow_redirects=False)
print(f"Status: {response.status_code} (Location: {response.headers.get('Location')})")
```

### 10. Send Form Data

```python
import requests

form_data = {
    "username": "john_doe",
    "password": "secret123",
    "remember_me": True
}

response = requests.post("https://httpbin.org/post", data=form_data)
print(response.json())
```

## Advanced Examples

### 1. Custom Retry with Exponential Backoff

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry
import time

class RetrySession:
    def __init__(self, retries=3, backoff_factor=0.5, status_forcelist=None):
        self.session = requests.Session()
        if status_forcelist is None:
            status_forcelist = [429, 500, 502, 503, 504]

        retry_strategy = Retry(
            total=retries,
            backoff_factor=backoff_factor,
            status_forcelist=status_forcelist,
            allowed_methods=["GET", "POST", "PUT", "DELETE"],
            raise_on_status=False
        )

        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("https://", adapter)
        self.session.mount("http://", adapter)

    def get(self, url, **kwargs):
        return self.session.get(url, **kwargs)

    def post(self, url, **kwargs):
        return self.session.post(url, **kwargs)

    def put(self, url, **kwargs):
        return self.session.put(url, **kwargs)

    def delete(self, url, **kwargs):
        return self.session.delete(url, **kwargs)

    def close(self):
        self.session.close()


client = RetrySession(retries=5, backoff_factor=1)
response = client.get("https://httpbin.org/delay/2", timeout=10)
print(f"Status: {response.status_code}")
client.close()
```

### 2. OAuth2 Token Management

```python
import requests
import time

class OAuth2Client:
    def __init__(self, client_id, client_secret, token_url):
        self.client_id = client_id
        self.client_secret = client_secret
        self.token_url = token_url
        self.access_token = None
        self.token_expiry = 0
        self.session = requests.Session()

    def _is_token_expired(self):
        return time.time() >= self.token_expiry

    def _fetch_new_token(self):
        response = requests.post(
            self.token_url,
            data={
                "grant_type": "client_credentials",
                "client_id": self.client_id,
                "client_secret": self.client_secret
            },
            headers={"Accept": "application/json"}
        )
        response.raise_for_status()
        token_data = response.json()
        self.access_token = token_data["access_token"]
        self.token_expiry = time.time() + token_data.get("expires_in", 3600) - 60
        self.session.headers.update({"Authorization": f"Bearer {self.access_token}"})

    def request(self, method, url, **kwargs):
        if self._is_token_expired():
            self._fetch_new_token()
        response = self.session.request(method, url, **kwargs)
        if response.status_code == 401:
            self._fetch_new_token()
            response = self.session.request(method, url, **kwargs)
        return response

    def get(self, url, **kwargs):
        return self.request("GET", url, **kwargs)

    def post(self, url, **kwargs):
        return self.request("POST", url, **kwargs)


client = OAuth2Client(
    client_id="my_client",
    client_secret="my_secret",
    token_url="https://httpbin.org/post"
)
response = client.get("https://httpbin.org/headers")
print(response.json())
```

### 3. Async HTTP Requests with `httpx`

```python
import httpx
import asyncio

async def fetch_url(client, url):
    response = await client.get(url)
    return response.json()

async def main():
    urls = [
        "https://jsonplaceholder.typicode.com/posts/1",
        "https://jsonplaceholder.typicode.com/posts/2",
        "https://jsonplaceholder.typicode.com/posts/3",
        "https://jsonplaceholder.typicode.com/posts/4",
        "https://jsonplaceholder.typicode.com/posts/5",
    ]

    async with httpx.AsyncClient() as client:
        tasks = [fetch_url(client, url) for url in urls]
        results = await asyncio.gather(*tasks)

    for result in results:
        print(f"Post {result['id']}: {result['title']}")

asyncio.run(main())
```

### 4. Request Hooks and Events

```python
import requests

def log_request(response, *args, **kwargs):
    print(f"Request: {response.request.method} {response.request.url}")
    print(f"Response: {response.status_code}")

def log_error(response, *args, **kwargs):
    if response.status_code >= 400:
        print(f"ERROR: {response.status_code} for {response.url}")

session = requests.Session()
session.hooks["response"] = [log_request, log_error]

session.get("https://httpbin.org/status/200")
session.get("https://httpbin.org/status/404")
session.get("https://httpbin.org/status/500")
```

### 5. Rate Limiting with Token Bucket

```python
import requests
import time
import threading

class RateLimiter:
    def __init__(self, max_requests, time_window):
        self.max_requests = max_requests
        self.time_window = time_window
        self.tokens = max_requests
        self.last_refill = time.time()
        self.lock = threading.Lock()

    def _refill(self):
        now = time.time()
        elapsed = now - self.last_refill
        new_tokens = elapsed * (self.max_requests / self.time_window)
        self.tokens = min(self.max_requests, self.tokens + new_tokens)
        self.last_refill = now

    def acquire(self):
        while True:
            with self.lock:
                self._refill()
                if self.tokens >= 1:
                    self.tokens -= 1
                    return
            time.sleep(0.01)

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            self.acquire()
            return func(*args, **kwargs)
        return wrapper


limiter = RateLimiter(max_requests=10, time_window=1)

@limiter
def rate_limited_request(url):
    return requests.get(url)

for i in range(20):
    response = rate_limited_request("https://httpbin.org/get")
    print(f"Request {i+1}: Status {response.status_code}")
    time.sleep(0.05)
```

### 6. Circuit Breaker Pattern

```python
import requests
import time

class CircuitBreaker:
    def __init__(self, failure_threshold=5, reset_timeout=30):
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.failure_count = 0
        self.last_failure_time = 0
        self.state = "CLOSED"

    def call(self, func, *args, **kwargs):
        if self.state == "OPEN":
            if time.time() - self.last_failure_time > self.reset_timeout:
                self.state = "HALF_OPEN"
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = func(*args, **kwargs)
            if self.state == "HALF_OPEN":
                self.state = "CLOSED"
                self.failure_count = 0
            self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            if self.failure_count >= self.failure_threshold:
                self.state = "OPEN"
            raise e


cb = CircuitBreaker(failure_threshold=3, reset_timeout=10)
session = requests.Session()

for i in range(10):
    try:
        response = cb.call(session.get, "https://httpbin.org/status/500")
        print(f"Request {i+1}: Success")
    except Exception as e:
        print(f"Request {i+1}: {e}")
    time.sleep(1)
```

### 7. Multipart Form Upload with Metadata

```python
import requests
import json

url = "https://httpbin.org/post"

multipart_data = {
    "metadata": (None, json.dumps({"user_id": 123, "type": "document"}), "application/json"),
    "document": ("report.pdf", b"%PDF-1.4 fake pdf content", "application/pdf"),
    "thumbnail": ("thumb.png", b"PNG fake content", "image/png"),
    "tags": (None, "python,http,upload")
}

response = requests.post(url, files=multipart_data)
result = response.json()
print(f"Uploaded files: {list(result.get('files', {}).keys())}")
print(f"Form data: {result.get('form', {})}")
```

### 8. Connection Pool Tuning

```python
import requests
from requests.adapters import HTTPAdapter

session = requests.Session()

adapter = HTTPAdapter(
    pool_connections=20,
    pool_maxsize=100,
    max_retries=3
)

session.mount("https://", adapter)
session.mount("http://", adapter)

urls = [f"https://jsonplaceholder.typicode.com/posts/{i}" for i in range(1, 21)]

responses = []
for url in urls:
    resp = session.get(url)
    responses.append(resp)

print(f"Fetched {len(responses)} posts")
print(f"Status codes: {[r.status_code for r in responses][:5]}...")
session.close()
```

### 9. Asynchronous Batch Processing

```python
import requests
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def fetch_post(post_id):
    url = f"https://jsonplaceholder.typicode.com/posts/{post_id}"
    response = requests.get(url, timeout=10)
    return response.json()

def batch_fetch(post_ids, max_workers=10):
    results = {}
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        future_to_id = {executor.submit(fetch_post, pid): pid for pid in post_ids}
        for future in as_completed(future_to_id):
            post_id = future_to_id[future]
            try:
                results[post_id] = future.result()
            except Exception as e:
                print(f"Post {post_id} failed: {e}")
    return results


start = time.time()
results = batch_fetch(range(1, 51), max_workers=20)
elapsed = time.time() - start

print(f"Fetched {len(results)} posts in {elapsed:.2f}s")
for pid, post in list(results.items())[:3]:
    print(f"Post {pid}: {post['title'][:50]}")
```

### 10. Full-Featured HTTP Client Class

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry
import logging
import time

logging.basicConfig(level=logging.INFO)

class HTTPClient:
    def __init__(self, base_url="", default_timeout=30, retries=3):
        self.base_url = base_url.rstrip("/")
        self.default_timeout = default_timeout
        self.session = requests.Session()

        retry_strategy = Retry(
            total=retries,
            backoff_factor=0.5,
            status_forcelist=[429, 500, 502, 503, 504],
            allowed_methods=["HEAD", "GET", "PUT", "POST", "DELETE", "OPTIONS", "TRACE"]
        )
        adapter = HTTPAdapter(
            max_retries=retry_strategy,
            pool_connections=20,
            pool_maxsize=100
        )
        self.session.mount("https://", adapter)
        self.session.mount("http://", adapter)
        self.logger = logging.getLogger(self.__class__.__name__)

    def _build_url(self, endpoint):
        if self.base_url:
            return f"{self.base_url}/{endpoint.lstrip('/')}"
        return endpoint

    def _log_request(self, method, url, kwargs):
        self.logger.info(f"{method} {url}")
        if "headers" in kwargs:
            self.logger.debug(f"Headers: {kwargs['headers']}")
        if "json" in kwargs:
            self.logger.debug(f"JSON Body: {kwargs['json']}")
        if "params" in kwargs:
            self.logger.debug(f"Params: {kwargs['params']}")

    def request(self, method, endpoint, **kwargs):
        url = self._build_url(endpoint)
        kwargs.setdefault("timeout", self.default_timeout)
        self._log_request(method, url, kwargs)

        try:
            response = self.session.request(method, url, **kwargs)
            response.raise_for_status()
            return response
        except requests.exceptions.HTTPError as e:
            self.logger.error(f"HTTP Error: {e}")
            raise
        except requests.exceptions.ConnectionError as e:
            self.logger.error(f"Connection Error: {e}")
            raise
        except requests.exceptions.Timeout as e:
            self.logger.error(f"Timeout: {e}")
            raise
        except requests.exceptions.RequestException as e:
            self.logger.error(f"Request Error: {e}")
            raise

    def get(self, endpoint, **kwargs):
        return self.request("GET", endpoint, **kwargs)

    def post(self, endpoint, **kwargs):
        return self.request("POST", endpoint, **kwargs)

    def put(self, endpoint, **kwargs):
        return self.request("PUT", endpoint, **kwargs)

    def patch(self, endpoint, **kwargs):
        return self.request("PATCH", endpoint, **kwargs)

    def delete(self, endpoint, **kwargs):
        return self.request("DELETE", endpoint, **kwargs)

    def head(self, endpoint, **kwargs):
        return self.request("HEAD", endpoint, **kwargs)

    def options(self, endpoint, **kwargs):
        return self.request("OPTIONS", endpoint, **kwargs)

    def close(self):
        self.session.close()


client = HTTPClient(base_url="https://jsonplaceholder.typicode.com", retries=2)

try:
    resp = client.get("/posts/1")
    print(f"GET: {resp.status_code} - {resp.json()['title'][:40]}")

    resp = client.post("/posts", json={"title": "Test", "body": "Body", "userId": 1})
    print(f"POST: {resp.status_code} - ID {resp.json()['id']}")

    resp = client.put("/posts/1", json={"title": "Updated", "body": "Updated", "userId": 1})
    print(f"PUT: {resp.status_code}")

    resp = client.delete("/posts/1")
    print(f"DELETE: {resp.status_code}")
finally:
    client.close()
```

## Real-World Use Cases

- **API Clients**: Wrapping third-party APIs (Stripe, GitHub, Slack, Twitter) with Python SDKs
- **Web Scraping**: Fetching HTML content for parsing and data extraction
- **CI/CD Pipelines**: Triggering builds, fetching artifacts, reporting to services
- **Monitoring Tools**: Health-checking endpoints, collecting metrics
- **Data Pipelines**: Pulling data from REST APIs into databases or data lakes
- **Chat Bots**: Sending messages to webhook endpoints (Slack, Discord, Telegram)
- **Payment Processing**: Communicating with payment gateway APIs
- **Cloud Automation**: Managing cloud resources through provider APIs (AWS, Azure, GCP)
- **Microservices Communication**: Service-to-service HTTP calls in distributed systems
- **Authentication Services**: Token exchange, OAuth flows, session management

## Common Mistakes

- **Not handling timeouts** — requests hang indefinitely, blocking threads
- **Ignoring status codes** — assuming every 200 response succeeds without checking
- **Hardcoding tokens** — exposing secrets in code instead of environment variables
- **Not using sessions** — creating new connections for every request (slow)
- **Forgetting to close sessions** — causing resource leaks
- **Not setting User-Agent** — some APIs block clients without proper headers
- **Overlooking redirects** — not handling 3xx responses appropriately
- **Ignoring rate limits** — getting blocked for sending too many requests
- **Not validating SSL** — disabling certificate verification in production
- **Swallowing exceptions** — bare `except:` catching all errors silently

## Best Practices

- Always use `timeout` to prevent hanging requests
- Use `requests.Session()` for multiple calls to the same host
- Implement retry logic with exponential backoff for transient failures
- Store secrets in environment variables, never in code
- Validate response status codes and raise meaningful exceptions
- Use connection pooling for high-throughput applications
- Set appropriate `User-Agent` headers for identification
- Handle all exception types (`Timeout`, `ConnectionError`, `HTTPError`)
- Use context managers (`with`) for session and response objects
- Log request/response details for debugging without sensitive data

## Interview Questions

**Q1: What is the difference between `urllib.request` and `requests`?**
A: `urllib.request` is built-in but verbose and requires manual encoding/decoding. `requests` is third-party with a cleaner API, automatic JSON handling, session support, and better error handling.

**Q2: How do you handle request timeouts in Python?**
A: Pass the `timeout` parameter to requests methods. The timeout applies to connect and read phases. Use `timeout=(connect_timeout, read_timeout)` to set both separately.

**Q3: How does `requests.Session()` improve performance?**
A: Sessions persist cookies, reuse underlying TCP connections (connection pooling), and maintain default headers across requests, reducing latency and resource usage.

**Q4: What is the difference between `data` and `json` parameters in `requests.post()`?**
A: `data` sends form-encoded data (application/x-www-form-urlencoded). `json` serializes the dict to JSON and sets Content-Type to application/json.

**Q5: How do you implement retry logic with exponential backoff?**
A: Use `urllib3.util.retry.Retry` with `HTTPAdapter` mounted on a `Session`. Configure total retries, backoff_factor, and status_forcelist for automatic retries.

## Coding Challenges

**Challenge 1: Simple URL Shortener Client**
Write a function `shorten_url(long_url)` that sends a POST request to https://api.example.com/shorten with the long URL and returns the shortened URL. Handle errors and timeouts.

**Challenge 2: Rate-Limited API Fetcher**
Create a class that fetches data from a public API (e.g., JSONPlaceholder) while respecting a rate limit of 5 requests per second. Use a token bucket algorithm.

**Challenge 3: Multi-Threaded Web Checker**
Write a script that checks HTTP status codes for a list of 100 URLs concurrently using ThreadPoolExecutor. Report which URLs are down (non-200).

**Challenge 4: HTTP Client with Circuit Breaker**
Extend the `HTTPClient` class to include a circuit breaker that opens after 5 consecutive failures and resets after 30 seconds.

**Challenge 5: File Downloader with Resume**
Build a downloader that supports resuming interrupted downloads using the `Range` header and checking `Content-Range` in responses.

## Summary

HTTP requests are a cornerstone of networked Python applications. The `requests` library provides a powerful, user-friendly API for sending HTTP requests, handling sessions, authentication, timeouts, retries, and more. Understanding how to properly configure and manage HTTP interactions is essential for building robust, production-ready applications that communicate over the web. From simple API calls to complex asynchronous batch processing, mastering HTTP requests unlocks the ability to integrate with virtually any web service.

## Related Topics

- [71. APIs](./71_apis.md)
- [75. REST API Design](./75_rest_api_design.md)
- [76. Authentication](./76_authentication.md)
- [72. Flask](./72_flask.md)
- [73. FastAPI](./73_fastapi.md)
- [74. Web Scraping](./74_web_scraping.md)
