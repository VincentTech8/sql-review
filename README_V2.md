# Djongo to MongoEngine Migration Guide (Continuation of RM_Django_Frontend_New)

## Table of Contents
- [1. Project Overview](#1-project-overview)
- [2. Updated Djongo to MongoEngine Migration Logic](#2-updated-djongo-to-mongoengine-migration-logic)
- [3. Added MongoEngine Documents](#3-added-mongoengine-documents)
- [4. Foreign Key Migration for Django User Model](#4-foreign-key-migration-for-django-user)
- [5. PostgreSQL Setup for User Model](#5-postgresql-setup-for-user-model)
- [6. Index Management](#6-index-management)
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

## 2. Updated Djongo to MongoEngine Migration Logic

### Djongo Principles
- **null**: If **True**, Django will store empty values as **NULL** in the database. Default is **False**.
- **blank**: If **True**, the field is allowed to be blank. Default is **False**.

### MongoEngine Principles
- **required**: (Default: **False** - Equivalent to null=True and blank=True in Djongo) If set to **True** and the field is not set on the document instance, a **ValidationError** will be raised when the document is validated.

### Decision: Added the missing required=True and db_fields="field_name" to all models.py MongoEngine Documents that requires it.

### Before (Djongo)
```python
class Strategy_Notes(models.Model):
    id = models.IntegerField(primary_key = True)
    strategy_id = models.ForeignKey(Strategy, null=True, blank=True, on_delete=models.DO_NOTHING, related_name="strategy", unique=True)
    notes = models.ArrayField(model_container=StepNote)
```
### After (MongoEngine)
```python
class Strategy_Notes(Document):
    meta = {
        "db_alias": "default",
        "collection": "strategy_strategy_notes",
        "id_field": "mongo_oid",
        "strict": True,
        "allow_inheritance": False
    }
    
    mongo_oid = ObjectIdField(primary_key=True, db_field="_id", default=ObjectId)
    strategy_notes_id = IntField(db_field="id", unique=True, required=True)
    strategy_id = RelReferenceField(
        'Strategy',  
        reverse_delete_rule=DO_NOTHING,
        related_name="strategy",
        target_field="strategy_id",
        db_field="strategy_id_id",
        unique=True,        
        null=True,
        required=False
    )
    notes = ListField(EmbeddedDocumentField(StepNote), default=list, required=True)  
    
    def save(self, *args, **kwargs):
        # Auto-increment business ID
        if self.strategy_notes_id is None:
            self.strategy_notes_id = auto_increment_id(self, 'strategy_notes_id')
        
        return super().save(*args, **kwargs)
```

## 3. Added MongoEngine Documents

## 4. Foreign Key Migration for Django User Model

## 5. PostgreSQL Setup for User Model

## 6. Index Management
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
