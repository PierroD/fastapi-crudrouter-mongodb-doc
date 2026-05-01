# CRUDPopulate

**CRUDPopulate** is a configuration object used by `CRUDRouter` to describe how a reference field should be populated.

It does not perform database work by itself. Instead, you pass it to `CRUDRouter(populates=[...])`, and the router/service layer uses that configuration when resolving related documents.

## Constructor

```python
CRUDPopulate(
  field: str,
  collection: str,
  model: type,
  model_out: type[BaseModel] | None = None,
)
```

## What Problem Does It Solve?

When you store references in MongoDB, your document often contains ids rather than full nested objects. `CRUDPopulate` tells the router how to resolve those ids from another collection.

For example, without population a track may look like this:

```json
{
  "id": "track123",
  "title": "Beautiful Song",
  "artist_ids": ["artist456", "artist789"]
}
```

With a matching `CRUDPopulate` declaration, `CRUDRouter` can resolve those ids into documents from another collection.

## How to Configure It

```python
CRUDPopulate(
    field="artist_ids",       # Field name in your model containing IDs
    collection="artists",     # MongoDB collection to fetch documents from
  model=Artist,              # Model class for the fetched documents
  model_out=ArtistOut,       # Optional output schema for populated documents
)
```

### Parameters

- **`field`**: Name of the field containing reference ids. Must be a non-empty string.
- **`collection`**: Name of the target MongoDB collection. Must be a non-empty string.
- **`model`**: `MongoModel` subclass used to deserialize populated documents.
- **`model_out`**: Optional `BaseModel` subclass used to serialize populated documents.

## Validation Rules

The current implementation validates constructor input eagerly and raises `ValueError` in these cases:

- `field` is missing, empty, or not a string
- `collection` is missing, empty, or not a string
- `model` is not a `MongoModel` subclass
- `model_out` is provided but is not a `BaseModel` subclass

### Example Errors

```python
CRUDPopulate(field="", collection="artists", model=Artist)
# ValueError: CRUDPopulate: 'field' must be a non-empty string

CRUDPopulate(field="artist_ids", collection="", model=Artist)
# ValueError: CRUDPopulate: 'collection' must be a non-empty string
```

## Complete Example: Music Tracks & Artists

This example shows `CRUDPopulate` used with `CRUDRouter`.

```python
from fastapi import FastAPI
from pydantic import Field
import motor.motor_asyncio
from fastapi_crudrouter_mongodb import (
    CRUDPopulate,
    CRUDRouter,
    MongoModel,
  ObjectIdType,
)

# Define your models
class Artist(MongoModel):
    id: ObjectIdType | None = None
    name: str
    genre: str


class ArtistOut(MongoModel):
  id: str
  name: str
  genre: str


class Track(MongoModel):
    id: ObjectIdType | None = None
    title: str
  artist_ids: list[ObjectIdType] = Field(default_factory=list)


class TrackOut(MongoModel):
  id: str
  title: str
  artist_ids: list[ArtistOut] = Field(default_factory=list)


# Setup FastAPI and MongoDB
app = FastAPI()
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017/local")
db = client.local

tracks_router = CRUDRouter(
    model=Track,
  model_out=TrackOut,
    db=db,
    collection_name="tracks",
    prefix="/tracks",
  populates=[
        CRUDPopulate(
      field="artist_ids",
      collection="artists",
      model=Artist,
      model_out=ArtistOut,
        ),
    ],
)

app.include_router(tracks_router)
```

### What Happens Now

When clients call your API:

```http
GET /tracks
```

The response automatically includes full artist details:

```json
[
  {
    "id": "507f1f77bcf86cd799439011",
    "title": "Beautiful Song",
    "artist_ids": [
      {
        "id": "507f1f77bcf86cd799439012",
        "name": "John Doe",
        "genre": "Rock"
      },
      {
        "id": "507f1f77bcf86cd799439013",
        "name": "Jane Smith",
        "genre": "Jazz"
      }
    ]
  }
]
```

Both `GET /tracks` and `GET /tracks/{id}` automatically resolve `artist_ids` with documents from the `artists` collection.

## When to Use `model_out`

Use `model_out` when you want the populated documents to be serialized differently from the database model.

For example, you may want to hide internal fields from populated documents just as you do with top-level router responses.

```python
populate = CRUDPopulate(
    field="artist_ids",
    collection="artists",
    model=Artist,
    model_out=ArtistOut,
)
```

If you do not need a separate schema for populated documents, you can omit `model_out`.

## Notes

- `CRUDPopulate` only defines how a field should be resolved.
- The actual population behavior happens inside the router/service layer.
- This class does not itself query MongoDB, detect circular references, or serialize responses.

## Related Features

- [CRUDRouter](CRUDRouter.md): where `populates=[...]` is configured
- [CRUDLookup](CRUDLookup.md): nested child routes for related collections
- [CRUDEmbed](CRUDEmbed.md): embedded child models stored inside the parent document
