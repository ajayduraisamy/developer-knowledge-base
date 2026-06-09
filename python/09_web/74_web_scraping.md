# Web Scraping - BeautifulSoup, requests, Selenium, Playwright

## Introduction

Web scraping is the automated process of extracting data from websites. Python is the most popular language for web scraping due to its rich ecosystem of libraries — BeautifulSoup4 for HTML parsing, lxml for fast XML/HTML processing, Selenium and Playwright for JavaScript-rendered content, Scrapy for large-scale crawling, and requests/httpx for HTTP fetching. Web scraping enables data collection from websites that do not provide official APIs, powering use cases from price monitoring to research datasets.

## Why It Is Important

Web scraping is essential when data is only available through web interfaces and not via APIs. It enables competitive intelligence (price monitoring, product cataloging), research (academic datasets, news aggregation), automation (form filling, report generation), and integration (legacy system data extraction). Understanding ethical scraping practices — respecting robots.txt, implementing rate limiting, identifying yourself via User-Agent — is crucial to avoid legal issues and IP bans. Modern web scraping also requires handling JavaScript-rendered content, which is where Selenium and Playwright come in.

## Syntax

### Basic BeautifulSoup Scraping

```python
import requests
from bs4 import BeautifulSoup

response = requests.get("https://example.com")
soup = BeautifulSoup(response.text, "html.parser")

title = soup.find("h1").text
items = soup.find_all("div", class_="item")
links = soup.select("a.link-class")
```

### Using Playwright for JS Pages

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    content = page.content()
    browser.close()
```

## Examples

### Simple HTML Parser

```python
import requests
from bs4 import BeautifulSoup

url = "https://quotes.toscrape.com"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")

for quote in soup.find_all("div", class_="quote"):
    text = quote.find("span", class_="text").text
    author = quote.find("small", class_="author").text
    print(f"{author}: {text}")
```

### CSS Selectors

```python
from bs4 import BeautifulSoup

html = """
<html><body>
<div class="container">
    <ul id="items">
        <li class="item active" data-id="1">First</li>
        <li class="item" data-id="2">Second</li>
        <li class="item inactive" data-id="3">Third</li>
    </ul>
</div>
</body></html>
"""

soup = BeautifulSoup(html, "html.parser")

items = soup.select("ul#items li.item")
active = soup.select("li.active")
first_item = soup.select_one("li[data-id='1']")

print([i.text for i in items])
print([i.text for i in active])
print(first_item.text)
```

## Beginner Examples

### 1. Extract All Links from a Page

```python
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin

url = "https://quotes.toscrape.com"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")

links = []
for link in soup.find_all("a", href=True):
    href = link["href"]
    full_url = urljoin(url, href)
    links.append({"text": link.text.strip(), "url": full_url})

for link in links[:10]:
    print(f"{link['text']:30} -> {link['url']}")
```

### 2. Extract Table Data

```python
import requests
from bs4 import BeautifulSoup

html = """
<table>
    <thead>
        <tr><th>Name</th><th>Age</th><th>City</th></tr>
    </thead>
    <tbody>
        <tr><td>Alice</td><td>30</td><td>NYC</td></tr>
        <tr><td>Bob</td><td>25</td><td>LA</td></tr>
        <tr><td>Charlie</td><td>35</td><td>Chicago</td></tr>
    </tbody>
</table>
"""

soup = BeautifulSoup(html, "html.parser")
table = soup.find("table")

headers = [th.text for th in table.find("thead").find_all("th")]
rows = []
for tr in table.find("tbody").find_all("tr"):
    row = [td.text for td in tr.find_all("td")]
    rows.append(dict(zip(headers, row)))

for row in rows:
    print(row)
```

### 3. Scrape Multiple Pages

```python
import requests
from bs4 import BeautifulSoup
import time

base_url = "https://quotes.toscrape.com/page/{}/"
all_quotes = []
page = 1

while True:
    url = base_url.format(page)
    response = requests.get(url)

    if "No quotes found!" in response.text or response.status_code != 200:
        break

    soup = BeautifulSoup(response.text, "html.parser")
    quotes = soup.find_all("div", class_="quote")

    if not quotes:
        break

    for quote in quotes:
        text = quote.find("span", class_="text").text
        author = quote.find("small", class_="author").text
        tags = [tag.text for tag in quote.find_all("a", class_="tag")]
        all_quotes.append({"text": text, "author": author, "tags": tags})

    print(f"Scraped page {page}, got {len(quotes)} quotes")
    page += 1
    time.sleep(1)

print(f"Total quotes scraped: {len(all_quotes)}")
```

### 4. Save Data to CSV

```python
import csv
import requests
from bs4 import BeautifulSoup

url = "https://quotes.toscrape.com"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")

data = []
for quote in soup.find_all("div", class_="quote"):
    text = quote.find("span", class_="text").text
    author = quote.find("small", class_="author").text
    tags = ", ".join([tag.text for tag in quote.find_all("a", class_="tag")])
    data.append({"text": text, "author": author, "tags": tags})

with open("quotes.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["text", "author", "tags"])
    writer.writeheader()
    writer.writerows(data)

print(f"Saved {len(data)} quotes to quotes.csv")
```

### 5. Extract Attributes

```python
import requests
from bs4 import BeautifulSoup

url = "https://quotes.toscrape.com"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")

for quote in soup.find_all("div", class_="quote"):
    text_elem = quote.find("span", class_="text")
    author_elem = quote.find("small", class_="author")
    tags = quote.find_all("a", class_="tag")

    print(f"Text: {text_elem.text}")
    print(f"Author: {author_elem.text}")
    print(f"Tags: {[t.text for t in tags]}")
    print("---")
```

## Intermediate Examples

### 1. Scrape with lxml Parser

```python
import requests
from lxml import html

url = "https://quotes.toscrape.com"
response = requests.get(url)
tree = html.fromstring(response.content)

quotes = tree.xpath("//div[@class='quote']")
for quote in quotes:
    text = quote.xpath(".//span[@class='text']/text()")[0]
    author = quote.xpath(".//small[@class='author']/text()")[0]
    tags = quote.xpath(".//a[@class='tag']/text()")
    print(f"{author}: {text}")
    print(f"Tags: {tags}")
```

### 2. Scrape JS with Selenium

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options

options = Options()
options.add_argument("--headless")

driver = webdriver.Chrome(options=options)
driver.get("https://quotes.toscrape.com/js/")

wait = WebDriverWait(driver, 10)
quotes = wait.until(EC.presence_of_all_elements_located((By.CLASS_NAME, "quote")))

data = []
for quote in quotes:
    text = quote.find_element(By.CLASS_NAME, "text").text
    author = quote.find_element(By.CLASS_NAME, "author").text
    tags = [tag.text for tag in quote.find_elements(By.CLASS_NAME, "tag")]
    data.append({"text": text, "author": author, "tags": tags})

driver.quit()

for item in data[:3]:
    print(f"{item['author']}: {item['text'][:50]}")
```

### 3. Scrape with Playwright

```python
from playwright.sync_api import sync_playwright

def scrape_with_playwright(url):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        page.goto(url, wait_until="networkidle")

        quotes = page.query_selector_all(".quote")
        data = []
        for quote in quotes:
            text = quote.query_selector(".text").inner_text()
            author = quote.query_selector(".author").inner_text()
            tags = [t.inner_text() for t in quote.query_selector_all(".tag")]
            data.append({"text": text, "author": author, "tags": tags})

        browser.close()
        return data

data = scrape_with_playwright("https://quotes.toscrape.com/js/")
print(f"Scraped {len(data)} quotes")
```

### 4. Handle Infinite Scroll

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
import time

options = Options()
options.add_argument("--headless")
driver = webdriver.Chrome(options=options)
driver.get("https://quotes.toscrape.com/scroll/")

data = []
last_height = driver.execute_script("return document.body.scrollHeight")

while len(data) < 50:
    quotes = driver.find_elements(By.CLASS_NAME, "quote")
    for quote in quotes[len(data):]:
        text = quote.find_element(By.CLASS_NAME, "text").text
        author = quote.find_element(By.CLASS_NAME, "author").text
        data.append({"text": text, "author": author})

    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    time.sleep(2)
    new_height = driver.execute_script("return document.body.scrollHeight")
    if new_height == last_height:
        break
    last_height = new_height

driver.quit()
print(f"Scraped {len(data)} quotes via infinite scroll")
```

### 5. Respect robots.txt

```python
from urllib.robotparser import RobotFileParser
from urllib.parse import urlparse

def can_scrape(url, user_agent="MyScraperBot"):
    parsed = urlparse(url)
    robots_url = f"{parsed.scheme}://{parsed.netloc}/robots.txt"

    rp = RobotFileParser()
    rp.set_url(robots_url)
    try:
        rp.read()
        return rp.can_fetch(user_agent, url)
    except Exception as e:
        print(f"Could not read robots.txt: {e}")
        return True

urls = [
    "https://quotes.toscrape.com/",
    "https://quotes.toscrape.com/admin/",
    "https://example.com/"
]

for url in urls:
    allowed = can_scrape(url)
    print(f"{'[OK]' if allowed else '[BLOCKED]'} {url}")
```

### 6. Rate Limiting with Delay

```python
import requests
from bs4 import BeautifulSoup
import time
import random

class PoliteScraper:
    def __init__(self, min_delay=1, max_delay=3, user_agent="PoliteScraper/1.0"):
        self.min_delay = min_delay
        self.max_delay = max_delay
        self.session = requests.Session()
        self.session.headers.update({"User-Agent": user_agent})
        self.last_request_time = 0

    def _wait(self):
        elapsed = time.time() - self.last_request_time
        delay = random.uniform(self.min_delay, self.max_delay)
        if elapsed < delay:
            time.sleep(delay - elapsed)

    def get(self, url):
        self._wait()
        self.last_request_time = time.time()
        response = self.session.get(url)
        response.raise_for_status()
        return response

    def scrape_quotes(self, base_url, pages=5):
        all_quotes = []
        for page in range(1, pages + 1):
            url = f"{base_url}/page/{page}/"
            response = self.get(url)
            soup = BeautifulSoup(response.text, "html.parser")

            for quote in soup.find_all("div", class_="quote"):
                text = quote.find("span", class_="text").text
                author = quote.find("small", class_="author").text
                tags = [tag.text for tag in quote.find_all("a", class_="tag")]
                all_quotes.append({"text": text, "author": author, "tags": tags})

            print(f"Page {page}: scraped")
        return all_quotes

scraper = PoliteScraper(min_delay=1, max_delay=2)
quotes = scraper.scrape_quotes("https://quotes.toscrape.com", pages=3)
print(f"Total: {len(quotes)} quotes")
```

### 7. Proxy Rotation

```python
import requests
from bs4 import BeautifulSoup
import random

class ProxyScraper:
    def __init__(self, proxies=None):
        self.session = requests.Session()
        self.proxies = proxies or [
            {"http": "http://proxy1:8080", "https": "http://proxy1:8080"},
            {"http": "http://proxy2:8080", "https": "http://proxy2:8080"},
        ]
        self.current_proxy = 0

    def get_next_proxy(self):
        proxy = self.proxies[self.current_proxy]
        self.current_proxy = (self.current_proxy + 1) % len(self.proxies)
        return proxy

    def get(self, url):
        proxy = self.get_next_proxy()
        try:
            response = self.session.get(url, proxies=proxy, timeout=10)
            return response
        except requests.exceptions.RequestException as e:
            print(f"Proxy {proxy} failed: {e}")
            return self.get(url)

scraper = ProxyScraper()
try:
    response = scraper.get("https://httpbin.org/ip")
    print(f"IP: {response.json()}")
except Exception as e:
    print(f"Proxy error: {e}")
```

### 8. Form Authentication for Scraping

```python
import requests
from bs4 import BeautifulSoup

class AuthenticatedScraper:
    def __init__(self, login_url):
        self.session = requests.Session()
        self.login_url = login_url

    def login(self, username, password):
        response = self.session.get(self.login_url)
        soup = BeautifulSoup(response.text, "html.parser")
        csrf = soup.find("input", {"name": "csrf_token"})
        csrf_token = csrf["value"] if csrf else None

        form_data = {"username": username, "password": password}
        if csrf_token:
            form_data["csrf_token"] = csrf_token

        response = self.session.post(self.login_url, data=form_data)
        return response.ok

    def get(self, url):
        return self.session.get(url)

    def scrape_protected(self, url):
        response = self.get(url)
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, "html.parser")
            return soup
        return None

scraper = AuthenticatedScraper("https://httpbin.org/post")
login_success = scraper.login("user", "pass")
print(f"Login: {'Success' if login_success else 'Failed'}")
```

### 9. Scrapy Integration (Basic Spider)

```python
import scrapy
from scrapy.crawler import CrawlerProcess

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = ["https://quotes.toscrape.com/page/1/"]

    def parse(self, response):
        for quote in response.css("div.quote"):
            yield {
                "text": quote.css("span.text::text").get(),
                "author": quote.css("small.author::text").get(),
                "tags": quote.css("a.tag::text").getall(),
            }

        next_page = response.css("li.next a::attr(href)").get()
        if next_page:
            yield response.follow(next_page, self.parse)

process = CrawlerProcess(settings={
    "FEEDS": {"quotes.json": {"format": "json"}},
    "LOG_LEVEL": "INFO"
})
process.crawl(QuotesSpider)
process.start()
```

### 10. Full Scraping Pipeline

```python
import requests
from bs4 import BeautifulSoup
import csv
import json
import time
import os
from urllib.parse import urljoin, urlparse

class WebScraperPipeline:
    def __init__(self, base_url, output_dir="output", delay=1):
        self.base_url = base_url
        self.output_dir = output_dir
        self.delay = delay
        self.session = requests.Session()
        self.session.headers.update({"User-Agent": "Mozilla/5.0"})
        self.visited_urls = set()
        os.makedirs(output_dir, exist_ok=True)

    def fetch(self, url):
        if url in self.visited_urls:
            return None
        self.visited_urls.add(url)
        time.sleep(self.delay)
        try:
            response = self.session.get(url, timeout=10)
            response.raise_for_status()
            return response
        except requests.exceptions.RequestException as e:
            print(f"Error fetching {url}: {e}")
            return None

    def parse_links(self, soup, current_url):
        links = []
        for a in soup.find_all("a", href=True):
            href = a["href"]
            full_url = urljoin(current_url, href)
            parsed = urlparse(full_url)
            if parsed.netloc == urlparse(self.base_url).netloc:
                links.append(full_url)
        return list(set(links))

    def extract_data(self, soup):
        return {
            "title": soup.title.text.strip() if soup.title else "",
            "h1_tags": [h.text.strip() for h in soup.find_all("h1")],
            "paragraphs": [p.text.strip() for p in soup.find_all("p")[:5]],
            "links": len(soup.find_all("a", href=True))
        }

    def save_json(self, data, filename):
        with open(os.path.join(self.output_dir, filename), "w", encoding="utf-8") as f:
            json.dump(data, f, indent=2)

    def crawl(self, start_url, max_pages=10):
        to_visit = [start_url]
        all_data = []

        while to_visit and len(self.visited_urls) < max_pages:
            url = to_visit.pop(0)
            response = self.fetch(url)
            if not response:
                continue

            soup = BeautifulSoup(response.text, "html.parser")
            page_data = self.extract_data(soup)
            page_data["url"] = url
            all_data.append(page_data)
            print(f"[{len(self.visited_urls)}/{max_pages}] Scraped: {url}")

            new_links = self.parse_links(soup, url)
            to_visit.extend([l for l in new_links if l not in self.visited_urls])

        self.save_json(all_data, "scraped_data.json")
        print(f"Crawl complete. {len(all_data)} pages saved.")
        return all_data

scraper = WebScraperPipeline("https://quotes.toscrape.com", delay=0.5)
data = scraper.crawl("https://quotes.toscrape.com", max_pages=5)
```

## Advanced Examples

### 1. Distributed Scraping with Queue

```python
import requests
from bs4 import BeautifulSoup
from queue import Queue
from threading import Thread, Lock
import time

class DistributedScraper:
    def __init__(self, base_url, num_workers=5, delay=0.5):
        self.base_url = base_url
        self.num_workers = num_workers
        self.delay = delay
        self.url_queue = Queue()
        self.results = []
        self.lock = Lock()
        self.visited = set()
        self.session = requests.Session()

    def worker(self):
        while True:
            try:
                url = self.url_queue.get(timeout=5)
            except:
                break

            if url in self.visited:
                self.url_queue.task_done()
                continue

            with self.lock:
                self.visited.add(url)

            try:
                time.sleep(self.delay)
                response = self.session.get(url, timeout=10)
                soup = BeautifulSoup(response.text, "html.parser")

                with self.lock:
                    self.results.append({
                        "url": url,
                        "title": soup.title.text.strip() if soup.title else "",
                        "status": response.status_code
                    })

                links = soup.find_all("a", href=True)
                for link in links:
                    href = link["href"]
                    if href.startswith("/"):
                        full_url = self.base_url + href
                        with self.lock:
                            if full_url not in self.visited:
                                self.url_queue.put(full_url)
            except Exception as e:
                print(f"Error scraping {url}: {e}")
            finally:
                self.url_queue.task_done()

    def scrape(self, start_urls, max_urls=50):
        for url in start_urls:
            self.url_queue.put(url)

        threads = []
        for _ in range(self.num_workers):
            t = Thread(target=self.worker, daemon=True)
            t.start()
            threads.append(t)

        self.url_queue.join()

        for t in threads:
            t.join(timeout=1)

        return self.results[:max_urls]

scraper = DistributedScraper("https://quotes.toscrape.com", num_workers=3, delay=0.3)
results = scraper.scrape(["https://quotes.toscrape.com"], max_urls=10)
print(f"Scraped {len(results)} pages")
```

### 2. Dynamic Content with Playwright

```python
from playwright.sync_api import sync_playwright

def scrape_dynamic_with_stealth(url):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        context = browser.new_context(
            user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
            viewport={"width": 1920, "height": 1080},
            locale="en-US"
        )
        page = context.new_page()

        page.route("**/*.jpg", lambda route: route.abort())
        page.route("**/*.png", lambda route: route.abort())

        page.goto(url, wait_until="networkidle", timeout=30000)
        page.evaluate("window.scrollTo(0, document.body.scrollHeight)")
        page.wait_for_timeout(1000)

        data = page.evaluate("""() => {
            const items = [];
            document.querySelectorAll('.quote').forEach(el => {
                items.push({
                    text: el.querySelector('.text')?.innerText,
                    author: el.querySelector('.author')?.innerText,
                    tags: Array.from(el.querySelectorAll('.tag')).map(t => t.innerText)
                });
            });
            return items;
        }""")

        browser.close()
        return data

result = scrape_dynamic_with_stealth("https://quotes.toscrape.com/js/")
print(f"Items: {len(result)}")
```

### 3. Image Scraper

```python
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin
import os
import hashlib

class ImageScraper:
    def __init__(self, output_dir="images"):
        self.output_dir = output_dir
        self.session = requests.Session()
        self.session.headers.update({"User-Agent": "ImageScraper/1.0"})
        os.makedirs(output_dir, exist_ok=True)

    def extract_images(self, url):
        response = self.session.get(url)
        soup = BeautifulSoup(response.text, "html.parser")
        images = []
        for img in soup.find_all("img", src=True):
            src = img["src"]
            full_url = urljoin(url, src)
            alt = img.get("alt", "")
            images.append({"url": full_url, "alt": alt})
        return images

    def download_image(self, image_url):
        try:
            response = self.session.get(image_url, timeout=10)
            response.raise_for_status()
            ext = os.path.splitext(image_url)[1] or ".jpg"
            filename = hashlib.md5(image_url.encode()).hexdigest() + ext
            filepath = os.path.join(self.output_dir, filename)
            with open(filepath, "wb") as f:
                f.write(response.content)
            return filepath, len(response.content)
        except Exception as e:
            return None, 0

    def scrape(self, url, max_images=20):
        images = self.extract_images(url)
        print(f"Found {len(images)} images on {url}")
        downloaded = 0
        for img in images[:max_images]:
            filepath, size = self.download_image(img["url"])
            if filepath:
                downloaded += 1
                print(f"  [{downloaded}] Saved {filepath} ({size} bytes)")
        return downloaded

scraper = ImageScraper()
downloaded = scraper.scrape("https://quotes.toscrape.com", max_images=5)
print(f"Downloaded {downloaded} images")
```

### 4. Scraping with Retry and Checkpoint

```python
import requests
from bs4 import BeautifulSoup
import time
import json
import os

class RobustScraper:
    def __init__(self):
        self.session = requests.Session()

    def fetch_with_retry(self, url, max_attempts=3, delay=2):
        for attempt in range(max_attempts):
            try:
                response = self.session.get(url, timeout=10)
                response.raise_for_status()
                return response
            except requests.exceptions.RequestException as e:
                if attempt < max_attempts - 1:
                    sleep_time = delay * (2 ** attempt)
                    print(f"Attempt {attempt + 1} failed: {e}. Retrying in {sleep_time}s...")
                    time.sleep(sleep_time)
                else:
                    raise

    def scrape_with_checkpoint(self, urls, checkpoint_file="checkpoint.json"):
        completed = []
        if os.path.exists(checkpoint_file):
            with open(checkpoint_file) as f:
                completed = json.load(f)

        results = list(completed)
        for url in urls:
            if url in [r["url"] for r in results]:
                print(f"Skipping already scraped: {url}")
                continue
            try:
                response = self.fetch_with_retry(url)
                soup = BeautifulSoup(response.text, "html.parser")
                page_data = {
                    "url": url,
                    "title": soup.title.text.strip() if soup.title else "",
                    "status": response.status_code
                }
                results.append(page_data)
                print(f"Scraped: {url}")
                with open(checkpoint_file, "w") as f:
                    json.dump(results, f)
            except Exception as e:
                print(f"Failed to scrape {url}: {e}")
                continue
        return results

scraper = RobustScraper()
urls = [f"https://quotes.toscrape.com/page/{i}/" for i in range(1, 6)]
results = scraper.scrape_with_checkpoint(urls)
print(f"Scraped {len(results)} pages")
```

### 5. Async Web Scraping

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup
import time

class AsyncWebScraper:
    def __init__(self, concurrency=10, delay=0.5):
        self.concurrency = concurrency
        self.delay = delay
        self.semaphore = asyncio.Semaphore(concurrency)

    async def fetch(self, session, url):
        async with self.semaphore:
            try:
                async with session.get(url, timeout=aiohttp.ClientTimeout(total=30)) as response:
                    await asyncio.sleep(self.delay)
                    html = await response.text()
                    return url, html, response.status
            except Exception as e:
                return url, None, str(e)

    async def scrape_all(self, urls):
        async with aiohttp.ClientSession(headers={"User-Agent": "AsyncScraper/1.0"}) as session:
            tasks = [self.fetch(session, url) for url in urls]
            results = await asyncio.gather(*tasks)
            scraped = []
            for url, html, status in results:
                if html:
                    soup = BeautifulSoup(html, "html.parser")
                    scraped.append({
                        "url": url,
                        "title": soup.title.text.strip() if soup.title else "",
                        "status": status
                    })
            return scraped

    def scrape(self, urls):
        return asyncio.run(self.scrape_all(urls))

urls = [f"https://quotes.toscrape.com/page/{i}/" for i in range(1, 11)]
start = time.time()
scraper = AsyncWebScraper(concurrency=5, delay=0.2)
results = scraper.scrape(urls)
elapsed = time.time() - start
print(f"Scraped {len(results)} pages in {elapsed:.2f}s")
```

## Real-World Use Cases

- **Price Monitoring**: Track competitor pricing across e-commerce sites for dynamic pricing
- **News Aggregation**: Collect headlines, summaries, and articles from multiple news sources
- **Real Estate Listings**: Scrape property details, prices, and images from listing sites
- **Job Boards**: Aggregate job postings from multiple platforms
- **Social Media Analytics**: Extract public posts, engagement metrics, and trends
- **Product Reviews**: Collect customer reviews and ratings for sentiment analysis
- **Research Datasets**: Build structured datasets from web sources for ML models
- **SEO Monitoring**: Track search rankings, meta descriptions, and backlinks
- **Lead Generation**: Extract business contact information from directories
- **Compliance Monitoring**: Track regulatory changes from government sites

## Common Mistakes

- **Ignoring robots.txt** — scraping disallowed paths, risking legal action
- **No rate limiting** — flooding servers with requests, getting IP banned
- **Hardcoded selectors** — breaking when websites update their HTML structure
- **Not handling JavaScript** — missing content rendered dynamically by JS
- **Blocked by anti-bot measures** — not rotating User-Agent, IP, or cookies
- **Scraping without error handling** — crashing on network errors or page changes
- **Storing data without deduplication** — saving duplicate entries
- **Not respecting copyright** — scraping and republishing copyrighted content
- **Ignoring pagination** — only scraping the first page of results
- **Not encoding properly** — garbled text from wrong charset handling

## Best Practices

- Always check and respect robots.txt before scraping
- Implement polite delays between requests (1-5 seconds)
- Rotate User-Agent strings to avoid detection
- Use sessions for cookie and connection reuse
- Handle JavaScript-rendered content with Playwright or Selenium when needed
- Implement comprehensive error handling and retry logic
- Use checkpoint/resume for long-running scrapes
- Cache responses to avoid repeated requests
- Validate and sanitize extracted data
- Consider using official APIs when available instead of scraping

## Interview Questions

**Q1: What is the difference between BeautifulSoup and lxml?**
A: Both parse HTML/XML. lxml is faster (C-based) and supports XPath natively. BeautifulSoup is more forgiving with malformed HTML and has a simpler API.

**Q2: How do you handle JavaScript-rendered content?**
A: Use Selenium (WebDriver API) or Playwright (modern browser automation). These control a real browser that executes JavaScript, then extract the rendered HTML.

**Q3: What is robots.txt and why is it important?**
A: robots.txt is a file at the website root that tells crawlers which paths they may or may not access. Ethical scrapers should always check and respect it.

**Q4: How do you avoid getting blocked while scraping?**
A: Use polite delays, rotate User-Agents and proxies, mimic human behavior, handle cookies, and avoid scraping too aggressively.

**Q5: What is the difference between scraping and crawling?**
A: Crawling is systematically discovering and visiting URLs across a website. Scraping is extracting specific data from individual pages.

## Coding Challenges

**Challenge 1: Product Price Tracker**
Build a scraper that monitors product prices on an e-commerce site, detects price changes, and alerts when prices drop below a threshold.

**Challenge 2: News Headline Collector**
Create a scraper that collects headlines, summaries, and publication dates from multiple news sites, deduplicates them, and exports to CSV.

**Challenge 3: Recipe Scraper**
Scrape recipe websites for ingredients, instructions, prep time, and nutritional info. Handle structured data (JSON-LD) embedded in pages.

**Challenge 4: Dynamic Content Scraper**
Use Playwright to scrape a single-page application that loads content via API calls. Intercept network requests to capture JSON data directly.

**Challenge 5: Distributed Crawler**
Build a multi-threaded crawler that respects robots.txt, implements politeness policies, uses checkpoint/resume, and exports structured data to a database.

## Summary

Web scraping is a powerful technique for extracting data from websites when APIs are unavailable. Python provides a mature ecosystem — BeautifulSoup for parsing, lxml for speed, Selenium/Playwright for JavaScript rendering, and Scrapy for large-scale projects. Ethical scraping requires respecting robots.txt, implementing rate limiting, and identifying yourself properly. Modern web scraping must handle dynamic content, anti-bot protections, pagination, and large-scale crawling.

## Related Topics

- [70. HTTP Requests](./70_http_requests.md)
- [71. APIs](./71_apis.md)
- [75. REST API Design](./75_rest_api_design.md)
- [76. Authentication](./76_authentication.md)
- [72. Flask](./72_flask.md)
- [73. FastAPI](./73_fastapi.md)
