# CRUDService

**CRUDService** is the business logic layer that sits between `CRUDRouter` and `CRUDRepository`. It handles HTTP errors, response serialization, and automatic population of referenced documents.

Most users work with `CRUDRouter` (which uses CRUDService internally), but you can use CRUDService directly when you need custom error handling or business logic without building a full router.

## Architecture Overview

The typical layering is:

1. **CRUDRouter** - HTTP routes and FastAPI integration
2. **CRUDService** - Business logic, errors, serialization (this layer)
3. **CRUDRepository** - Low-level database operations

## When to Use CRUDService

Use CRUDService when:

- you need custom business logic between routes and database
- you're building custom endpoints with special validation
- you want to reuse CRUD logic across multiple endpoints
- you want automatic HTTP error handling for 404, validation, etc.

## Quick Setup

```python
from fastapi_crudrouter_mongodb import CRUDService, MongoModel, ObjectIdType
from typing import Annotated
import motor.motor_asyncio

ObjectIdType = Annotated[ObjectId, MongoObjectId]


class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str


# Initialize the service
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.myapp

service = CRUDService(
    model=UserModel,
    db=db,
    collection_name="users",
)

# Use service in your custom endpoint
@app.get("/users")
async def list_users():
    return await service.find_all()
```

## Core Methods

All methods are async and include automatic HTTP error handling.

### find_all()

Retrieve all documents with optional filtering, sorting, and pagination.

```python
async def find_all(
    skip: int | None = None,
    limit: int | None = None,
    sort_by: str | None = None,
    order_by: str | None = None,
    filters: dict | None = None,
    populates: list | None = None,
) -> list[Any]:
```

**Parameters:**

- `skip`: number of documents to skip (pagination)
- `limit`: maximum documents to return (pagination)
- `sort_by`: field name to sort by
- `order_by`: sort direction—`"ASC"` or `"DESC"`
- `filters`: MongoDB filter document (e.g., `{"status": "active"}`)
- `populates`: list of `CRUDPopulate` objects to resolve references

**Returns:** List of documents (populated if requested).

**Example:**

```python
from fastapi_crudrouter_mongodb import CRUDPopulate

artist_populate = CRUDPopulate(
    field="artist_ids",
    collection="artists",
    model=ArtistModel,
)

tracks = await service.find_all(
    filters={"genre": "rock"},
    sort_by="title",
    order_by="ASC",
    populates=[artist_populate],
)
```

### find_one()

Retrieve a single document by identifier.

```python
async def find_one(
    id: str,
    populates: list | None = None,
) -> Callable[..., Any]:
```

**Parameters:**

- `id`: document identifier
- `populates`: list of `CRUDPopulate` objects to resolve references

**Returns:** The document (populated if requested).

**Errors:** Raises `HTTPException(400, "Document not found")` if not found.

**Example:**

```python
user = await service.find_one("507f1f77bcf86cd799439011")
# Raises HTTP 400 if not found
```

### create_one()

Create a new document.

```python
async def create_one(
    data,
    *args: Any,
    **kwargs: Any,
) -> Callable[..., Any]:
```

**Parameters:**

- `data`: an instance of your MongoModel with values to insert

**Returns:** The created document.

**Errors:** Raises `HTTPException(422, "Document not created")` if creation fails.

**Example:**

```python
new_user = UserModel(name="Bob", email="bob@example.com")
created = await service.create_one(new_user)
```

### replace_one()

Replace an entire document by id.

```python
async def replace_one(
    id: str,
    data,
    *args: Any,
    **kwargs: Any,
) -> Callable[..., Any]:
```

**Parameters:**

- `id`: identifier of document to replace
- `data`: new document content

**Returns:** The replaced document.

**Errors:** Raises `HTTPException(422, "Document not replaced")` if not found or fails.

**Example:**

```python
updated_user = UserModel(name="Robert", email="robert@example.com")
result = await service.replace_one("507f1f77bcf86cd799439011", updated_user)
```

### update_one()

Partially update specific fields of a document.

```python
async def update_one(
    id: str,
    data,
    *args: Any,
    **kwargs: Any,
) -> Callable[..., Any]:
```

**Parameters:**

- `id`: identifier of document to update
- `data`: fields to update

**Returns:** The updated document.

**Errors:** Raises `HTTPException(422, "Document not updated")` if not found or fails.

**Example:**

```python
partial = UserModel(email="newemail@example.com")
result = await service.update_one("507f1f77bcf86cd799439011", partial)
```

### delete_one()

Delete a document by id.

```python
async def delete_one(id: str, *args: Any, **kwargs: Any) -> Response:
```

**Parameters:**

- `id`: identifier of document to delete

**Returns:** HTTP 204 No Content response.

**Errors:** Raises `HTTPException(422, "Document not deleted")` if not found or fails.

**Example:**

```python
response = await service.delete_one("507f1f77bcf86cd799439011")
# Returns 204 No Content status
```

## Automatic Populate Resolution

CRUDService automatically resolves referenced documents using `CRUDPopulate`.

### Single Document with Populate

```python
from fastapi_crudrouter_mongodb import CRUDPopulate

# Define what to populate
author_populate = CRUDPopulate(
    field="author_id",
    collection="authors",
    model=AuthorModel,
)

# Fetch and populate
book = await service.find_one(
    "507f1f77bcf86cd799439011",
    populates=[author_populate],
)

# book.author_id is now a full AuthorModel, not just an ObjectId
```

### Multiple Documents with Populate

```python
books = await service.find_all(
    populates=[author_populate],
)

# All books now have fully resolved author_ids
```

### Multiple Populate Relationships

```python
author_populate = CRUDPopulate(
    field="author_id",
    collection="authors",
    model=AuthorModel,
)

publisher_populate = CRUDPopulate(
    field="publisher_id",
    collection="publishers",
    model=PublisherModel,
)

books = await service.find_all(
    populates=[author_populate, publisher_populate],
)

# Both author_id and publisher_id are resolved
```

## Error Handling

CRUDService automatically converts repository errors to HTTP exceptions:

| Error Case                      | HTTP Status | Message                     |
|---------------------------------|-------------|---------------------------|
| Document not found              | 400         | Document not found         |
| Create fails                    | 422         | Document not created       |
| Replace fails                   | 422         | Document not replaced      |
| Update fails                    | 422         | Document not updated       |
| Delete fails                    | 422         | Document not deleted       |
| Circular populate reference     | 422         | Circular reference detected|

### Example: Catching Errors

```python
from fastapi import HTTPException

try:
    user = await service.find_one("invalid-id")
except HTTPException as e:
    print(f"Error: {e.status_code} - {e.detail}")
    # Error: 400 - Document not found
```

## Output Schemas

Like `CRUDRepository` and `CRUDRouter`, CRUDService supports output schemas:

```python
class UserModelOut(MongoModel):
    id: str
    name: str
    email: str
    # password is intentionally omitted


service = CRUDService(
    model=UserModel,
    db=db,
    collection_name="users",
    model_out=UserModelOut,  # All responses use this schema
)

user = await service.find_one("507f1f77bcf86cd799439011")
# user is now a UserModelOut instance (no password exposed)
```

## Custom Identifier Fields

Use custom fields (like email) instead of MongoDB's `_id`:

```python
service = CRUDService(
    model=UserModel,
    db=db,
    collection_name="users",
    identifier_field="email",
)

# Now find by email
user = await service.find_one("alice@example.com")
```

## Deprecated Methods

The following methods are deprecated but still functional:

- `get_all()` → use `find_all()` instead
- `get_one(id)` → use `find_one(id)` instead

## How It Works: Internal Serialization

CRUDService automatically serializes populated documents so that:

1. Referenced documents are resolved to full objects
2. Nested lists of references are handled correctly
3. Model aliases are respected in JSON output
4. Output schemas are applied if defined

This happens transparently—you just request populate and get the resolved data.

## Related

- [CRUDRouter](../routers/CRUDRouter.md): HTTP router built on top of CRUDService
- [CRUDRepository](./CRUDRepository.md): low-level database access layer
- [CRUDPopulate](../routers/CRUDPopulate.md): configure reference resolution
- [MongoModel](../models/MongoModel.md): base model class
