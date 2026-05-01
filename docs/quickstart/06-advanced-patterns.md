# 6. Advanced Patterns

Master custom features and optimization techniques.

## Custom Identifier Fields

By default, MongoDB uses `_id` as the unique identifier. But you can use any unique field.

### Use Email as ID

```python
users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    identifier_field="email",  # ← Use email instead of _id
)
```

Now routes use email:

```
GET /users/alice@example.com    # Instead of /users/{object_id}
POST /users/alice@example.com
PATCH /users/alice@example.com
DELETE /users/alice@example.com
```

### Use Username as ID

```python
identifier_field="username"
```

```
GET /users/alice_smith
PATCH /users/alice_smith
```

### Advantages

- More readable URLs
- Better for natural identifiers
- Easier for API consumers

### Disadvantages

- Must ensure field is unique
- Slower queries than native `_id`
- Renaming becomes complex

## Disable Specific Routes

Not every API needs all CRUD operations. Disable unwanted routes:

```python
users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    disable_create_one=True,     # No POST - can't create users
    disable_delete_one=True,     # No DELETE - can't delete users
)
```

Now only GET and PATCH are available.

### Common Scenarios

Read-only collection:
```python
disable_create_one=True
disable_replace_one=True
disable_update_one=True
disable_delete_one=True
```

Append-only log:
```python
disable_replace_one=True
disable_update_one=True
disable_delete_one=True
```

## Add Dependencies for Auth

Require authentication on specific routes:

```python
from fastapi import Depends, HTTPException

async def require_admin(user: User = Depends(get_current_user)):
    if not user.is_admin:
        raise HTTPException(status_code=403, detail="Admin only")
    return user


users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    dependencies_delete_one=[Depends(require_admin)],  # Only DELETE needs auth
)
```

Now only admins can delete users.

### Per-Route Dependencies

```python
users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    dependencies_get_all=[Depends(require_auth)],     # Auth for GET all
    dependencies_create_one=[Depends(require_auth)],  # Auth for POST
    dependencies_delete_one=[Depends(require_admin)], # Admin for DELETE
)
```

## Custom Output Schemas in Depth

Different contexts might need different response shapes:

```python
# Full schema (has internal fields)
class UserFullOut(MongoModel):
    id: str
    name: str
    email: str
    is_admin: bool
    created_at: str
    api_quota: int


# Public schema (minimal fields)
class UserPublicOut(MongoModel):
    id: str
    name: str


# Admin schema (includes audit info)
class UserAdminOut(MongoModel):
    id: str
    name: str
    email: str
    is_admin: bool
    created_at: str
    last_login: str | None
```

Use different schemas in different routers:

```python
# Public users API
public_router = CRUDRouter(
    model=UserModel,
    model_out=UserPublicOut,
    db=db,
    collection_name="users",
    prefix="/api/v1/users",  # Public endpoint
)

# Admin users API
admin_router = CRUDRouter(
    model=UserModel,
    model_out=UserAdminOut,
    db=db,
    collection_name="users",
    prefix="/admin/users",  # Admin endpoint
    dependencies_get_all=[Depends(require_admin)],
)

app.include_router(public_router)
app.include_router(admin_router)
```

## Direct Repository/Service Usage

For maximum control, use CRUDRepository or CRUDService directly:

```python
from fastapi_crudrouter_mongodb import CRUDRepository, CRUDService

# Initialize repository
repo = CRUDRepository(
    model=UserModel,
    db=db,
    collection_name="users",
)

# Or service (with error handling)
service = CRUDService(
    model=UserModel,
    db=db,
    collection_name="users",
)

# Build custom endpoint
@app.post("/users/batch-import")
async def import_users(users: list[UserModel]):
    """Custom endpoint: import multiple users"""
    results = []
    for user in users:
        created = await repo.create_one(user)
        results.append(created)
    return results
```

## Advanced Filtering Patterns

### Regex Search

Find users by name pattern:

```
GET /users?filters={"name":{"$regex":"^alice","$options":"i"}}
```

Finds names starting with "alice" (case-insensitive).

### Date Range Queries

Find tracks added in last month:

```
GET /tracks?filters={"created_at":{"$gte":"2024-04-01","$lt":"2024-05-01"}}
```

### Complex Filters

Find premium users from New York:

```
GET /users?filters={"$and":[{"premium":true},{"city":"New York"}]}
```

Or find premium users OR users from New York:

```
GET /users?filters={"$or":[{"premium":true},{"city":"New York"}]}
```

## Pagination Optimization

For large datasets, always use pagination:

```
GET /users?limit=100&skip=0     # Page 1
GET /users?limit=100&skip=100   # Page 2
GET /users?limit=100&skip=200   # Page 3
```

### Best Practice: Include Total Count

Add a separate endpoint:

```python
@app.get("/users/count")
async def count_users(filters: dict | None = None):
    """Get total user count for pagination info"""
    count = await db["users"].count_documents(filters or {})
    return {"total": count}
```

Client uses this to calculate pages:

```javascript
const totalUsers = await fetch('/users/count').then(r => r.json());
const totalPages = Math.ceil(totalUsers.total / pageSize);
```

## Tags and Documentation

Organize routes in API documentation:

```python
users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    tags=["Users"],  # Groups routes in /docs
)

tracks_router = CRUDRouter(
    model=TrackModel,
    db=db,
    collection_name="tracks",
    prefix="/tracks",
    tags=["Tracks"],
)
```

Now /docs shows separate "Users" and "Tracks" sections.

## Response Time Optimization

### Use Limit in find_all

```
GET /users?limit=10  # Fast
GET /users           # Slow if millions of users
```

### Sort Indexes

For frequently sorted fields, add MongoDB indexes:

```python
# After app startup
await db["tracks"].create_index("plays")
await db["users"].create_index("created_at")
```

### Combine Efficiently

```
GET /tracks?sort_by=plays&limit=100&skip=0
```

Gets top 100 played tracks efficiently.

## API Versioning

Support multiple API versions:

```python
# v1 API
v1_router = CRUDRouter(
    model=UserModel,
    model_out=UserModelOutV1,
    prefix="/api/v1/users",
    ...
)

# v2 API (improved schema)
v2_router = CRUDRouter(
    model=UserModel,
    model_out=UserModelOutV2,
    prefix="/api/v2/users",
    ...
)

app.include_router(v1_router)
app.include_router(v2_router)
```

Both versions coexist at:
- `/api/v1/users`
- `/api/v2/users`

## Monitoring & Logging

Log database operations:

```python
import logging

logger = logging.getLogger(__name__)

# Before creating router
@app.middleware("http")
async def log_requests(request, call_next):
    start = time.time()
    response = await call_next(request)
    duration = time.time() - start
    logger.info(f"{request.method} {request.url} - {response.status_code} - {duration:.2f}s")
    return response
```

## Next Steps

Continue learning:

- [**Production Ready**](07-production-ready.md) - Deployment and best practices
- [**Back to Quickstart**](index.md)
