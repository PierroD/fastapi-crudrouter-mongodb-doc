# 2. Models & Output Schemas

Learn how to structure your data and protect sensitive fields.

## The Problem

Imagine you have a user with a password. You don't want that password exposed in API responses!

```json
// Bad: Password exposed to clients
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Alice",
  "email": "alice@example.com",
  "password": "super-secret-123"  // ❌ Exposed!
}
```

This is where **output schemas** come in.

## Database Model vs Output Schema

- **Database Model** (`model`): represents everything stored in MongoDB (including secrets)
- **Output Schema** (`model_out`): represents safe data sent to API clients

```python
from fastapi_crudrouter_mongodb import MongoModel, ObjectIdType
from typing import Annotated

# Full database model
class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str
    password: str  # ← Secret stored in DB


# Public response schema
class UserModelOut(MongoModel):
    id: str
    name: str
    email: str
    # password intentionally omitted!
```

Now when clients call your API, they get:

```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Alice",
  "email": "alice@example.com"
}
```

Password never exposed. ✅

## Complete Example: Music API

Let's build a music streaming API. Create `music_api.py`:

```python
import motor.motor_asyncio
from fastapi import FastAPI
from fastapi_crudrouter_mongodb import CRUDRouter, MongoModel, ObjectIdType
from typing import Annotated

ObjectIdType = Annotated[ObjectId, MongoObjectId]


# ===== Database Models (include secrets) =====

class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str
    password: str  # Secret - don't expose
    premium: bool = False
    created_at: str | None = None


class ArtistModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    bio: str


class TrackModel(MongoModel):
    id: ObjectIdType | None = None
    title: str
    artist_id: ObjectIdType
    duration_seconds: int
    plays: int = 0


# ===== Output Schemas (safe for public API) =====

class UserModelOut(MongoModel):
    id: str
    name: str
    email: str
    premium: bool
    # password is hidden ✓
    # created_at is hidden ✓


class ArtistModelOut(MongoModel):
    id: str
    name: str
    bio: str


class TrackModelOut(MongoModel):
    id: str
    title: str
    artist_id: str
    duration_seconds: int
    plays: int


# ===== Setup =====

client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.music_app


# ===== Routers with Output Schemas =====

users_router = CRUDRouter(
    model=UserModel,
    model_out=UserModelOut,  # ← Use output schema
    db=db,
    collection_name="users",
    prefix="/users",
    tags=["Users"],
)

artists_router = CRUDRouter(
    model=ArtistModel,
    model_out=ArtistModelOut,
    db=db,
    collection_name="artists",
    prefix="/artists",
    tags=["Artists"],
)

tracks_router = CRUDRouter(
    model=TrackModel,
    model_out=TrackModelOut,
    db=db,
    collection_name="tracks",
    prefix="/tracks",
    tags=["Tracks"],
)


# ===== App =====

app = FastAPI(title="Music API")
app.include_router(users_router)
app.include_router(artists_router)
app.include_router(tracks_router)
```

## Test It

Run the app:

```console
python music_api.py
```

Go to http://localhost:8000/docs

### Create a User

**POST /users**:

```json
{
  "name": "Alice",
  "email": "alice@example.com",
  "password": "my-secure-password"
}
```

**Response** (password hidden):

```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Alice",
  "email": "alice@example.com",
  "premium": false
}
```

### Create an Artist

**POST /artists**:

```json
{
  "name": "Taylor Swift",
  "bio": "American singer-songwriter"
}
```

## What Fields to Hide

Common sensitive fields to exclude from `model_out`:

- `password` - never expose
- `api_key` - internal only
- `internal_id` - for linking
- `created_at` - timestamp metadata
- `updated_at` - timestamp metadata
- `deleted` - internal state
- `is_admin` - security risk
- `payment_info` - PII
- `user_agents` - tracking data
- Any field prefixed with `_` - usually internal

## Best Practices

1. **Always define `model_out` for production APIs** - don't rely on auto-generation
2. **Be explicit about what's public** - easier to review and maintain
3. **Use descriptive names** - `UserModelOut` makes intent clear
4. **Inherit when possible** - reduce duplication:

```python
class UserModelOut(MongoModel):
    # Reuse base fields
    id: str
    name: str
    email: str


class AdminUserModelOut(UserModelOut):
    # Extend for admin context
    is_admin: bool
    created_at: str
```

## No Output Schema Needed?

If your model doesn't have secrets, you can skip `model_out`:

```python
class PublicArtistModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    bio: str
    # No secrets - safe to use as-is


router = CRUDRouter(
    model=PublicArtistModel,
    # model_out omitted - uses model automatically
    ...
)
```

## Next Steps

Continue learning:

- [**CRUD Operations**](03-crud-operations.md) - Error handling and edge cases
- [**Filtering & Pagination**](04-filtering-sorting-pagination.md) - Query your data
- [**Back to Quickstart**](index.md)
