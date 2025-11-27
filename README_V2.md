# Djongo to MongoEngine Migration Guide (Continuation of RM_Django_Frontend_New)

## Table of Contents
- [1. Project Overview](#1-project-overview)
- [2. Updated Djongo to MongoEngine Migration Logic](#2-updated-djongo-to-mongoengine-migration-logic)
- [3. Added New MongoEngine Documents](#3-added-new-mongoengine-documents)
- [4. Foreign Key Migration for Django User Model](#4-foreign-key-migration-for-django-user-model)
- [5. PostgreSQL Setup for User Model](#5-postgresql-setup-for-user-model)
- [6. Updated Packages](#6-updated-packages)
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

### 2.1. Missing required=True, null=True/False, or blank=True/False Argument
### 2.1a. Djongo Principles
- **null**: If **True**, Django will store empty values as **NULL** in the database. Default is **False**.
- **blank**: If **True**, the field is allowed to be blank. Default is **False**.
- **In default**: Django will store the CharField() and TextField() as **default=""**.

### 2.1b. MongoEngine Principles
- **required**: (Default: **False** - Equivalent to null=True and blank=True in Djongo) If set to **True** and the field is not set on the document instance, a **ValidationError** will be raised when the document is validated.

#### Decision: Added the missing default="", null=True/False, or blank=True/False to all models.py MongoEngine Documents that require it.

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
    step_name = StringField(max_length=200, default="")
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

Added new MongoEngine documents to support expanded data structures, including:
- **XeroxDistributor** from account/models.py
- **Generated_Image** from dashboard/models.py

```python
class XeroxDistributor(Document):
    meta = {
        "db_alias": "default",
        "collection": "account_xeroxdistributor",
        "id_field": "mongo_oid",
        "strict": True,
        "allow_inheritance": False
    }
      
    mongo_oid = ObjectIdField(primary_key=True, db_field="_id", default=ObjectId)
    distributor_id = IntField(db_field="id", unique=True)
    
    first_name = StringField(max_length=255)
    last_name = StringField(max_length=255)
    email = StringField(max_length=255)
    company_name = StringField(max_length=255)
    registered_date = DateTimeField()
    
    def save(self, *args, **kwargs):
        if not self.registered_date:
            self.registered_date = utc_now()
        return super().save(*args, **kwargs)     
```

```python
class Generated_Image(Document):
    meta = {
        "db_alias": "default", 
        "collection": "dashboard_generated_image"
    }
    
    image_company = RelReferenceField(
        Company, 
        required=True, 
        unique=True, 
        reverse_delete_rule=DO_NOTHING,
        verbose_name="User Company", 
        target_field="company_domain",
        db_field = "image_company_id"
    )
    
    user_bg_templates = ListField(DictField(), default=[])
    data = ListField(DictField(), default=[])
    
    def __str__(self):
        return str(self.image_company)    
```

## 4. Foreign Key Migration for Django User Model

Completed foreign key migration for the Django User model to ensure consistent relational mapping across:
- **Campaign** from strategy/models.py -> campaign_creator variable
- **Strategy** from strategy/models.py -> strategy_creator variable



## 5. PostgreSQL Setup for User Model

Once the PostgreSQL database is created, update the settings.py file with the appropriate connection parameters to enable Django to use PostgreSQL, as shown below.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': your_db_name,
        'USER': your_db_username,
        'PASSWORD': your_db_password,
        'HOST': your_db_hostname,
        'PORT': '5432',
        'OPTIONS': {
            'sslmode': 'require',
        },
        'CONN_MAX_AGE': 600,
    }
}
```

Then migrate:
```bash
python manage.py migrate
```

## 6. Updated Packages

---
