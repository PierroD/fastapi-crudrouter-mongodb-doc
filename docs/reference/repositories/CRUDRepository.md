# CRUDRepository

**CRUDRepository** is the low-level data access layer that handles all MongoDB operations. It provides a clean, async interface for creating, reading, updating, and deleting documents.

Most users work with `CRUDRouter` (which uses CRUDRepository internally), but you can use CRUDRepository directly when you need finer control over database operations or want to build custom logic.

## When to Use CRUDRepository

Use CRUDRepository when:

- you need direct control over database queries
- `CRUDRouter` doesn't fit your use case
- you're building custom endpoints with special logic
- you want to reuse database operations across multiple routers

## Quick Setup

```python
from fastapi_crudrouter_mongodb import CRUDRepository, MongoModel, ObjectIdType
from typing import Annotated
import motor.motor_asyncio

ObjectIdType = Annotated[ObjectId, MongoObjectId]


class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str


# Initialize the repository
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.myapp

repo = CRUDRepository(
    model=UserModel,
    db=db,
    collection_name="users",
)

# Now use repo to access data
users = await repo.find_all()
```

## Core Methods

### find_all()

Retrieve all documents from the collection with optional filtering, sorting, and pagination.

```python
async def find_all(
    skip: int | None = None,
    limit: int | None = None,
    sort_by: str | None = None,
    order_by: str | None = None,
    filters: dict | None = None,
    apply_model_out: bool = True,
) -> list:
```

**Parameters:**

- `skip`: number of documents to skip (pagination)
- `limit`: maximum documents to return (pagination)
- `sort_by`: field name to sort by
- `order_by`: sort direction—`"ASC"` or `"DESC"`
- `filters`: MongoDB filter document (e.g., `{"status": "active"}`)
- `apply_model_out`: convert to output schema if defined (default: `True`)

**Example:**

```python
# Get active users, sorted by name, limit 20
users = await repo.find_all(
    filters={"status": "active"},
    sort_by="name",
    order_by="ASC",
    limit=20,
)
```

### find_one()

Retrieve a single document by identifier.

```python
async def find_one(
    id: str,
    apply_model_out: bool = True,
):
```

**Parameters:**

- `id`: document identifier (MongoDB ObjectId or custom identifier)
- `apply_model_out`: convert to output schema if defined (default: `True`)

**Returns:** The document model, or `None` if not found.

**Example:**

```python
user = await repo.find_one("507f1f77bcf86cd799439011")
if user:
    print(user.name)
else:
    print("User not found")
```

### create_one()

Create a new document in the collection.

```python
async def create_one(
    data: MongoModel,
):
```

**Parameters:**

- `data`: an instance of your MongoModel with values to insert

**Returns:** The created document with MongoDB-generated id.

**Example:**

```python
new_user = UserModel(name="Alice", email="alice@example.com")
created = await repo.create_one(new_user)
print(f"Created user with id: {created.id}")
```

### replace_one()

Replace an entire document by id.

```python
async def replace_one(
    id: str,
    data: MongoModel,
):
```

**Parameters:**

- `id`: identifier of document to replace
- `data`: new document content

**Returns:** The replaced document.

**Example:**

```python
updated_user = UserModel(name="Alice Smith", email="alice.smith@example.com")
result = await repo.replace_one("507f1f77bcf86cd799439011", updated_user)
```

### update_one()

Partially update specific fields of a document (like PATCH).

```python
async def update_one(
    id: str,
    data: MongoModel,
):
```

**Parameters:**

- `id`: identifier of document to update
- `data`: fields to update (use MongoDB `$set` operator internally)

**Returns:** The updated document.

**Example:**

```python
# Only update the email field
partial_data = UserModel(email="newemail@example.com")
result = await repo.update_one("507f1f77bcf86cd799439011", partial_data)
```

### delete_one()

Delete a document by id.

```python
async def delete_one(id: str):
```

**Parameters:**

- `id`: identifier of document to delete

**Returns:** The deleted document, or `None` if not found.

**Example:**

```python
deleted = await repo.delete_one("507f1f77bcf86cd799439011")
if deleted:
    print(f"Deleted user: {deleted.name}")
```

## Advanced: Output Schemas

Like `CRUDRouter`, CRUDRepository supports output schemas to hide sensitive fields and control response shape.

```python
class UserModelOut(MongoModel):
    id: str
    name: str
    email: str
    # password field is intentionally omitted


repo = CRUDRepository(
    model=UserModel,
    db=db,
    collection_name="users",
    model_out=UserModelOut,  # All responses use this schema
)

user = await repo.find_one("507f1f77bcf86cd799439011")
# user is now a UserModelOut instance (no password exposed)
```

## Advanced: Custom Identifier Fields

Use custom fields (like email) instead of MongoDB's `_id`:

```python
repo = CRUDRepository(
    model=UserModel,
    db=db,
    collection_name="users",
    identifier_field="email",  # Use email as the id
)

# Now find by email
user = await repo.find_one("alice@example.com")
```

## Advanced: Resolving Populate References

CRUDRepository can automatically resolve reference IDs into full documents using efficient batch queries:

```python
from fastapi_crudrouter_mongodb import CRUDPopulate

artist_populate = CRUDPopulate(
    field="artist_ids",
    collection="artists",
    model=ArtistModel,
)

tracks = await repo.find_all()
tracks = await repo.resolve_populate(tracks, [artist_populate])
# Now track.artist_ids contains full artist documents, not just IDs
```

## Deprecated Methods

The following methods are deprecated but still functional:

- `get_all()` → use `find_all()` instead
- `get_one(id)` → use `find_one(id)` instead
- `mongo` (on MongoModel) → use `to_mongo()` instead

## Related

- [CRUDRouter](../routers/CRUDRouter.md): high-level router builder using CRUDRepository internally
- [MongoModel](../models/MongoModel.md): base model class
- [CRUDPopulate](../routers/CRUDPopulate.md): auto-resolve reference relationships
