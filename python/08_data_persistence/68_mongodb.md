# MongoDB - pymongo.MongoClient(), CRUD, aggregation pipeline

## Introduction
MongoDB is a leading NoSQL document database that stores data in flexible, JSON-like BSON (Binary JSON) documents. Unlike relational databases with fixed schemas, MongoDB allows each document to have different fields, making it ideal for applications with evolving data models, hierarchical data, and rapid prototyping. Python developers interact with MongoDB through `pymongo`, the official MongoDB driver, which provides a synchronous API and integrates tightly with Python data structures (dict, list). MongoDB's query language supports rich field queries, geospatial indexing, text search, and a powerful aggregation pipeline for data transformation and analysis.

## pymongo.MongoClient()

### What It Is
`pymongo.MongoClient()` is the entry point for connecting to a MongoDB server or replica set. It manages connection pooling, server discovery, authentication, and failover. Multiple databases and collections can be accessed from a single client instance.

### Why It Is Important
The client handles all low-level networking, connection lifecycle, and topology monitoring. It maintains a pool of sockets to the MongoDB servers, automatically discovers replica set members, selects the correct primary for writes, and distributes reads to secondaries when configured. Without it, every operation would require manual connection management and server selection.

### How It Works Internally
`MongoClient` uses the SDAM (Server Discovery and Monitoring) specification to periodically check server status. It maintains a thread pool for monitoring and a connection pool for application operations. The client parses the connection URI to determine the deployment type (standalone, replica set, sharded cluster) and configures itself accordingly. Operations are serialized using the MongoDB Wire Protocol over TCP, with optional TLS encryption.

### Syntax
```python
from pymongo import MongoClient

# Basic connection
client = MongoClient("mongodb://localhost:27017")

# With authentication
client = MongoClient("mongodb://user:pass@localhost:27017/mydb")

# Connection string with options
client = MongoClient("mongodb://localhost:27017/?maxPoolSize=50&w=majority&retryWrites=true")

# Connect to replica set
client = MongoClient("mongodb://host1:27017,host2:27017,host3:27017/?replicaSet=myReplicaSet")

# Connect to Atlas (cloud)
client = MongoClient("mongodb+srv://user:pass@cluster0.xxxxx.mongodb.net/myFirstDB")

# Access database and collection
db = client["mydatabase"]
collection = db["users"]
```

### Beginner Examples
```python
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017")
db = client["shop"]
products = db["products"]

# Insert a document
product = {
    "name": "Laptop",
    "price": 1200.00,
    "category": "Electronics",
    "in_stock": True,
    "tags": ["computers", "portable"]
}
result = products.insert_one(product)
print(f"Inserted ID: {result.inserted_id}")

# Find documents
for doc in products.find({"category": "Electronics"}):
    print(doc["name"], doc["price"])
```

### Intermediate Examples
```python
from pymongo import MongoClient, ASCENDING, DESCENDING
from datetime import datetime
import pymongo.errors

client = MongoClient("mongodb://localhost:27017", maxPoolSize=10)
db = client["analytics"]
events = db["events"]

# Create indexes
events.create_index([("timestamp", DESCENDING)])
events.create_index([("user_id", ASCENDING), ("event_type", ASCENDING)], unique=False)

# Bulk insert
event_batch = [
    {"user_id": 1, "event_type": "page_view", "timestamp": datetime.utcnow(), "url": "/home"},
    {"user_id": 1, "event_type": "click", "timestamp": datetime.utcnow(), "element": "signup_btn"},
    {"user_id": 2, "event_type": "purchase", "timestamp": datetime.utcnow(), "amount": 49.99},
]
result = events.insert_many(event_batch)
print(f"Inserted {len(result.inserted_ids)} documents")

# Find with projection and sorting
cursor = events.find(
    {"user_id": 1},
    {"event_type": 1, "timestamp": 1, "_id": 0}
).sort("timestamp", DESCENDING).limit(10)

for event in cursor:
    print(event)

# Count documents
count = events.count_documents({"event_type": "purchase"})
print(f"Total purchases: {count}")
```

## CRUD operations

### What It Is
CRUD (Create, Read, Update, Delete) operations form the foundation of MongoDB data interaction. PyMongo provides methods on Collection objects to insert, query, modify, and remove documents, with extensive query operators for filtering and projection.

### Create
```python
# Insert a single document
result = collection.insert_one({"name": "Alice", "age": 30})

# Insert multiple documents
result = collection.insert_many([
    {"name": "Bob", "age": 25},
    {"name": "Charlie", "age": 35},
])

# Insert with custom _id
result = collection.insert_one({"_id": 1001, "name": "Diana"})
```

### Read
```python
# Find one document
doc = collection.find_one({"name": "Alice"})

# Find with query operators
docs = collection.find({"age": {"$gte": 25, "$lte": 40}})
docs = collection.find({"name": {"$regex": "^A"}})
docs = collection.find({"tags": {"$in": ["python", "mongodb"]}})
docs = collection.find({"$or": [{"status": "active"}, {"age": {"$lt": 18}}]})

# Projection (select fields)
docs = collection.find({}, {"name": 1, "email": 1, "_id": 0})

# Pagination
page = collection.find().skip(20).limit(10)
```

### Update
```python
# Update one document
result = collection.update_one(
    {"name": "Alice"},
    {"$set": {"age": 31, "updated_at": datetime.utcnow()}}
)
print(f"Matched: {result.matched_count}, Modified: {result.modified_count}")

# Update many documents
result = collection.update_many(
    {"status": "inactive"},
    {"$set": {"status": "active"}, "$inc": {"reactivation_count": 1}}
)

# Upsert (insert if not found)
result = collection.update_one(
    {"email": "new@example.com"},
    {"$setOnInsert": {"name": "New User", "created_at": datetime.utcnow()}},
    upsert=True
)

# Array operations
collection.update_one(
    {"_id": 1},
    {"$push": {"tags": "new_tag"}}  # Add to array
)
collection.update_one(
    {"_id": 1},
    {"$pull": {"tags": "old_tag"}}  # Remove from array
)
```

### Delete
```python
# Delete one document
result = collection.delete_one({"status": "temporary"})
print(f"Deleted: {result.deleted_count}")

# Delete many documents
result = collection.delete_many({"created_at": {"$lt": cutoff_date}})

# Drop entire collection
collection.drop()
```

### Advanced Examples
```python
from pymongo import MongoClient, ReturnDocument
from datetime import datetime, timedelta

client = MongoClient("mongodb://localhost:27017")
db = client["ecommerce"]
orders = db["orders"]
inventory = db["inventory"]

# Atomic find-and-update for order processing
def place_order(user_id, product_id, quantity):
    # Reserve inventory atomically
    result = inventory.find_one_and_update(
        {"_id": product_id, "stock": {"$gte": quantity}},
        {"$inc": {"stock": -quantity}},
        return_document=ReturnDocument.AFTER
    )
    if not result:
        raise ValueError("Insufficient stock")

    # Create order
    order = {
        "user_id": user_id,
        "product_id": product_id,
        "quantity": quantity,
        "status": "confirmed",
        "created_at": datetime.utcnow(),
    }
    order_id = orders.insert_one(order).inserted_id
    return order_id

# Bulk write operations
from pymongo import InsertOne, UpdateOne, DeleteOne

operations = [
    InsertOne({"name": "Product X", "price": 19.99}),
    UpdateOne({"name": "Product Y"}, {"$set": {"price": 29.99}}),
    DeleteOne({"name": "Product Z"}),
]
result = inventory.bulk_write(operations)
```

## Aggregation pipeline

### What It It Is
The aggregation pipeline is MongoDB's most powerful data processing framework. It processes documents through a series of stages, where each stage transforms the data stream. Stages can filter, group, sort, project, unwind arrays, join collections, and perform complex computations.

### Why It Is Important
The aggregation pipeline enables server-side data processing that would otherwise require multiple queries and client-side logic. It supports operations equivalent to SQL's GROUP BY, JOIN, WHERE, HAVING, and window functions, plus MongoDB-specific features like `$unwind` for array deconstruction and `$facet` for multi-dimensional analysis.

### How It Works Internally
Each stage receives a stream of documents and outputs a transformed stream to the next stage. The pipeline uses indexes where possible (`$match` at the start), processes documents in batches, and can spill to disk for large datasets. Stages are optimized internally: `$match` and `$sort` at the beginning can use indexes; `$limit` early reduces processing.

### Syntax
```python
pipeline = [
    {"$match": {"status": "active"}},
    {"$group": {"_id": "$category", "count": {"$sum": 1}, "avg_price": {"$avg": "$price"}}},
    {"$sort": {"count": -1}},
    {"$limit": 10},
]

results = collection.aggregate(pipeline)
for doc in results:
    print(doc)
```

### Intermediate Examples
```python
from pymongo import MongoClient
from datetime import datetime, timedelta

client = MongoClient("mongodb://localhost:27017")
db = client["analytics"]
sales = db["sales"]

# Monthly sales report
pipeline = [
    {"$match": {"date": {"$gte": datetime(2024, 1, 1)}}},
    {"$group": {
        "_id": {"year": {"$year": "$date"}, "month": {"$month": "$date"}},
        "total_sales": {"$sum": "$amount"},
        "order_count": {"$sum": 1},
        "avg_order_value": {"$avg": "$amount"},
    }},
    {"$sort": {"_id.year": 1, "_id.month": 1}},
]

for report in sales.aggregate(pipeline):
    print(f"{report['_id']['year']}-{report['_id']['month']:02d}: "
          f"${report['total_sales']:.2f} ({report['order_count']} orders)")

# Customer lifetime value with multiple stages
pipeline = [
    {"$group": {
        "_id": "$customer_id",
        "total_spent": {"$sum": "$amount"},
        "order_count": {"$sum": 1},
        "first_purchase": {"$min": "$date"},
        "last_purchase": {"$max": "$date"},
    }},
    {"$addFields": {
        "avg_order_value": {"$divide": ["$total_spent", "$order_count"]},
        "customer_lifetime_days": {
            "$divide": [
                {"$subtract": ["$last_purchase", "$first_purchase"]},
                1000 * 60 * 60 * 24
            ]
        }
    }},
    {"$sort": {"total_spent": -1}},
    {"$limit": 100},
]

# Unwind arrays
pipeline = [
    {"$unwind": "$items"},
    {"$group": {
        "_id": "$items.product_id",
        "total_quantity": {"$sum": "$items.quantity"},
        "total_revenue": {"$sum": {"$multiply": ["$items.quantity", "$items.price"]}},
    }},
    {"$sort": {"total_revenue": -1}},
]
```

### Advanced Examples
```python
# Complex aggregation with $lookup (JOIN), $facet, and $bucket
pipeline = [
    # Join with customers collection
    {"$lookup": {
        "from": "customers",
        "localField": "customer_id",
        "foreignField": "_id",
        "as": "customer"
    }},
    {"$unwind": "$customer"},
    # Filter active customers
    {"$match": {"customer.status": "active"}},
    # Compute various metrics in parallel
    {"$facet": {
        "by_category": [
            {"$group": {
                "_id": "$category",
                "total": {"$sum": "$amount"},
                "avg": {"$avg": "$amount"},
                "count": {"$sum": 1}
            }},
            {"$sort": {"total": -1}}
        ],
        "by_price_range": [
            {"$bucket": {
                "groupBy": "$amount",
                "boundaries": [0, 50, 100, 200, 500, 1000],
                "default": "1000+",
                "output": {
                    "count": {"$sum": 1},
                    "total": {"$sum": "$amount"}
                }
            }}
        ],
        "monthly_trend": [
            {"$group": {
                "_id": {"$dateToString": {"format": "%Y-%m", "date": "$date"}},
                "total": {"$sum": "$amount"},
                "count": {"$sum": 1}
            }},
            {"$sort": {"_id": 1}}
        ]
    }}
]

# Conditional aggregation
pipeline = [
    {"$group": {
        "_id": "$region",
        "total_revenue": {"$sum": "$amount"},
        "high_value_count": {
            "$sum": {"$cond": [{"$gte": ["$amount", 100]}, 1, 0]}
        },
        "low_value_count": {
            "$sum": {"$cond": [{"$lt": ["$amount", 100]}, 1, 0]}
        },
    }},
    {"$addFields": {
        "high_value_pct": {
            "$multiply": [
                {"$divide": ["$high_value_count", {"$add": ["$high_value_count", "$low_value_count"]}]},
                100
            ]
        }
    }}
]

# Aggregation with output to collection
pipeline = [
    {"$match": {"date": {"$gte": start_date}}},
    {"$group": {"_id": "$product_id", "total": {"$sum": "$amount"}}},
    {"$out": "daily_summary"}  # Write results to a collection
]
collection.aggregate(pipeline)
```

### Real-World Use Cases
- **Real-time analytics**: Aggregation pipelines for dashboards, metrics, and reporting.
- **E-commerce catalogs**: Flexible product schemas with varying attributes per category.
- **Content management systems**: Hierarchical content with embedded comments and tags.
- **IoT data ingestion**: Time-series sensor data with high write throughput.
- **User activity feeds**: Storing and querying event streams for personalized feeds.

### Common Mistakes
- **Not using indexes**: Queries on unindexed fields scan entire collections.
- **Oversized documents**: Deeply nested or huge arrays cause performance issues.
- **Missing error handling**: Network failures, duplicate key errors, and timeout handling.
- **Using `find()` with no filter on large collections without pagination.
- **Creating too many indexes**: Each index slows writes. Monitor index usage.
- **Not configuring write concern**: Losing data on primary failure with default settings.

### Best Practices
- Always create indexes for fields used in `find()`, `sort()`, and `$match` stages.
- Use projections to return only needed fields.
- Prefer `$set` and `$unset` for partial updates over replacing entire documents.
- Keep document size reasonable (under 16MB BSON limit, ideally under a few KB).
- Use `$lookup` sparingly; prefer embedded documents or denormalization for performance.
- Enable retryable writes for production deployments.
- Monitor with `explain()` to understand query performance.

### Performance Considerations
- Use `batch_size()` on cursors to control network round trips.
- Aggregation pipeline stages should be ordered `$match -> $sort -> $limit -> $project`.
- Use `allowDiskUse=True` for large aggregations exceeding 100MB RAM.
- For time-series data, consider MongoDB's Time Series Collections.
- Write operations benefit from using `ordered=False` for bulk inserts.
- Connection pooling with `maxPoolSize` matched to application concurrency.

### Interview Questions
1. What is the difference between MongoDB and a relational database?
2. How does the aggregation pipeline compare to SQL's GROUP BY?
3. Explain the `$lookup` stage and when you would use it vs. embedded documents.
4. What are indexes in MongoDB and how do you choose which fields to index?
5. How does MongoDB handle transactions (ACID compliance)?
6. What is the difference between `update_one()` and `replace_one()`?
7. Explain MongoDB's replica set architecture and write concern.

### Coding Challenges
1. **Product Catalog API**: Build a product catalog with dynamic attributes using MongoDB, supporting filtering by any field and text search.
2. **Sales Dashboard**: Create an aggregation pipeline that produces daily, weekly, and monthly sales reports with product category breakdowns.
3. **Social Media Feed**: Design a schema and queries for a social media feed with posts, comments, likes, and followers using MongoDB.

### Related Topics
- MongoDB Atlas (cloud-hosted MongoDB)
- Mongoose (ODM for Node.js, conceptual comparison)
- Redis caching alongside MongoDB
- Beanie/Motor (async MongoDB drivers for Python)
- Database migration strategies for document databases
