# Djongo to MongoEngine Migration Guide (Continuation of RM_Django_Frontend_New)

## Table of Contents
- [1. Project Overview](#1-project-overview)
- [2. Migration Strategy](#2-migration-strategy)
- [3. Primary Key Migration](#3-primary-key-migration)
- [4. Foreign Key Migration](#4-foreign-key-migration)
- [5. Index Management](#5-index-management)
---

## 1. Project Overview

### Goal
Replace Djongo ORM with MongoEngine ODM, using the latest library versions while minimizing database changes and codebase modifications.

### Tech Stack Migration
| Current | Target |
|---------|--------|
| Django 2.2.28 | Django 5.2.5 |
| Djongo 1.3.6 | MongoEngine 0.29.1 |
| PyMongo 3.13.3 | PyMongo 4.14.0 |
| Celery 4.4.7 | Celery (latest) |

### Success Criteria
- Basic CRUD operations working
- Queryset operations functional
- Relationships and foreign keys were properly migrated
- Minimal codebase changes required

### Migration Progress Check

For detailed migration progress, please see the [Updated Migration Progress Table](./MIGRATION_PROGRESS_V2.md).

| App Name     | Progress |
|--------------|----------|
| account      | ❌ Not started |
| competitors  | ❌ Not started |
| dashboard    | ❌ Not started |
| strategy     | ✅ Partially completed |

---

## 2. Migration Strategy

### Core Principles
- **Gradual Migration**: New and old systems coexist during transition
- **Data Preservation**: No data loss or major schema changes
- **Minimal Downtime**: Business operations continue uninterrupted

### Migration Phases
1. **Phase 1**: Independent models (no relationships)
2. **Phase 2**: Models with internal dependencies
3. **Phase 3**: Complex models with cross-app relationships

---

## 3. Primary Key Migration

### Decision: Dual Key System

**When original model has `id` as primary key: Keep both `id` (as business field) and `_id` (as primary key).**

### Before (Djongo)
```python
class Notifications(models.Model):
    id = models.AutoField(primary_key=True)
    title = models.CharField(max_length=200, unique=True)
```
### After (MongoEngine)
```python
class Notifications(Document):
    meta = {
        "collection": "account_notifications",
        "id_field": "mongo_oid"  # Key change
    }

    mongo_oid = ObjectIdField(primary_key=True, db_field="_id", default=ObjectId)
    notification_id = IntField(db_field="id")  # Business ID
    title = StringField(max_length=200, unique=True, required=True)
```
### Why mongo_oid?
**Problem**: MongoEngine's default behavior causes field conflicts
```python
# Without mongo_oid - BROKEN
obj = Notifications.objects.first()
print(obj.pk)              # 6 (wrong: reads business id)
print(obj.notification_id) # None (field consumed by PK logic)
```
**Solution**: Separate primary key and business ID
```python
# With mongo_oid - CORRECT
obj = Notifications.objects.first()
print(obj.pk)              # ObjectId("...") (MongoDB primary key)
print(obj.notification_id) # 6 (business ID preserved)
```
### Access Patterns

- **Primary key**: obj.pk or obj.mongo_oid
- **Business ID**: obj.notification_id (or obj.<name>_id)
- **Legacy compatibility**: 
```python 
# Add this property to model
@property 
def id(self): 
    return self.mongo_oid
```
---
## 4. Foreign Key Migration

### Why RelReferenceField?

Originally, we needed a way to model relationships between documents — just like `ForeignKey` in Django or `ReferenceField` in MongoEngine.

```python
class CartItem(models.Model):
   product = models.ForeignKey(Product, on_delete=models.PROTECT)
```
At first, we tried to use MongoEngine's built-in **ReferenceField**. However, it has a **hard dependency** on the *.id* attribute of the target document. Since our models use *mongo_oid* as the primary key, we added an *@property id* to return *mongo_oid* in order to make it work.
```python
class Product(Document):
    meta = {
        "db_alias": "default", 
        "collection": "account_product", 
        "id_field": "mongo_oid",
        "strict": True,
        "allow_inheritance": False
        }

    mongo_oid = ObjectIdField(primary_key=True, db_field="_id", default=ObjectId)
    
    # original pk
    product_id = IntField(db_field="id", unique=True, required=True)
    
    # let .id to return the primary key value
    @property
    def id(self):
        return self.mongo_oid
```
```python
class CartItem(Document):
    
    meta = {
        "db_alias": "default", 
        "collection": "account_cartitem",
    }

    product = ReferenceField(Product, .....)
```
This hack allowed saving documents, but when reading them back, MongoEngine internally tried to **call .id.to_python()**, which clashed with the @property id. 
As a result, deserialization raised errors like:
```python
Field 'product' - 'property' object has no attribute 'to_python'
```
**Solution: Custom RelReferenceField**
To solve this cleanly, we created the custom field in relationship.py: RelReferenceField. It is based on **BaseField**, **avoids any hardcoded .id usage**, and restores the useful parts of ReferenceField:

- Stores the referenced document's primary key (mongo_oid / pk)
- Lazy loads the related document with instance-level caching
- Supports delete rules (DENY, CASCADE, NULLIFY, PULL)
- Provides safe reverse accessors (related_name)
- Enables bulk prefetch to avoid N+1 queries

```python
from .relationship import RelReferenceField

class CartItem(Document):
    meta = {
        "db_alias": "default", 
        "collection": "account_cartitem"
    }
    product = RelReferenceField(
        Product, 
        required=True, 
        reverse_delete_rule=DENY, 
        related_name="cart_items", 
        validate_reference=True)
```
In short: **RelReferenceField** was introduced because **the built-in ReferenceField** could not work reliably with our custom primary key. It gives us the same convenience as Django's ForeignKey, but adapted for MongoDB and our mongo_oid scheme.

---
## 5. Index Management
### Current Strategy
- **Removed**: Legacy `__primary_key__` indexes from converted models
- **Using**: MongoEngine auto-created indexes (e.g., `id_1` for unique fields)
- **New system**: Standard MongoDB index naming

### Index State After Migration
```javascript
// Current indexes in converted collections:
db.account_order.getIndexes()
[
  {"key": {"_id": 1}, "name": "_id_"},     // MongoDB default
  {"key": {"id": 1}, "name": "id_1"}      // Auto-created by unique=True
]
// Removed: {"key": {"id": 1}, "name": "__primary_key__"}
```

### For Legacy System Performance
If old system queries become slow, restore legacy index:
```python
# create index to account collection 
db.account_collection.createIndex({"id": 1}, {"name": "__primary_key__"})
```
---
