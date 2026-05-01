# 3. CRUD Operations & Error Handling

Master creating, reading, updating, and deleting data with proper error handling.

## The Four CRUD Operations

| Operation | HTTP | Endpoint | Purpose |
|-----------|------|----------|---------|
| **C**reate | POST | `/resource` | Insert new document |
| **R**ead | GET | `/resource/{id}` | Fetch document(s) |
| **U**pdate | PATCH | `/resource/{id}` | Modify fields |
| **D**elete | DELETE | `/resource/{id}` | Remove document |

## Getting Started

We'll use the music API from the previous guide. Update `music_api.py`:

```python
import motor.motor_asyncio
from fastapi import FastAPI
from fastapi_crudrouter_mongodb import CRUDRouter, MongoModel, ObjectIdType

ObjectIdType = Annotated[ObjectId, MongoObjectId]


# Models (same as before)
class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str
    password: str


class UserModelOut(MongoModel):
    id: str
    name: str
    email: str


# Setup
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.music_app

users_router = CRUDRouter(
    model=UserModel,
    model_out=UserModelOut,
    db=db,
    collection_name="users",
    prefix="/users",
)

app = FastAPI()
app.include_router(users_router)
```

## CREATE: Adding Data

### Insert One Document

**Endpoint:** `POST /users`

**Request:**
```json
{
  "name": "Bob Johnson",
  "email": "bob@example.com",
  "password": "secure123"
}
```

**Response:** `201 Created`
```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Bob Johnson",
  "email": "bob@example.com"
}
```

### Error: Duplicate Insert

If you try to create the same user twice (with same id), you get:

**Response:** `422 Unprocessable Entity`
```json
{
  "detail": "Document not created"
}
```

## READ: Fetching Data

### Get All Documents

**Endpoint:** `GET /users`

**Response:** `200 OK`
```json
[
  {
    "id": "507f1f77bcf86cd799439011",
    "name": "Alice",
    "email": "alice@example.com"
  },
  {
    "id": "507f1f77bcf86cd799439012",
    "name": "Bob Johnson",
    "email": "bob@example.com"
  }
]
```

### Get One Document

**Endpoint:** `GET /users/507f1f77bcf86cd799439011`

**Response:** `200 OK`
```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Alice",
  "email": "alice@example.com"
}
```

### Error: Not Found

**Endpoint:** `GET /users/invalid-id`

**Response:** `400 Bad Request`
```json
{
  "detail": "Document not found"
}
```

## UPDATE: Modifying Data

### Partial Update (PATCH)

Update only the fields you specify. Other fields stay unchanged.

**Endpoint:** `PATCH /users/507f1f77bcf86cd799439011`

**Request:**
```json
{
  "email": "alice.new@example.com"
}
```

**Response:** `200 OK`
```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Alice",
  "email": "alice.new@example.com"  // ← Updated
}
```

### Full Replacement (PUT)

Replace the entire document. All fields must be provided.

**Endpoint:** `PUT /users/507f1f77bcf86cd799439011`

**Request:**
```json
{
  "name": "Alice Smith",
  "email": "alice.smith@example.com",
  "password": "newsecure456"
}
```

**Response:** `200 OK`
```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Alice Smith",
  "email": "alice.smith@example.com"
}
```

### PATCH vs PUT

- **PATCH** - "change these fields"
- **PUT** - "replace entire document"

## DELETE: Removing Data

### Delete a Document

**Endpoint:** `DELETE /users/507f1f77bcf86cd799439011`

**Response:** `204 No Content` (empty body, just the status)

The user is now gone from the database.

### Error: Already Deleted

**Endpoint:** `DELETE /users/507f1f77bcf86cd799439011` (again)

**Response:** `422 Unprocessable Entity`
```json
{
  "detail": "Document not deleted"
}
```

## Common HTTP Status Codes

| Code | Meaning | When You See It |
|------|---------|-----------------|
| **200** | OK | Successful GET, PATCH, PUT |
| **201** | Created | Successful POST |
| **204** | No Content | Successful DELETE |
| **400** | Bad Request | Document not found, invalid id |
| **422** | Unprocessable Entity | Create/update/delete failed |

## Error Handling in Python

If you're building a client, handle errors:

```python
import httpx

async def get_user(user_id: str):
    async with httpx.AsyncClient() as client:
        response = await client.get(f"http://localhost:8000/users/{user_id}")
        
        if response.status_code == 200:
            return response.json()  # Success
        elif response.status_code == 400:
            print("User not found")
            return None
        elif response.status_code == 422:
            print("Error:", response.json()["detail"])
            return None
```

## Practical Example: Create-Read-Update-Delete Cycle

Here's a typical workflow:

```python
# 1. CREATE
new_user = {
    "name": "Charlie",
    "email": "charlie@example.com",
    "password": "pass123"
}
# POST /users → Returns: {"id": "123", ...}

# 2. READ
# GET /users/123 → Returns the user

# 3. UPDATE
update_data = {"email": "charlie.new@example.com"}
# PATCH /users/123 → Returns: {"id": "123", "email": "charlie.new@example.com", ...}

# 4. DELETE
# DELETE /users/123 → Returns: 204 No Content
```

## What's Automatic

The CRUDRouter automatically:

✅ Validates incoming data against your model  
✅ Generates appropriate HTTP status codes  
✅ Handles errors gracefully  
✅ Converts MongoDB ObjectIds to strings in responses  
✅ Applies output schemas  

You don't write any of this—it's all built-in.

## Best Practices

1. **Always check status codes** in client code
2. **Use PATCH for partial updates** (more efficient)
3. **Use PUT only when replacing entire documents**
4. **Handle 404/400 errors** for missing resources
5. **Handle 422 errors** for data validation failures

## Next Steps

Continue learning:

- [**Filtering & Pagination**](04-filtering-sorting-pagination.md) - Query and sort your data
- [**Related Data**](05-related-data.md) - Work with relationships (populates, lookups, embeds)
- [**Back to Quickstart**](index.md)
