# MongoDB - pymongo.MongoClient(), CRUD, aggregation pipeline

## Introduction

MongoDB is a document-oriented NoSQL database that stores data in flexible, JSON-like BSON documents. Python's `pymongo` library provides a full-featured driver for interacting with MongoDB, supporting CRUD operations, aggregation pipelines, indexing, geospatial queries, change streams, and replica set operations.

## Why It Is Important

MongoDB's schema-less document model allows rapid iteration and flexibility for applications where data shapes evolve frequently. It excels at hierarchical data storage, real-time analytics, content management, IoT data ingestion, and applications requiring horizontal scaling via sharding. Its aggregation pipeline provides powerful data processing capabilities without requiring a separate processing framework.

## Syntax

```python
from pymongo import MongoClient

client = MongoClient('mongodb://localhost:27017/')
db = client['mydatabase']
collection = db['mycollection']

# Insert
result = collection.insert_one({'name': 'Alice', 'age': 30})

# Find
doc = collection.find_one({'name': 'Alice'})
docs = collection.find({'age': {'$gt': 25}})

# Update
collection.update_one({'name': 'Alice'}, {'$set': {'age': 31}})

# Delete
collection.delete_one({'name': 'Alice'})
```

## Examples

### Connection and Database Setup

```python
import os
import json
import time
from datetime import datetime, timedelta
from typing import Optional, List, Dict, Any, Generator
from pymongo import MongoClient, ASCENDING, DESCENDING, TEXT, IndexModel
from pymongo.collection import Collection, ReturnDocument
from pymongo.database import Database
from pymongo.errors import (
    ConnectionFailure, DuplicateKeyError, BulkWriteError,
    OperationFailure, ServerSelectionTimeoutError
)
from bson.objectid import ObjectId
from bson.json_util import dumps, loads
from bson import CodecOptions
from bson.binary import Binary
from bson.codec_options import CodecOptions
import pprint

MONGO_URI = os.environ.get("MONGO_URI", "mongodb://localhost:27017/")
DB_NAME = "example_db"

client = MongoClient(
    MONGO_URI,
    serverSelectionTimeoutMS=5000,
    connectTimeoutMS=3000,
    socketTimeoutMS=10000,
    maxPoolSize=50,
    retryWrites=True,
    retryReads=True
)

db = client[DB_NAME]


def ping_server() -> bool:
    try:
        client.admin.command("ping")
        return True
    except (ConnectionFailure, ServerSelectionTimeoutError):
        return False


def get_or_create_collection(database: Database, name: str) -> Collection:
    if name not in database.list_collection_names():
        database.create_collection(name)
    return database[name]


users_collection = db["users"]
posts_collection = db["posts"]
products_collection = db["products"]
orders_collection = db["orders"]
logs_collection = db["logs"]
```

### CRUD Operations

```python
def create_indexes():
    users_collection.create_index([("email", ASCENDING)], unique=True)
    users_collection.create_index([("username", ASCENDING)], unique=True)
    users_collection.create_index([("created_at", DESCENDING)])
    posts_collection.create_index([("slug", ASCENDING)], unique=True)
    posts_collection.create_index([("author_id", ASCENDING), ("created_at", DESCENDING)])
    posts_collection.create_index([("title", TEXT), ("content", TEXT)])
    products_collection.create_index([("category", ASCENDING), ("price", ASCENDING)])
    products_collection.create_index([("tags", ASCENDING)])
    orders_collection.create_index([("user_id", ASCENDING), ("created_at", DESCENDING)])
    orders_collection.create_index([("status", ASCENDING)])
    logs_collection.create_index([("timestamp", DESCENDING)], expireAfterSeconds=604800)


create_indexes()


def insert_user(user_data: dict) -> str:
    existing = users_collection.find_one({
        "$or": [
            {"email": user_data["email"]},
            {"username": user_data["username"]}
        ]
    })
    if existing:
        raise DuplicateKeyError(f"User with email or username already exists")

    user_data["created_at"] = datetime.utcnow()
    user_data["updated_at"] = datetime.utcnow()
    user_data["is_active"] = True
    user_data["roles"] = user_data.get("roles", ["viewer"])
    user_data["profile"] = user_data.get("profile", {})

    result = users_collection.insert_one(user_data)
    return str(result.inserted_id)


def bulk_insert_users(users: List[dict]) -> List[str]:
    now = datetime.utcnow()
    for user in users:
        user["created_at"] = now
        user["updated_at"] = now
        user["is_active"] = True
        user["roles"] = user.get("roles", ["viewer"])

    try:
        result = users_collection.insert_many(users, ordered=False)
        return [str(oid) for oid in result.inserted_ids]
    except BulkWriteError as e:
        print(f"Bulk write errors: {e.details.get('writeErrors', [])}")
        return []


def find_user_by_id(user_id: str) -> Optional[dict]:
    return users_collection.find_one({"_id": ObjectId(user_id)})


def find_user_by_email(email: str) -> Optional[dict]:
    return users_collection.find_one({"email": email})


def find_users(filter_query: dict = None, sort: list = None, limit: int = 20, skip: int = 0) -> List[dict]:
    filter_query = filter_query or {}
    sort = sort or [("created_at", DESCENDING)]
    cursor = users_collection.find(filter_query).sort(sort).skip(skip).limit(limit)
    return list(cursor)


def count_users(filter_query: dict = None) -> int:
    return users_collection.count_documents(filter_query or {})


def find_active_users(page: int = 1, per_page: int = 20) -> dict:
    filter_query = {"is_active": True}
    total = count_users(filter_query)
    users = find_users(filter_query, limit=per_page, skip=(page - 1) * per_page)
    return {
        "users": users,
        "total": total,
        "page": page,
        "per_page": per_page,
        "pages": (total + per_page - 1) // per_page
    }


def search_users(search_term: str) -> List[dict]:
    return list(users_collection.find({
        "$or": [
            {"username": {"$regex": search_term, "$options": "i"}},
            {"email": {"$regex": search_term, "$options": "i"}},
            {"profile.full_name": {"$regex": search_term, "$options": "i"}}
        ]
    }).limit(20))


def update_user(user_id: str, update_data: dict) -> bool:
    update_data["updated_at"] = datetime.utcnow()
    result = users_collection.update_one(
        {"_id": ObjectId(user_id)},
        {"$set": update_data}
    )
    return result.modified_count > 0


def add_user_role(user_id: str, role: str) -> bool:
    result = users_collection.update_one(
        {"_id": ObjectId(user_id)},
        {"$addToSet": {"roles": role}}
    )
    return result.modified_count > 0


def remove_user_role(user_id: str, role: str) -> bool:
    result = users_collection.update_one(
        {"_id": ObjectId(user_id)},
        {"$pull": {"roles": role}}
    )
    return result.modified_count > 0


def increment_user_login_count(user_id: str) -> bool:
    result = users_collection.update_one(
        {"_id": ObjectId(user_id)},
        {
            "$inc": {"login_count": 1},
            "$set": {"last_login": datetime.utcnow()}
        }
    )
    return result.modified_count > 0


def upsert_user(filter_query: dict, update_data: dict) -> str:
    update_data["updated_at"] = datetime.utcnow()
    result = users_collection.update_one(
        filter_query,
        {"$set": update_data},
        upsert=True
    )
    if result.upserted_id:
        return str(result.upserted_id)
    return str(result)


def delete_user(user_id: str) -> bool:
    result = users_collection.delete_one({"_id": ObjectId(user_id)})
    return result.deleted_count > 0


def delete_inactive_users_before(days: int = 30) -> int:
    cutoff = datetime.utcnow() - timedelta(days=days)
    result = users_collection.delete_many({
        "is_active": False,
        "updated_at": {"$lt": cutoff}
    })
    return result.deleted_count


def insert_post(post_data: dict) -> str:
    post_data["created_at"] = datetime.utcnow()
    post_data["updated_at"] = datetime.utcnow()
    post_data["view_count"] = 0
    post_data["comments"] = []
    post_data["likes"] = []
    post_data["is_published"] = False

    result = posts_collection.insert_one(post_data)
    return str(result.inserted_id)


def find_posts_with_author(page: int = 1, per_page: int = 10) -> List[dict]:
    pipeline = [
        {"$match": {"is_published": True}},
        {"$lookup": {
            "from": "users",
            "localField": "author_id",
            "foreignField": "_id",
            "as": "author"
        }},
        {"$unwind": "$author"},
        {"$project": {
            "title": 1,
            "slug": 1,
            "excerpt": 1,
            "view_count": 1,
            "created_at": 1,
            "author.username": 1,
            "author.email": 1
        }},
        {"$sort": {"created_at": -1}},
        {"$skip": (page - 1) * per_page},
        {"$limit": per_page}
    ]
    return list(posts_collection.aggregate(pipeline))


def add_comment_to_post(post_id: str, comment: dict) -> bool:
    comment["_id"] = ObjectId()
    comment["created_at"] = datetime.utcnow()
    comment["approved"] = False

    result = posts_collection.update_one(
        {"_id": ObjectId(post_id)},
        {"$push": {"comments": comment}}
    )
    return result.modified_count > 0


def increment_post_view(post_id: str) -> bool:
    result = posts_collection.update_one(
        {"_id": ObjectId(post_id)},
        {"$inc": {"view_count": 1}}
    )
    return result.modified_count > 0


def like_post(post_id: str, user_id: str) -> bool:
    result = posts_collection.update_one(
        {"_id": ObjectId(post_id)},
        {"$addToSet": {"likes": ObjectId(user_id)}}
    )
    return result.modified_count > 0


def unlike_post(post_id: str, user_id: str) -> bool:
    result = posts_collection.update_one(
        {"_id": ObjectId(post_id)},
        {"$pull": {"likes": ObjectId(user_id)}}
    )
    return result.modified_count > 0


def find_posts_by_tag(tag: str, page: int = 1, per_page: int = 10) -> List[dict]:
    return list(posts_collection.find(
        {"tags": tag, "is_published": True}
    ).sort("created_at", DESCENDING).skip((page - 1) * per_page).limit(per_page))
```

### Aggregation Pipeline

```python
def user_statistics() -> dict:
    pipeline = [
        {"$match": {"is_active": True}},
        {"$group": {
            "_id": None,
            "total_users": {"$sum": 1},
            "avg_login_count": {"$avg": "$login_count"},
            "min_login_count": {"$min": "$login_count"},
            "max_login_count": {"$max": "$login_count"},
            "roles_distribution": {"$push": "$roles"}
        }},
        {"$project": {
            "_id": 0,
            "total_users": 1,
            "avg_login_count": {"$round": ["$avg_login_count", 2]},
            "min_login_count": 1,
            "max_login_count": 1,
            "unique_roles": {"$reduce": {
                "input": "$roles_distribution",
                "initialValue": [],
                "in": {"$setUnion": ["$$value", "$$this"]}
            }}
        }}
    ]
    result = list(users_collection.aggregate(pipeline))
    return result[0] if result else {}


def posts_by_category() -> List[dict]:
    pipeline = [
        {"$match": {"is_published": True}},
        {"$group": {
            "_id": "$category",
            "count": {"$sum": 1},
            "total_views": {"$sum": "$view_count"},
            "avg_views": {"$avg": "$view_count"},
            "latest_post": {"$max": "$created_at"}
        }},
        {"$sort": {"count": -1}},
        {"$project": {
            "category": "$_id",
            "count": 1,
            "total_views": 1,
            "avg_views": {"$round": ["$avg_views", 0]},
            "latest_post": 1,
            "_id": 0
        }}
    ]
    return list(posts_collection.aggregate(pipeline))


def user_engagement_report() -> List[dict]:
    pipeline = [
        {"$match": {"is_published": True}},
        {"$group": {
            "_id": "$author_id",
            "post_count": {"$sum": 1},
            "total_views": {"$sum": "$view_count"},
            "total_comments": {"$sum": {"$size": {"$ifNull": ["$comments", []]}}},
            "total_likes": {"$sum": {"$size": {"$ifNull": ["$likes", []]}}},
            "last_post_date": {"$max": "$created_at"}
        }},
        {"$lookup": {
            "from": "users",
            "localField": "_id",
            "foreignField": "_id",
            "as": "user"
        }},
        {"$unwind": "$user"},
        {"$project": {
            "username": "$user.username",
            "email": "$user.email",
            "post_count": 1,
            "total_views": 1,
            "total_comments": 1,
            "total_likes": 1,
            "last_post_date": 1,
            "_id": 0
        }},
        {"$sort": {"total_views": -1}},
        {"$limit": 10}
    ]
    return list(posts_collection.aggregate(pipeline))


def product_analytics() -> dict:
    pipeline = [
        {"$group": {
            "_id": "$category",
            "products": {"$sum": 1},
            "avg_price": {"$avg": "$price"},
            "min_price": {"$min": "$price"},
            "max_price": {"$max": "$price"},
            "total_value": {"$sum": {"$multiply": ["$price", "$stock"]}}
        }},
        {"$sort": {"products": -1}},
        {"$project": {
            "category": "$_id",
            "products": 1,
            "avg_price": {"$round": ["$avg_price", 2]},
            "min_price": {"$round": ["$min_price", 2]},
            "max_price": {"$round": ["$max_price", 2]},
            "total_value": {"$round": ["$total_value", 2]},
            "_id": 0
        }}
    ]
    return list(products_collection.aggregate(pipeline))


def complex_aggregation() -> List[dict]:
    pipeline = [
        {"$match": {"status": "completed", "total": {"$gt": 0}}},
        {"$addFields": {
            "year": {"$year": "$created_at"},
            "month": {"$month": "$created_at"},
            "quarter": {"$ceil": {"$divide": [{"$month": "$created_at"}, 3]}}
        }},
        {"$group": {
            "_id": {"year": "$year", "quarter": "$quarter"},
            "total_orders": {"$sum": 1},
            "total_revenue": {"$sum": "$total"},
            "avg_order_value": {"$avg": "$total"},
            "unique_customers": {"$addToSet": "$user_id"}
        }},
        {"$sort": {"_id.year": -1, "_id.quarter": -1}},
        {"$project": {
            "year": "$_id.year",
            "quarter": "$_id.quarter",
            "total_orders": 1,
            "total_revenue": {"$round": ["$total_revenue", 2]},
            "avg_order_value": {"$round": ["$avg_order_value", 2]},
            "unique_customers": {"$size": "$unique_customers"},
            "_id": 0
        }}
    ]
    return list(orders_collection.aggregate(pipeline))


def full_text_search(query: str) -> List[dict]:
    pipeline = [
        {"$match": {"$text": {"$search": query}}},
        {"$addFields": {
            "score": {"$meta": "textScore"}
        }},
        {"$sort": {"score": {"$meta": "textScore"}}},
        {"$limit": 10},
        {"$project": {
            "title": 1,
            "slug": 1,
            "excerpt": {"$substrCP": ["$content", 0, 200]},
            "score": 1,
            "_id": 0
        }}
    ]
    return list(posts_collection.aggregate(pipeline))


def facet_search() -> dict:
    pipeline = [
        {"$match": {"is_published": True}},
        {"$facet": {
            "categories": [
                {"$group": {"_id": "$category", "count": {"$sum": 1}}},
                {"$sort": {"count": -1}}
            ],
            "tags": [
                {"$unwind": "$tags"},
                {"$group": {"_id": "$tags", "count": {"$sum": 1}}},
                {"$sort": {"count": -1}},
                {"$limit": 20}
            ],
            "publishing_trend": [
                {"$group": {
                    "_id": {"$dateToString": {"format": "%Y-%m", "date": "$created_at"}},
                    "count": {"$sum": 1}
                }},
                {"$sort": {"_id": 1}}
            ],
            "total": [
                {"$count": "count"}
            ]
        }}
    ]
    result = list(posts_collection.aggregate(pipeline))
    return result[0] if result else {}
```

## Beginner Examples

```python
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client["school"]
students = db["students"]

student = {"name": "Alice", "age": 20, "grade": "A", "subjects": ["Math", "Physics"]}
result = students.insert_one(student)
print(f"Inserted ID: {result.inserted_id}")

alice = students.find_one({"name": "Alice"})
print("Alice:", alice)

all_students = students.find()
for s in all_students:
    print(s["name"], s["grade"])

students.update_one({"name": "Alice"}, {"$set": {"grade": "A+"}})

students.delete_one({"name": "Alice"})

many_students = [
    {"name": "Bob", "age": 22, "grade": "B"},
    {"name": "Charlie", "age": 21, "grade": "A"}
]
students.insert_many(many_students)

for s in students.find().sort("age", -1):
    print(f"{s['name']}: {s['age']}")

students.drop()
client.close()
```

## Intermediate Examples

```python
from pymongo import MongoClient, ASCENDING, DESCENDING
from datetime import datetime
from bson.objectid import ObjectId

client = MongoClient()
db = client["inventory_db"]
products = db["products"]
orders = db["orders"]

products.create_index([("sku", ASCENDING)], unique=True)
products.create_index([("category", ASCENDING), ("price", ASCENDING)])
orders.create_index([("user_email", ASCENDING)])

products.insert_many([
    {"sku": "LAP001", "name": "Laptop", "category": "Electronics", "price": 999.99, "stock": 50, "tags": ["computer", "portable"]},
    {"sku": "PHN001", "name": "Phone", "category": "Electronics", "price": 499.99, "stock": 200, "tags": ["mobile", "communication"]},
    {"sku": "BOK001", "name": "Python Book", "category": "Books", "price": 39.99, "stock": 500, "tags": ["programming", "python"]}
])

electronics = products.find({"category": "Electronics", "price": {"$gte": 500}})
for p in electronics:
    print(p["name"], p["price"])

products.update_one(
    {"sku": "LAP001"},
    {"$inc": {"stock": -5}, "$push": {"tags": "sale"}}
)

products.update_many(
    {"category": "Electronics"},
    {"$set": {"discount": 10}}
)

results = products.aggregate([
    {"$group": {"_id": "$category", "count": {"$sum": 1}, "avg_price": {"$avg": "$price"}}}
])
for r in results:
    print(r["_id"], r["count"], r["avg_price"])

order = {
    "user_email": "alice@example.com",
    "items": [
        {"sku": "LAP001", "qty": 1, "price": 999.99},
        {"sku": "BOK001", "qty": 2, "price": 39.99}
    ],
    "total": 1079.97,
    "status": "pending",
    "created_at": datetime.utcnow()
}
orders.insert_one(order)

client.close()
```

## Advanced Examples

```python
from pymongo import MongoClient, ASCENDING, DESCENDING, UpdateOne, DeleteMany
from pymongo.errors import BulkWriteError
from datetime import datetime, timedelta
from bson.objectid import ObjectId
import asyncio
import threading
import json


class MongoRepository:
    def __init__(self, collection: Collection):
        self.collection = collection

    def insert(self, document: dict) -> str:
        doc = {**document, "created_at": datetime.utcnow(), "updated_at": datetime.utcnow()}
        return str(self.collection.insert_one(doc).inserted_id)

    def insert_many(self, documents: list, ordered: bool = False) -> list:
        now = datetime.utcnow()
        docs = [{**d, "created_at": now, "updated_at": now} for d in documents]
        try:
            result = self.collection.insert_many(docs, ordered=ordered)
            return [str(oid) for oid in result.inserted_ids]
        except BulkWriteError as bwe:
            return [str(oid) for oid in bwe.details.get("insertedIds", [])]

    def find_by_id(self, doc_id: str) -> Optional[dict]:
        return self.collection.find_one({"_id": ObjectId(doc_id)})

    def find_one(self, filter_query: dict) -> Optional[dict]:
        return self.collection.find_one(filter_query)

    def find(self, filter_query: dict = None, projection: dict = None, sort: list = None,
             limit: int = 20, skip: int = 0) -> List[dict]:
        cursor = self.collection.find(filter_query or {}, projection or {})
        if sort:
            cursor = cursor.sort(sort)
        return list(cursor.skip(skip).limit(limit))

    def update_by_id(self, doc_id: str, update_data: dict) -> bool:
        update_data["updated_at"] = datetime.utcnow()
        result = self.collection.update_one(
            {"_id": ObjectId(doc_id)},
            {"$set": update_data}
        )
        return result.modified_count > 0

    def update_one(self, filter_query: dict, update_data: dict, upsert: bool = False) -> dict:
        update_data["updated_at"] = datetime.utcnow()
        result = self.collection.update_one(
            filter_query,
            {"$set": update_data},
            upsert=upsert
        )
        return {
            "matched_count": result.matched_count,
            "modified_count": result.modified_count,
            "upserted_id": str(result.upserted_id) if result.upserted_id else None
        }

    def delete_by_id(self, doc_id: str) -> bool:
        result = self.collection.delete_one({"_id": ObjectId(doc_id)})
        return result.deleted_count > 0

    def delete_many(self, filter_query: dict) -> int:
        result = self.collection.delete_many(filter_query)
        return result.deleted_count

    def count(self, filter_query: dict = None) -> int:
        return self.collection.count_documents(filter_query or {})

    def aggregate(self, pipeline: list) -> list:
        return list(self.collection.aggregate(pipeline))

    def bulk_write(self, operations: list, ordered: bool = True) -> dict:
        try:
            result = self.collection.bulk_write(operations, ordered=ordered)
            return {
                "inserted": result.inserted_count,
                "matched": result.matched_count,
                "modified": result.modified_count,
                "deleted": result.deleted_count,
                "upserted": len(result.upserted_ids)
            }
        except BulkWriteError as bwe:
            return {"error": str(bwe.details)}

    def find_one_and_update(self, filter_query: dict, update_data: dict, return_document=ReturnDocument.AFTER) -> Optional[dict]:
        update_data["updated_at"] = datetime.utcnow()
        return self.collection.find_one_and_update(
            filter_query,
            {"$set": update_data},
            return_document=return_document
        )

    def distinct(self, field: str, filter_query: dict = None) -> list:
        return self.collection.distinct(field, filter_query)

    def create_index(self, keys, **kwargs):
        return self.collection.create_index(keys, **kwargs)

    def create_text_index(self, *fields: str, **kwargs):
        index = [(field, TEXT) for field in fields]
        return self.collection.create_index(index, **kwargs)


class ChangeStreamWatcher:
    def __init__(self, collection: Collection, pipeline: list = None):
        self.collection = collection
        self.pipeline = pipeline or []
        self._thread = None
        self._running = False

    def start(self, callback):
        self._running = True
        self._thread = threading.Thread(target=self._watch, args=(callback,), daemon=True)
        self._thread.start()

    def stop(self):
        self._running = False

    def _watch(self, callback):
        while self._running:
            try:
                with self.collection.watch(self.pipeline) as stream:
                    for change in stream:
                        if not self._running:
                            break
                        callback(change)
            except Exception as e:
                if self._running:
                    time.sleep(1)


class TransactionManager:
    def __init__(self, client: MongoClient):
        self.client = client

    def execute_transaction(self, operations):
        with self.client.start_session() as session:
            with session.start_transaction():
                try:
                    result = operations(session)
                    session.commit_transaction()
                    return result
                except Exception as e:
                    session.abort_transaction()
                    raise


class TimeSeriesCollection:
    def __init__(self, collection: Collection):
        self.collection = collection

    def insert_measurement(self, metric: str, value: float, tags: dict = None):
        doc = {
            "metric": metric,
            "value": value,
            "tags": tags or {},
            "timestamp": datetime.utcnow()
        }
        self.collection.insert_one(doc)

    def query_range(self, metric: str, start: datetime, end: datetime, granularity: str = "hour") -> list:
        group_id = {}
        if granularity == "minute":
            group_id = {
                "year": {"$year": "$timestamp"},
                "month": {"$month": "$timestamp"},
                "day": {"$dayOfMonth": "$timestamp"},
                "hour": {"$hour": "$timestamp"},
                "minute": {"$minute": "$timestamp"}
            }
        elif granularity == "hour":
            group_id = {
                "year": {"$year": "$timestamp"},
                "month": {"$month": "$timestamp"},
                "day": {"$dayOfMonth": "$timestamp"},
                "hour": {"$hour": "$timestamp"}
            }
        elif granularity == "day":
            group_id = {
                "year": {"$year": "$timestamp"},
                "month": {"$month": "$timestamp"},
                "day": {"$dayOfMonth": "$timestamp"}
            }

        pipeline = [
            {"$match": {
                "metric": metric,
                "timestamp": {"$gte": start, "$lte": end}
            }},
            {"$group": {
                "_id": group_id,
                "avg_value": {"$avg": "$value"},
                "min_value": {"$min": "$value"},
                "max_value": {"$max": "$value"},
                "count": {"$sum": 1}
            }},
            {"$sort": {"_id": 1}}
        ]
        return list(self.collection.aggregate(pipeline))


def advanced_demo():
    client = MongoClient()
    db = client["advanced_demo"]

    users_repo = MongoRepository(db["users"])
    posts_repo = MongoRepository(db["posts"])

    users_repo.create_index("email", unique=True)
    users_repo.create_index([("created_at", DESCENDING)])
    posts_repo.create_text_index("title", "content")

    user_id = users_repo.insert({
        "username": "jane_doe",
        "email": "jane@example.com",
        "roles": ["admin", "editor"],
        "profile": {
            "full_name": "Jane Doe",
            "age": 28,
            "location": "San Francisco"
        }
    })
    print(f"Created user: {user_id}")

    user = users_repo.find_by_id(user_id)
    print(f"Found user: {user['username']}")

    updated = users_repo.update_by_id(user_id, {"profile.age": 29, "profile.location": "New York"})
    print(f"Updated: {updated}")

    query_results = users_repo.find({"roles": "admin"}, limit=5)
    print(f"Admin users: {len(query_results)}")

    bulk_data = [
        {"username": f"user_{i}", "email": f"user_{i}@example.com"} for i in range(10)
    ]
    inserted = users_repo.insert_many(bulk_data)
    print(f"Bulk inserted: {len(inserted)}")

    aggregator = TimeSeriesCollection(db["metrics"])
    for i in range(100):
        aggregator.insert_measurement("cpu_usage", 50 + 10 * (i % 10), {"host": f"server_{i % 5}"})

    now = datetime.utcnow()
    start = now - timedelta(hours=1)
    series = aggregator.query_range("cpu_usage", start, now, "minute")
    print(f"Time series points: {len(series)}")

    watcher = ChangeStreamWatcher(db["users"])
    watcher.start(lambda change: print(f"Change detected: {change.get('operationType')}"))
    time.sleep(0.5)
    users_repo.insert({"username": "test_watch", "email": "watch@example.com"})
    time.sleep(0.5)
    watcher.stop()

    bulk_ops = [
        UpdateOne({"email": "jane@example.com"}, {"$set": {"status": "verified"}}),
        UpdateOne({"email": "user_0@example.com"}, {"$set": {"status": "verified"}}),
        DeleteMany({"email": {"$regex": r"user_[5-9]@example\.com"}})
    ]
    result = users_repo.bulk_write(bulk_ops)
    print(f"Bulk write result: {result}")

    doc = users_repo.find_one_and_update(
        {"username": "jane_doe"},
        {"last_seen": datetime.utcnow().isoformat()}
    )
    print(f"Find and update: {doc['username']} - {doc.get('last_seen')}")

    distinct_roles = users_repo.distinct("roles")
    print(f"Distinct roles: {distinct_roles}")

    client.drop_database("advanced_demo")
    client.close()


advanced_demo()
```

## Real-World Use Cases

MongoDB is used in content management systems, real-time analytics platforms, IoT data ingestion, mobile app backends, catalog management for e-commerce, user profile stores, gaming leaderboards, log aggregation systems, and any application requiring flexible schema design and horizontal scaling.

## Common Mistakes

- Not creating indexes for frequently queried fields, causing slow queries.
- Using `find()` without limits on large collections.
- Not handling `DuplicateKeyError` for unique indexes.
- Storing large binary files directly in documents (use GridFS instead).
- Using deeply nested documents without considering the 16MB document size limit.
- Ignoring connection pooling configuration.
- Not using projection to limit returned fields.

## Best Practices

- Always create indexes for query patterns; use `explain()` to verify.
- Use projections to return only needed fields.
- Implement pagination with `skip`/`limit` or `_id`-based cursors.
- Use aggregation pipeline for complex transformations server-side.
- Enable retryable writes and reads for resilience.
- Use the MongoDB driver's connection pooling.
- For time-series data, use bucketing patterns or MongoDB Time Series collections.
- Monitor query performance with `db.currentOp()` and slow query logging.

## Interview Questions

1. What is the difference between MongoDB and relational databases? — MongoDB is schema-less, stores documents (BSON/JSON), supports embedded documents and arrays, scales horizontally via sharding, and uses a flexible query language with an aggregation pipeline.
2. How does MongoDB handle indexing? — Supports single-field, compound, multikey (array), text, geospatial, hashed, and TTL indexes; creates indexes with `createIndex()` and can enforce uniqueness.
3. Explain the MongoDB aggregation pipeline. — A data processing pipeline of stages (`$match`, `$group`, `$sort`, `$project`, `$lookup`, etc.) that process documents sequentially, similar to Unix pipes.
4. What is an ObjectId and how is it structured? — A 12-byte BSON type: 4-byte timestamp, 5-byte random value, 3-byte increment counter; used as default `_id` field.
5. How do transactions work in MongoDB? — Multi-document ACID transactions are supported in replica sets (4.0+) and sharded clusters (4.2+), using `start_transaction()` and `commit_transaction()`.

## Coding Challenges

1. **Blog Engine** — Build a blog with users, posts with embedded comments, tags, and categories. Implement full-text search, pagination, and aggregation for analytics.
2. **E-Commerce Catalog** — Create a product catalog with categories, variants, pricing, and inventory. Implement faceted search with aggregation pipeline.
3. **Real-Time Analytics** — Build a clickstream analytics system storing events, using aggregation for daily/weekly reports and time-series analysis.
4. **Social Feed** — Implement a social media feed with posts, likes, comments, and follows. Optimize for read-heavy workloads with proper indexing.
5. **Inventory Management** — Create an inventory system with products, warehouses, stock levels, and order management. Use transactions for atomic operations.

## Summary

MongoDB is a document-oriented NoSQL database offering flexible schemas, powerful query capabilities, and horizontal scaling. PyMongo provides a comprehensive driver supporting CRUD operations, aggregation pipelines, indexing, change streams, and transactions. MongoDB excels in applications requiring rapid iteration, hierarchical data, real-time analytics, and scalable document storage.

## Related Topics

- Motor (async MongoDB driver for Python)
- MongoDB Atlas (cloud-hosted MongoDB)
- Redis (in-memory cache often used alongside MongoDB)
- Elasticsearch (search engine for full-text search complementing MongoDB)
- SQLAlchemy (ORM for relational databases, alternative to document databases)
- Mongoose (similar ODM for Node.js)
