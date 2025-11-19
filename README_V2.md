# Djongo to MongoEngine Migration Guide (Continuation of RM_Django_Frontend_New)

## Table of Contents
- [1. Project Overview](#1-project-overview)
- [2. Updated Djongo to MongoEngine Migration Logic](#2-updated-djongo-to-mongoengine-migration-logic)
- [3. Added New MongoEngine Documents](#3-added-mongoengine-documents)
- [4. Foreign Key Migration for Django User Model](#4-foreign-key-migration-for-django-user-model)
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

### 2.1a. Djongo Principles
- **null**: If **True**, Django will store empty values as **NULL** in the database. Default is **False**.
- **blank**: If **True**, the field is allowed to be blank. Default is **False**.

### 2.1b. MongoEngine Principles
- **required**: (Default: **False** - Equivalent to null=True and blank=True in Djongo) If set to **True** and the field is not set on the document instance, a **ValidationError** will be raised when the document is validated.

#### Decision: Added the missing required=True and db_fields="field_name" to all models.py MongoEngine Documents that require it.

#### Before (Djongo)
```python
class StepNote(models.Model):
    step_name = models.CharField(max_length=200)
    notes = models.JSONField(blank=False, null=False, default=[ ])
    class Meta:
        abstract = True
```
#### After (MongoEngine)
```python
class StepNote(EmbeddedDocument):
    step_name = StringField(max_length=200, required=True)
    notes = ListField(StringField(), default=list, required=True)
```

### 2.2. Missing db_field Parameter in Some Relationships

#### Before (RM_Django_Frontend_New)
```python
class VoucherCode(Document):
    company = RelReferenceField(Company,reverse_delete_rule=DO_NOTHING, related_name="company_code_voucher", target_field="company_domain")
```
#### New
```python
class VoucherCode(Document):
    company = RelReferenceField(Company,reverse_delete_rule=DO_NOTHING, related_name="company_code_voucher", target_field="company_domain", db_field="company_id")
```

## 3. Added New MongoEngine Documents

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
