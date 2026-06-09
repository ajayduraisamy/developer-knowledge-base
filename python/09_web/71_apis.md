# APIs - Designing REST APIs, status codes, versioning, pagination

## Introduction

An Application Programming Interface (API) is a set of defined rules that enable different software components to communicate with each other. In modern web development, APIs typically refer to web APIs that expose endpoints over HTTP, allowing clients to perform operations on resources. REST (Representational State Transfer) is the most common architectural style for designing web APIs, though GraphQL, gRPC, and SOAP are also used. Python developers frequently build, consume, and interact with APIs across many domains — from microservices to data science to automation.

## Why It Is Important

APIs are the backbone of modern software integration. They allow developers to leverage third-party services (payment gateways, mapping services, AI models), decompose monolithic applications into microservices, enable mobile and frontend applications to access backend logic, and facilitate automation across tools and platforms. Understanding API design, consumption, versioning, authentication, and documentation is essential for building maintainable, scalable, and interoperable systems.

## Syntax

### Consuming a REST API

```python
import requests

# GET request
response = requests.get("https://api.github.com/users/octocat")
data = response.json()

# POST request
response = requests.post(
    "https://api.example.com/items",
    json={"name": "New Item", "price": 29.99},
    headers={"Authorization": "Bearer token123"}
)
```

### Defining API Endpoints with FastAPI

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id, "name": "Sample"}

@app.post("/items")
async def create_item(item: Item):
    return {"id": 1, **item.model_dump()}
```

## Examples

### Simple API Client

```python
import requests

class GithubClient:
    BASE_URL = "https://api.github.com"

    def __init__(self, token=None):
        self.session = requests.Session()
        if token:
            self.session.headers.update({"Authorization": f"Bearer {token}"})
        self.session.headers.update({"Accept": "application/vnd.github.v3+json"})

    def get_user(self, username):
        response = self.session.get(f"{self.BASE_URL}/users/{username}")
        response.raise_for_status()
        return response.json()

    def get_repos(self, username):
        response = self.session.get(f"{self.BASE_URL}/users/{username}/repos")
        response.raise_for_status()
        return response.json()

    def create_repo(self, name, description="", private=False):
        data = {"name": name, "description": description, "private": private}
        response = self.session.post(f"{self.BASE_URL}/user/repos", json=data)
        response.raise_for_status()
        return response.json()


client = GithubClient()
user = client.get_user("octocat")
print(f"User: {user['login']} - {user['name']}")
```

### Pagination Handling

```python
import requests

def fetch_all_pages(base_url, params=None):
    all_items = []
    page = 1

    while True:
        page_params = {"page": page, "per_page": 100}
        if params:
            page_params.update(params)

        response = requests.get(base_url, params=page_params)
        response.raise_for_status()

        data = response.json()
        if not data:
            break

        all_items.extend(data)
        page += 1

        # Check Link header for last page
        link_header = response.headers.get("Link", "")
        if 'rel="next"' not in link_header:
            break

    return all_items


repos = fetch_all_pages("https://api.github.com/users/octocat/repos")
print(f"Total repos: {len(repos)}")
```

## Beginner Examples

### 1. Consuming a Public REST API

```python
import requests

url = "https://jsonplaceholder.typicode.com/posts"
response = requests.get(url)

if response.status_code == 200:
    posts = response.json()
    print(f"Fetched {len(posts)} posts")
    for post in posts[:3]:
        print(f"  - {post['title'][:50]}")
else:
    print(f"Error: {response.status_code}")
```

### 2. Sending Data to an API

```python
import requests

new_post = {
    "title": "My New Post",
    "body": "This is the content of the post.",
    "userId": 1
}

response = requests.post(
    "https://jsonplaceholder.typicode.com/posts",
    json=new_post
)

if response.status_code == 201:
    created = response.json()
    print(f"Created post with ID: {created['id']}")
else:
    print(f"Failed: {response.status_code}")
```

### 3. Updating and Deleting Resources

```python
import requests

# Update (PUT - complete replacement)
update_data = {"id": 1, "title": "Updated Title", "body": "Updated body", "userId": 1}
response = requests.put(
    "https://jsonplaceholder.typicode.com/posts/1",
    json=update_data
)
print(f"PUT status: {response.status_code}")

# Partial update (PATCH)
patch_data = {"title": "Partially Updated Title"}
response = requests.patch(
    "https://jsonplaceholder.typicode.com/posts/1",
    json=patch_data
)
print(f"PATCH status: {response.status_code}")

# Delete
response = requests.delete("https://jsonplaceholder.typicode.com/posts/1")
print(f"DELETE status: {response.status_code}")
```

### 4. API with Query Parameters

```python
import requests

url = "https://jsonplaceholder.typicode.com/comments"
params = {"postId": 1}

response = requests.get(url, params=params)
comments = response.json()

print(f"Comments for post 1: {len(comments)}")
for comment in comments[:2]:
    print(f"  - {comment['email']}: {comment['name']}")
```

### 5. Error Handling in API Calls

```python
import requests
from requests.exceptions import RequestException

def safe_api_call(url, method="GET", **kwargs):
    try:
        response = requests.request(method, url, timeout=10, **kwargs)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.HTTPError as e:
        print(f"HTTP Error: {e.response.status_code} - {e.response.reason}")
        if e.response.status_code == 404:
            print("Resource not found")
        elif e.response.status_code == 403:
            print("Access forbidden")
        elif e.response.status_code == 429:
            print("Rate limited")
    except requests.exceptions.ConnectionError:
        print("Connection error - is the server running?")
    except requests.exceptions.Timeout:
        print("Request timed out")
    except RequestException as e:
        print(f"API call failed: {e}")
    return None


result = safe_api_call("https://jsonplaceholder.typicode.com/posts/1")
if result:
    print(f"Success: {result['title']}")

result = safe_api_call("https://jsonplaceholder.typicode.com/posts/99999")
```

## Intermediate Examples

### 1. API Client with Rate Limiting

```python
import requests
import time

class RateLimitedAPIClient:
    def __init__(self, base_url, rate_limit=10, period=1):
        self.base_url = base_url.rstrip("/")
        self.rate_limit = rate_limit
        self.period = period
        self.request_times = []
        self.session = requests.Session()

    def _wait_if_needed(self):
        now = time.time()
        self.request_times = [t for t in self.request_times if now - t < self.period]
        if len(self.request_times) >= self.rate_limit:
            sleep_time = self.request_times[0] + self.period - now
            if sleep_time > 0:
                print(f"Rate limit reached, sleeping {sleep_time:.2f}s")
                time.sleep(sleep_time)
        self.request_times.append(time.time())

    def request(self, method, endpoint, **kwargs):
        self._wait_if_needed()
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = self.session.request(method, url, **kwargs)
        response.raise_for_status()
        return response

    def get(self, endpoint, **kwargs):
        return self.request("GET", endpoint, **kwargs)


client = RateLimitedAPIClient(
    base_url="https://jsonplaceholder.typicode.com",
    rate_limit=5,
    period=1
)

for i in range(1, 11):
    resp = client.get(f"/posts/{i}")
    print(f"Post {i}: {resp.json()['title'][:30]}")
```

### 2. Retry with Exponential Backoff for API Calls

```python
import requests
import time

def api_call_with_retry(url, max_retries=5, base_delay=1, **kwargs):
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=10, **kwargs)
            if response.status_code == 429:
                retry_after = int(response.headers.get("Retry-After", base_delay * (2 ** attempt)))
                print(f"Rate limited. Waiting {retry_after}s...")
                time.sleep(retry_after)
                continue
            response.raise_for_status()
            return response.json()
        except (requests.exceptions.ConnectionError,
                requests.exceptions.Timeout) as e:
            if attempt == max_retries - 1:
                raise
            delay = base_delay * (2 ** attempt)
            print(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay}s...")
            time.sleep(delay)
        except requests.exceptions.HTTPError as e:
            if e.response.status_code in (500, 502, 503, 504):
                if attempt == max_retries - 1:
                    raise
                delay = base_delay * (2 ** attempt)
                print(f"Server error {e.response.status_code}. Retrying in {delay}s...")
                time.sleep(delay)
            else:
                raise
    return None


try:
    data = api_call_with_retry("https://jsonplaceholder.typicode.com/posts/1")
    print(f"Data: {data['title']}")
except Exception as e:
    print(f"Failed after retries: {e}")
```

### 3. Building a Pagination Iterator

```python
import requests
from typing import Iterator, Dict, Any

class PaginatedAPI:
    def __init__(self, base_url, params=None, page_size=100):
        self.base_url = base_url
        self.params = params or {}
        self.page_size = page_size

    def __iter__(self) -> Iterator[Dict[str, Any]]:
        page = 1
        while True:
            params = {**self.params, "page": page, "per_page": self.page_size}
            response = requests.get(self.base_url, params=params)
            response.raise_for_status()

            data = response.json()
            if not data:
                break

            for item in data:
                yield item

            page += 1

            link = response.headers.get("Link", "")
            if 'rel="next"' not in link:
                break


paginator = PaginatedAPI(
    base_url="https://api.github.com/users/octocat/repos",
    page_size=5
)

count = 0
for repo in paginator:
    print(f"Repo: {repo['name']}")
    count += 1
    if count >= 10:
        break
print(f"Listed {count} repos")
```

### 4. API Versioning Strategies

```python
import requests

# URL-based versioning (common)
BASE_URL_V1 = "https://api.example.com/v1"
BASE_URL_V2 = "https://api.example.com/v2"

def get_users_v1():
    response = requests.get(f"{BASE_URL_V1}/users")
    return response.json()

def get_users_v2():
    response = requests.get(f"{BASE_URL_V2}/users")
    return response.json()

# Header-based versioning
def get_users_header_version(version=1):
    headers = {"Accept": f"application/vnd.example.v{version}+json"}
    response = requests.get("https://api.example.com/users", headers=headers)
    return response.json()

# Query parameter versioning
def get_users_query_version(version=1):
    response = requests.get(
        "https://api.example.com/users",
        params={"api_version": version}
    )
    return response.json()


headers = {"Accept": "application/vnd.github.v3+json"}
response = requests.get("https://api.github.com/zen", headers=headers)
print(f"GitHub API response: {response.text}")
```

### 5. API Authentication Patterns

```python
import requests

# API Key in header
def call_with_api_key(url, api_key):
    headers = {"X-API-Key": api_key}
    response = requests.get(url, headers=headers)
    return response

# Bearer token (JWT)
def call_with_bearer_token(url, token):
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get(url, headers=headers)
    return response

# Basic authentication
def call_with_basic_auth(url, username, password):
    from requests.auth import HTTPBasicAuth
    response = requests.get(url, auth=HTTPBasicAuth(username, password))
    return response

# OAuth2 token from environment
import os
token = os.environ.get("API_TOKEN", "demo_token")
response = call_with_bearer_token("https://httpbin.org/bearer", token)
print(f"Authenticated: {response.status_code}")
```

### 6. Webhook Client

```python
import requests
import json
import hmac
import hashlib

class WebhookClient:
    def __init__(self, webhook_url, secret=None):
        self.webhook_url = webhook_url
        self.secret = secret

    def _compute_signature(self, payload):
        if not self.secret:
            return None
        return hmac.new(
            self.secret.encode(),
            payload.encode(),
            hashlib.sha256
        ).hexdigest()

    def send_event(self, event_type, data):
        payload = json.dumps({"event": event_type, "data": data})
        headers = {
            "Content-Type": "application/json",
            "X-Event-Type": event_type
        }

        signature = self._compute_signature(payload)
        if signature:
            headers["X-Signature-256"] = signature

        response = requests.post(
            self.webhook_url,
            data=payload,
            headers=headers,
            timeout=10
        )
        response.raise_for_status()
        return response.status_code


wh = WebhookClient(
    webhook_url="https://httpbin.org/post",
    secret="whsec_abc123"
)

status = wh.send_event("order.created", {"order_id": 123, "total": 49.99})
print(f"Webhook sent, status: {status}")
```

### 7. GraphQL API Client

```python
import requests
import json

class GraphQLClient:
    def __init__(self, endpoint, token=None):
        self.endpoint = endpoint
        self.session = requests.Session()
        if token:
            self.session.headers.update({"Authorization": f"Bearer {token}"})
        self.session.headers.update({"Content-Type": "application/json"})

    def query(self, query, variables=None):
        payload = {"query": query}
        if variables:
            payload["variables"] = variables
        response = self.session.post(self.endpoint, json=payload)
        response.raise_for_status()
        result = response.json()
        if "errors" in result:
            raise Exception(f"GraphQL errors: {result['errors']}")
        return result["data"]

    def mutate(self, mutation, variables=None):
        return self.query(mutation, variables)


client = GraphQLClient("https://api.github.com/graphql", token="demo_token")

query = """
query {
    repository(owner: "octocat", name: "Hello-World") {
        name
        description
        stargazerCount
        forkCount
    }
}
"""

try:
    data = client.query(query)
    repo = data["repository"]
    print(f"Repo: {repo['name']} - Stars: {repo['stargazerCount']}")
except Exception as e:
    print(f"GraphQL error (expected with demo token): {e}")
```

### 8. Bulk API Operations

```python
import requests
from concurrent.futures import ThreadPoolExecutor, as_completed

class BulkAPIClient:
    def __init__(self, base_url, max_workers=10):
        self.base_url = base_url.rstrip("/")
        self.session = requests.Session()
        self.executor = ThreadPoolExecutor(max_workers=max_workers)

    def fetch_single(self, resource_type, resource_id):
        url = f"{self.base_url}/{resource_type}/{resource_id}"
        response = self.session.get(url, timeout=10)
        response.raise_for_status()
        return (resource_id, response.json())

    def fetch_bulk(self, resource_type, ids):
        futures = {
            self.executor.submit(self.fetch_single, resource_type, id): id
            for id in ids
        }
        results = {}
        for future in as_completed(futures):
            resource_id = futures[future]
            try:
                _, data = future.result()
                results[resource_id] = data
            except Exception as e:
                print(f"Failed to fetch {resource_type}/{resource_id}: {e}")
        return results


client = BulkAPIClient("https://jsonplaceholder.typicode.com", max_workers=5)
results = client.fetch_bulk("posts", range(1, 21))

print(f"Fetched {len(results)} posts")
for pid in sorted(results.keys())[:3]:
    print(f"  Post {pid}: {results[pid]['title'][:40]}")
```

### 9. API Response Caching

```python
import requests
import time
from functools import lru_cache

class CachedAPIClient:
    def __init__(self, base_url, cache_ttl=60):
        self.base_url = base_url.rstrip("/")
        self.session = requests.Session()
        self.cache_ttl = cache_ttl
        self._cache = {}
        self._cache_times = {}

    def _is_cache_valid(self, key):
        if key not in self._cache_times:
            return False
        return time.time() - self._cache_times[key] < self.cache_ttl

    def get(self, endpoint, params=None, use_cache=True):
        cache_key = f"{endpoint}:{str(params)}"

        if use_cache and self._is_cache_valid(cache_key):
            print(f"Cache hit for {cache_key}")
            return self._cache[cache_key]

        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = self.session.get(url, params=params, timeout=10)
        response.raise_for_status()
        data = response.json()

        self._cache[cache_key] = data
        self._cache_times[cache_key] = time.time()
        return data

    def clear_cache(self):
        self._cache.clear()
        self._cache_times.clear()


client = CachedAPIClient("https://jsonplaceholder.typicode.com", cache_ttl=3)

client.get("/posts/1")
print("First call (cache miss)")

client.get("/posts/1")
print("Second call (cache hit)")

time.sleep(4)
client.get("/posts/1")
print("Third call (cache expired)")
```

### 10. Comprehensive API Client with All Features

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry
import time
import logging
from typing import Optional, Dict, Any, List

logging.basicConfig(level=logging.INFO)

class ComprehensiveAPIClient:
    def __init__(
        self,
        base_url: str,
        api_key: Optional[str] = None,
        token: Optional[str] = None,
        rate_limit: int = 60,
        rate_period: int = 60,
        max_retries: int = 3,
        default_timeout: int = 30
    ):
        self.base_url = base_url.rstrip("/")
        self.default_timeout = default_timeout
        self.logger = logging.getLogger(self.__class__.__name__)

        self.session = requests.Session()

        if api_key:
            self.session.headers.update({"X-API-Key": api_key})
        if token:
            self.session.headers.update({"Authorization": f"Bearer {token}"})

        retry_strategy = Retry(
            total=max_retries,
            backoff_factor=1,
            status_forcelist=[429, 500, 502, 503, 504],
            allowed_methods=["GET", "POST", "PUT", "PATCH", "DELETE"]
        )
        adapter = HTTPAdapter(max_retries=retry_strategy, pool_connections=20, pool_maxsize=100)
        self.session.mount("https://", adapter)
        self.session.mount("http://", adapter)

        self.rate_limit = rate_limit
        self.rate_period = rate_period
        self.request_times: List[float] = []

        self.cache: Dict[str, Any] = {}
        self.cache_ttl: Dict[str, float] = {}

    def _rate_limit_wait(self):
        now = time.time()
        self.request_times = [t for t in self.request_times if now - t < self.rate_period]
        if len(self.request_times) >= self.rate_limit:
            sleep = self.request_times[0] + self.rate_period - now
            if sleep > 0:
                self.logger.warning(f"Rate limit reached, sleeping {sleep:.1f}s")
                time.sleep(sleep)
        self.request_times.append(time.time())

    def _build_url(self, endpoint: str) -> str:
        return f"{self.base_url}/{endpoint.lstrip('/')}"

    def _cache_key(self, method: str, endpoint: str, params: Optional[Dict] = None) -> str:
        return f"{method}:{endpoint}:{str(params)}"

    def _get_from_cache(self, key: str) -> Optional[Any]:
        if key in self.cache:
            if time.time() - self.cache_ttl.get(key, 0) < 60:
                self.logger.debug(f"Cache hit: {key}")
                return self.cache[key]
            else:
                del self.cache[key]
                del self.cache_ttl[key]
        return None

    def _set_cache(self, key: str, data: Any):
        self.cache[key] = data
        self.cache_ttl[key] = time.time()

    def request(
        self,
        method: str,
        endpoint: str,
        use_cache: bool = False,
        **kwargs
    ) -> requests.Response:
        url = self._build_url(endpoint)
        kwargs.setdefault("timeout", self.default_timeout)

        cache_key = self._cache_key(method, endpoint, kwargs.get("params"))
        if use_cache and method == "GET":
            cached = self._get_from_cache(cache_key)
            if cached:
                return cached

        self._rate_limit_wait()
        self.logger.info(f"{method} {url}")

        try:
            response = self.session.request(method, url, **kwargs)
            response.raise_for_status()

            if use_cache and method == "GET":
                self._set_cache(cache_key, response)

            return response
        except requests.exceptions.HTTPError as e:
            self.logger.error(f"HTTP {e.response.status_code}: {e}")
            raise
        except requests.exceptions.ConnectionError as e:
            self.logger.error(f"Connection error: {e}")
            raise
        except requests.exceptions.Timeout as e:
            self.logger.error(f"Timeout: {e}")
            raise

    def get(self, endpoint: str, **kwargs):
        return self.request("GET", endpoint, **kwargs)

    def post(self, endpoint: str, **kwargs):
        return self.request("POST", endpoint, **kwargs)

    def put(self, endpoint: str, **kwargs):
        return self.request("PUT", endpoint, **kwargs)

    def patch(self, endpoint: str, **kwargs):
        return self.request("PATCH", endpoint, **kwargs)

    def delete(self, endpoint: str, **kwargs):
        return self.request("DELETE", endpoint, **kwargs)

    def close(self):
        self.session.close()


client = ComprehensiveAPIClient(
    base_url="https://jsonplaceholder.typicode.com",
    rate_limit=30,
    rate_period=60
)

try:
    resp = client.get("/posts/1")
    print(f"Fetched post: {resp.json()['title'][:40]}")

    resp = client.post("/posts", json={"title": "Test", "body": "Body", "userId": 1})
    print(f"Created post: ID {resp.json()['id']}")
finally:
    client.close()
```

## Real-World Use Cases

- **Payment Processing**: Stripe, PayPal, Square API integration for payments
- **Social Media Automation**: Twitter, LinkedIn, Instagram API clients for posting
- **Cloud Infrastructure**: AWS, Azure, GCP SDKs for resource management
- **Mapping and Geolocation**: Google Maps, Mapbox, OpenStreetMap APIs
- **Communication**: Twilio (SMS/voice), SendGrid (email), Slack (messaging)
- **AI and ML**: OpenAI, Google Cloud Vision, AWS Rekognition APIs
- **E-commerce**: Shopify, WooCommerce, Magento APIs for store management
- **Financial Data**: Alpha Vantage, Yahoo Finance, Plaid APIs
- **Travel**: Amadeus, Skyscanner, Kayak APIs for flight/hotel data
- **Blockchain**: Ethereum, Bitcoin, CoinGecko APIs for crypto data

## Common Mistakes

- **Ignoring rate limits** — causing IP bans or account suspension
- **Not handling pagination** — only fetching the first page of results
- **Hardcoding API keys** — committing secrets to version control
- **No error handling** — crashing on network errors or unexpected responses
- **Not validating responses** — assuming the API always returns expected data
- **Missing timeout values** — threads hanging indefinitely
- **Overfetching data** — requesting more fields than needed, wasting bandwidth
- **Poor versioning strategy** — breaking existing clients when APIs change
- **Inconsistent error formats** — making error handling difficult for consumers
- **No logging/observability** — making debugging API issues nearly impossible

## Best Practices

- Always use environment variables for API keys and secrets
- Implement proper error handling with meaningful error messages
- Use pagination for large datasets and expose page/page_size or cursor-based params
- Version your APIs from day one (URL or header based)
- Return consistent, well-structured error responses (RFC 7807 Problem Details)
- Use appropriate HTTP status codes correctly
- Document APIs thoroughly with OpenAPI/Swagger
- Implement rate limiting and communicate limits via headers
- Validate input data with schemas (Pydantic, Marshmallow)
- Use idempotency keys for mutation operations to prevent duplicate processing

## Interview Questions

**Q1: What is REST and what are its constraints?**
A: REST (Representational State Transfer) is an architectural style with constraints: statelessness, client-server separation, cacheability, uniform interface, layered system, and code on demand (optional).

**Q2: What is the difference between PUT and PATCH?**
A: PUT replaces the entire resource. PATCH applies a partial update. PUT is idempotent; PATCH may not be depending on implementation.

**Q3: How do you handle API versioning?**
A: Common strategies include URL path versioning (`/v1/users`), header versioning (`Accept: application/vnd.example.v1+json`), and query parameter versioning (`?version=1`).

**Q4: What is HATEOAS?**
A: HATEOAS (Hypermedia as the Engine of Application State) means API responses include links to related actions, allowing clients to discover and navigate the API dynamically.

**Q5: How do you design a paginated API?**
A: Use either page-based (page, per_page) or cursor-based (cursor, limit) pagination. Return metadata (total count, next/previous links) and support consistent ordering.

## Coding Challenges

**Challenge 1: Weather API Client**
Build a client for OpenWeatherMap API that fetches current weather for a city. Handle 401 (invalid key), 404 (city not found), and rate limiting.

**Challenge 2: Paginated GitHub Issues Viewer**
Write a function that fetches all issues from a GitHub repository, handling pagination via Link headers. Return a list of issue titles and labels.

**Challenge 3: API Rate Limiter Middleware**
Create a decorator that wraps any API client method, ensuring no more than N requests per second. Use a token bucket algorithm.

**Challenge 4: Multi-API Dashboard Aggregator**
Query multiple APIs (news, weather, stocks) concurrently and aggregate results into a single dashboard-friendly response.

**Challenge 5: API Gateway Simulation**
Build a simple API gateway that routes requests to different backend services, handles authentication, applies rate limiting, and logs all requests.

## Summary

APIs are the fundamental building blocks of modern software integration and communication. Python's rich ecosystem — particularly the `requests` library — makes consuming APIs straightforward, while frameworks like FastAPI and Flask simplify building them. Successful API development requires attention to design principles, authentication, error handling, rate limiting, caching, pagination, and documentation. Mastering these patterns enables developers to build robust, scalable, and maintainable systems that integrate seamlessly with the broader software ecosystem.

## Related Topics

- [70. HTTP Requests](./70_http_requests.md)
- [75. REST API Design](./75_rest_api_design.md)
- [73. FastAPI](./73_fastapi.md)
- [72. Flask](./72_flask.md)
- [76. Authentication](./76_authentication.md)
- [74. Web Scraping](./74_web_scraping.md)
