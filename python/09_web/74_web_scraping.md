# Web Scraping - BeautifulSoup, requests, Selenium, Playwright

## Introduction
Web scraping is the automated extraction of data from websites. Python is the language of choice for web scraping due to its rich ecosystem of libraries that handle HTTP requests, HTML parsing, JavaScript rendering, and data extraction. The core tools include `requests` for fetching pages, `BeautifulSoup` for parsing HTML/XML, `Selenium` for automating browsers with JavaScript execution, and `Playwright` for modern, fast browser automation. Choosing the right tool depends on the complexity of the target site: static HTML can be scraped with `requests` and `BeautifulSoup`, while dynamic single-page applications (SPAs) require `Selenium` or `Playwright`.

## BeautifulSoup

### What It Is
BeautifulSoup is a Python library for parsing HTML and XML documents. It creates a parse tree that makes it easy to navigate, search, and modify the document structure. BeautifulSoup handles malformed HTML gracefully, making it ideal for real-world web scraping where pages often have inconsistent markup.

### Why It Is Important
BeautifulSoup transforms messy, real-world HTML into a navigable tree structure that can be queried with Python idioms (tag names, attributes, CSS selectors). Without it, extracting data from HTML would require complicated regex patterns that are brittle and error-prone.

### How It Works Internally
BeautifulSoup wraps an HTML parser (lxml, html.parser, or html5lib) to build a document object model. The parser reads the HTML string and creates a tree of `Tag` and `NavigableString` objects. BeautifulSoup then provides methods like `find()`, `find_all()`, `select()` (CSS selectors), and attribute access to query this tree. The library stores metadata about the original document encoding and handles character encoding automatically.

### Syntax
```python
from bs4 import BeautifulSoup

html = """
<html>
    <body>
        <h1>Main Title</h1>
        <div class="content">
            <p class="text">First paragraph</p>
            <p class="text highlight">Second paragraph</p>
            <a href="https://example.com">Link</a>
        </div>
    </body>
</html>
"""

soup = BeautifulSoup(html, "html.parser")

# Tag access
print(soup.h1.text)           # "Main Title"
print(soup.a["href"])         # "https://example.com"

# find() and find_all()
first_p = soup.find("p")
all_ps = soup.find_all("p")
by_class = soup.find_all("p", class_="text")

# CSS selectors
items = soup.select("div.content p.text")
first_item = soup.select_one("a")

# Navigating the tree
parent = soup.p.parent
next_sibling = soup.p.find_next_sibling("p")
```

### Beginner Examples
```python
import requests
from bs4 import BeautifulSoup

# Fetch and parse a news page
url = "https://news.ycombinator.com/"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")

# Extract headlines
for item in soup.find_all("span", class_="titleline"):
    link = item.find("a")
    title = link.text
    href = link["href"]
    print(f"{title}: {href}")
```

### Intermediate Examples
```python
import requests
from bs4 import BeautifulSoup
import csv
import time

def scrape_products(url):
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"}
    response = requests.get(url, headers=headers)
    soup = BeautifulSoup(response.content, "lxml")

    products = []
    for product in soup.select("div.product-card"):
        name = product.select_one("h2.product-name")
        price = product.select_one("span.price")
        rating = product.select_one("div.rating")

        if name and price:
            products.append({
                "name": name.text.strip(),
                "price": price.text.strip(),
                "rating": rating.text.strip() if rating else "N/A",
            })

    return products

# Pagination with rate limiting
all_products = []
for page in range(1, 6):
    url = f"https://example-shop.com/products?page={page}"
    products = scrape_products(url)
    all_products.extend(products)
    time.sleep(2)  # Be respectful

# Save to CSV
with open("products.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "price", "rating"])
    writer.writeheader()
    writer.writerows(all_products)
```

## requests for scraping

### What It Is
The `requests` library fetches web pages programmatically, handling HTTP methods, headers, cookies, sessions, and SSL verification. For scraping, it is typically used with BeautifulSoup: `requests` gets the HTML, BeautifulSoup parses it.

### Why It Is Important
Requests provides the HTTP transport layer for scraping. It handles cookies (sessions), redirects, custom headers (User-Agent, Referer), authentication, and timeouts. Proper use of requests is essential for mimicking browser behavior and avoiding blocks.

### Advanced Examples
```python
import requests
from bs4 import BeautifulSoup
import time
from urllib.parse import urljoin

session = requests.Session()
session.headers.update({
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    "Accept-Language": "en-US,en;q=0.9",
    "Accept-Encoding": "gzip, deflate",
})

# Handle cookies and redirects
login_url = "https://example.com/login"
session.post(login_url, data={"username": "user", "password": "pass"})

# Scrape authenticated content
dashboard = session.get("https://example.com/dashboard")
soup = BeautifulSoup(dashboard.text, "html.parser")

# Handle pagination with session
def scrape_all_pages(base_url):
    page = 1
    all_data = []
    while True:
        url = f"{base_url}?page={page}"
        response = session.get(url)
        if response.status_code != 200:
            break
        soup = BeautifulSoup(response.text, "html.parser")
        items = soup.select("div.item")
        if not items:
            break
        all_data.extend(items)
        page += 1
        time.sleep(1)
    return all_data

# Retry with exponential backoff
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

retry_strategy = Retry(
    total=3,
    backoff_factor=1,
    status_forcelist=[429, 500, 502, 503, 504],
)
adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("https://", adapter)
session.mount("http://", adapter)
```

## Selenium

### What It Is
Selenium is a browser automation framework that controls a real web browser (Chrome, Firefox, Edge, Safari) programmatically. It can execute JavaScript, interact with dynamic content, handle complex user interactions, and take screenshots. Selenium is essential for scraping JavaScript-heavy websites that render content asynchronously.

### Why It Is Important
Modern websites increasingly rely on JavaScript frameworks (React, Angular, Vue) that render content dynamically. `requests` + BeautifulSoup cannot execute JavaScript, so Selenium is necessary for such sites. It also handles CAPTCHAs (with third-party services), complex navigation flows, and file downloads.

### How It Works Internally
Selenium communicates with browser drivers (ChromeDriver, GeckoDriver) via the WebDriver protocol. The driver translates Selenium commands into browser-native automation. Selenium creates a real browser instance (headless or visible), loads pages (including JavaScript execution), and exposes methods to query the DOM, click elements, fill forms, and extract data.

### Syntax
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Setup
options = webdriver.ChromeOptions()
options.add_argument("--headless")  # Run without GUI
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
driver = webdriver.Chrome(options=options)

# Navigate
driver.get("https://example.com")

# Find elements
element = driver.find_element(By.ID, "main")
elements = driver.find_elements(By.CLASS_NAME, "item")
element = driver.find_element(By.CSS_SELECTOR, "div.content p")
element = driver.find_element(By.XPATH, "//div[@class='content']")

# Interact
driver.find_element(By.NAME, "q").send_keys("search term")
driver.find_element(By.ID, "submit").click()

# Wait for elements
wait = WebDriverWait(driver, 10)
element = wait.until(EC.presence_of_element_located((By.ID, "dynamic-content")))

# Extract data
html = driver.page_source
text = element.text
attribute = element.get_attribute("href")

driver.quit()
```

### Advanced Examples
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait, Select
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
import json

def scrape_react_app():
    options = Options()
    options.add_argument("--headless")
    options.add_argument("--window-size=1920,1080")
    options.add_experimental_option("prefs", {
        "profile.default_content_setting_values.notifications": 2,
        "credentials_enable_service": False,
    })
    driver = webdriver.Chrome(options=options)

    try:
        driver.get("https://react-ecommerce.example.com/products")
        wait = WebDriverWait(driver, 15)

        # Wait for dynamic content to load
        wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, ".product-grid")))

        # Scroll to load lazy content
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(2)

        # Infinite scroll: load more
        for _ in range(5):
            driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
            time.sleep(1)

        # Extract product data
        products = []
        for card in driver.find_elements(By.CSS_SELECTOR, ".product-card"):
            products.append({
                "name": card.find_element(By.CSS_SELECTOR, ".name").text,
                "price": card.find_element(By.CSS_SELECTOR, ".price").text,
                "rating": card.find_element(By.CSS_SELECTOR, ".rating").get_attribute("data-rating"),
                "url": card.find_element(By.CSS_SELECTOR, "a").get_attribute("href"),
            })

        return products

    finally:
        driver.quit()

# Handle iframes
def scrape_iframe(url):
    driver = webdriver.Chrome()
    driver.get(url)
    iframe = driver.find_element(By.TAG_NAME, "iframe")
    driver.switch_to.frame(iframe)
    content = driver.find_element(By.TAG_NAME, "body").text
    driver.switch_to.default_content()
    driver.quit()
    return content
```

## Playwright

### What It Is
Playwright is a modern browser automation library developed by Microsoft that supports Chromium, Firefox, and WebKit through a single API. It is faster, more reliable, and has better API design than Selenium. Playwright supports auto-waiting (automatic waiting for elements to be ready), network interception, mobile emulation, and multiple browser contexts.

### Why It Is Important
Playwright addresses many of Selenium's pain points: flaky waits, slow execution, complex setup. Its auto-waiting mechanism reduces test flakiness. Network interception enables powerful patterns like blocking images (for speed), mocking API responses, and recording network traffic. Playwright also generates screenshots, PDFs, and traces for debugging.

### How It Works Internally
Playwright communicates with browser instances via the Chrome DevTools Protocol (CDP) for Chromium, or equivalent protocols for Firefox and WebKit. Each browser runs as a separate process. Playwright's API is fully async and uses event-driven patterns. The auto-waiting system checks element state (visible, stable, enabled) before performing actions, eliminating the need for explicit waits.

### Syntax
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()

    # Navigate
    page.goto("https://example.com")

    # Query elements
    title = page.query_selector("h1")
    items = page.query_selector_all(".item")

    # Extract
    text = title.inner_text()
    href = page.get_attribute("a", "href")

    # Fill forms
    page.fill("input#search", "query")
    page.click("button#submit")

    # Wait for navigation
    with page.expect_navigation():
        page.click("a#next-page")

    # Screenshot
    page.screenshot(path="page.png")

    browser.close()
```

### Advanced Examples
```python
from playwright.sync_api import sync_playwright
import json
import re

def scrape_with_playwright():
    with sync_playwright() as p:
        browser = p.chromium.launch(
            headless=True,
            args=["--disable-blink-features=AutomationControlled"]
        )

        # Stealth: mimic real browser
        context = browser.new_context(
            user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
            viewport={"width": 1920, "height": 1080},
            locale="en-US",
            timezone_id="America/New_York",
            permissions=["geolocation"],
            geolocation={"latitude": 40.7128, "longitude": -74.0060},
        )

        page = context.new_page()

        # Block unnecessary resources for speed
        page.route("**/*.{png,jpg,jpeg,gif,svg,css,font}", lambda route: route.abort())

        # Intercept network requests
        api_responses = []
        page.on("response", lambda response: (
            api_responses.append(response.json())
            if "/api/" in response.url
            else None
        ))

        # Navigate and wait for network idle
        page.goto("https://spa-website.example.com", wait_until="networkidle")

        # Infinite scroll with network monitoring
        for _ in range(5):
            page.evaluate("window.scrollTo(0, document.body.scrollHeight)")
            page.wait_for_timeout(1000)

        # Extract all text content
        data = page.evaluate("""
            () => {
                return Array.from(document.querySelectorAll('.post')).map(post => ({
                    title: post.querySelector('.title')?.innerText,
                    content: post.querySelector('.content')?.innerText,
                    date: post.querySelector('.date')?.getAttribute('datetime'),
                }));
            }
        """)

        # Export to file
        with open("scraped_data.json", "w") as f:
            json.dump(data, f, indent=2)

        browser.close()
        return data

# Async Playwright
import asyncio
from playwright.async_api import async_playwright

async def scrape_async():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()
        await page.goto("https://example.com")
        content = await page.content()
        await browser.close()
        return content

# result = asyncio.run(scrape_async())

# Multi-page scraping with contexts
def scrape_multiple_sites(urls):
    with sync_playwright() as p:
        browser = p.chromium.launch()
        results = []
        for url in urls:
            context = browser.new_context()
            page = context.new_page()
            page.goto(url)
            results.append(page.title())
            context.close()
        browser.close()
        return results
```

### Real-World Use Cases
- **Price monitoring**: Track competitor pricing across e-commerce sites.
- **Lead generation**: Extract contact information from business directories.
- **Content aggregation**: News aggregation, job listings, real estate listings.
- **Social media analytics**: Extract posts, comments, and engagement metrics.
- **Research**: Academic data collection, sentiment analysis, market research.

### Common Mistakes
- Not respecting `robots.txt` and terms of service.
- Scraping too fast without rate limiting (getting IP banned).
- Not rotating User-Agent headers (pattern detection).
- Ignoring dynamic content (needs browser automation).
- Not handling JavaScript-rendered pages (using only requests+BeautifulSoup).
- Not using proper error handling for network failures and CAPTCHAs.

### Best Practices
- Respect `robots.txt` and website terms of service.
- Implement rate limiting (time.sleep between requests).
- Use rotating User-Agent strings and proxy rotation when needed.
- Handle errors gracefully with retries and backoff.
- Cache downloaded pages to avoid redundant requests.
- Use headless browsers only when JavaScript execution is needed.
- Extract data as close to the source as possible (APIs > scraping).

### Performance Considerations
- Playwright is 2-3x faster than Selenium for most tasks.
- Use `page.route()` in Playwright to block images, fonts, CSS (saves 50-80% bandwidth).
- Use `wait_until="networkidle"` in Playwright for complete page loads.
- Use connection pooling (requests.Session) for HTTP scraping.
- Parallelize with `ThreadPoolExecutor` or `asyncio` for multi-page scraping.

### Interview Questions
1. What is the difference between BeautifulSoup and Selenium?
2. When would you use Playwright over Selenium?
3. How do you handle pagination in web scraping?
4. What techniques can you use to avoid being blocked while scraping?
5. How do you scrape JavaScript-rendered content?
6. Explain the concept of CSS selectors in BeautifulSoup.
7. What is auto-waiting in Playwright and how does it help?

### Coding Challenges
1. **Price Tracker**: Build a price monitoring script that scrapes an e-commerce product page daily, stores prices in a database, and alerts on price drops.
2. **Article Extractor**: Create a tool that extracts article title, author, date, and body from news websites using Playwright.
3. **Social Media Scraper**: Scrape a social media profile page to extract posts, likes, comments, and followers (simulate infinite scroll).

### Related Topics
- Scrapy (framework for large-scale scraping)
- lxml (fast HTML/XML parser, alternative to BeautifulSoup)
- Requests-HTML (library combining requests with JS rendering)
- Proxy rotation services (ScraperAPI, Zyte)
- Web scraping legal and ethical considerations
