# ⚡ Quickstart Guides

Learn fastapi-crudrouter-mongodb progressively, from beginner to advanced mastery.

## Learning Path

Follow this structured journey to master the library:

### 🟢 Beginner

**[1. Installation & Setup](01-installation-setup.md)**  
Get up and running with your first CRUD API in minutes. Learn the basics of models, MongoDB connection, and generating automatic routes.

- Prerequisites and setup
- Creating your first API
- Generating 6 CRUD endpoints automatically
- Testing with Swagger UI

**[2. Models & Schemas](02-models-and-schemas.md)**  
Understand the critical distinction between database models and output schemas. Learn why separating them is essential for security.

- Database models vs output schemas
- Hiding sensitive data (passwords)
- Practical security patterns
- Automatic schema generation

### 🟡 Intermediate

**[3. CRUD Operations & Error Handling](03-crud-operations.md)**  
Master all four operations: Create, Read, Update, and Delete. Learn proper error handling and HTTP status codes.

- POST (Create) - 201 status
- GET (Read) - 200 status, handling 404s
- PATCH (Update) - 201 status
- DELETE (Delete) - 204 status
- Error handling patterns

**[4. Filtering, Sorting & Pagination](04-filtering-sorting-pagination.md)**  
Query and organize your data efficiently. Build powerful search and pagination features.

- Skip/limit pagination
- Sorting by fields
- MongoDB filters
- Combining filters with pagination
- Real-world query examples

**[5. Working with Related Data](05-related-data.md)**  
Connect models together using Populates, Lookups, and Embeds. Learn when to use each approach.

- **Populate** - Resolve references across collections
- **Lookup** - Nested routes for related data
- **Embed** - Nested documents inside parent
- Choosing the right approach

### 🔴 Advanced

**[6. Advanced Patterns](06-advanced-patterns.md)**  
Master custom features, optimization techniques, and production considerations.

- Custom identifier fields (use email, username, etc)
- Disabling specific routes
- Authentication dependencies
- Direct repository/service usage
- Advanced filtering patterns
- API versioning
- Monitoring and logging

**[7. Production Ready](07-production-ready.md)**  
Deploy your API confidently. Learn testing, security, performance, and operational best practices.

- Environment variables and secrets
- Unit and integration testing
- Security best practices (hashing, HTTPS, rate limiting, CORS)
- Docker deployment
- Cloud deployment options
- Monitoring and health checks
- Database backups
- Production checklists

## Quick Start (2 Minutes)

If you prefer the ultra-fast version:

### 1. Install

```bash
pip install fastapi-crudrouter-mongodb
```

### 2. Create your API

```python
import motor.motor_asyncio
from fastapi import FastAPI
from fastapi_crudrouter_mongodb import CRUDRouter, MongoModel
from typing import Annotated
from bson import ObjectId
from fastapi_crudrouter_mongodb import MongoObjectId

ObjectIdType = Annotated[ObjectId, MongoObjectId]

# Model
class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str

# Database
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.music_app

# Router
users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
)

# App
app = FastAPI()
app.include_router(users_router)
```

### 3. Run

```bash
uvicorn main:app --reload
```

### 4. Test

Open [http://localhost:8000/docs](http://localhost:8000/docs) in your browser.

## 🎉 You're Ready!

You now have a fully working CRUD API with:

✅ 6 automatic endpoints (GET all, GET one, POST, PATCH, PUT, DELETE)  
✅ Automatic data validation  
✅ OpenAPI documentation  
✅ No boilerplate code  

**Ready to go deeper?** Start with [Guide 1: Installation & Setup](01-installation-setup.md) for a comprehensive walkthrough.

## Features Overview

| Feature | Status | Guide |
|---------|--------|-------|
| Basic CRUD | ✅ | [Guide 1](01-installation-setup.md) |
| Security (schemas) | ✅ | [Guide 2](02-models-and-schemas.md) |
| Error handling | ✅ | [Guide 3](03-crud-operations.md) |
| Filtering/sorting | ✅ | [Guide 4](04-filtering-sorting-pagination.md) |
| Relationships | ✅ | [Guide 5](05-related-data.md) |
| Custom features | ✅ | [Guide 6](06-advanced-patterns.md) |
| Production ready | ✅ | [Guide 7](07-production-ready.md) |

