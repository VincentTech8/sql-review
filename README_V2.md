# Djongo to MongoEngine Migration Guide (Continuation of RM_Django_Frontend_New)

## Table of Contents
- [1. Project Overview](#1-project-overview)
- [2. Added MongoEngine Documents](#2-added-mongoengine-documents)
- [3. Foreign Key Migration for Django User Model](#3-foreign-key-migration-for-django-user)
- [4. PostgreSQL Setup for User Model](#4-postgresql-setup-for-user-model)
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

## 2. Added MongoEngine Documents

## 3. Foreign Key Migration for Django User Model

## 4. PostgreSQL Setup for User Model

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
