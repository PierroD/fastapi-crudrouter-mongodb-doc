# CRUDPopulate

`CRUDPopulate` declares how a reference field should be resolved from ObjectIds to full documents.

## Signature

```python
CRUDPopulate(field: str, collection: str, model)
```

- `field`: field name on the parent model (for example `artist_ids`)
- `collection`: target MongoDB collection name (for example `artists`)
- `model`: model used to deserialize resolved documents

## End-to-End Example

```python
from typing import Annotated
from bson import ObjectId
from fastapi import FastAPI
from fastapi_crudrouter_mongodb import (
    CRUDPopulate,
    CRUDRouter,
    MongoModel,
    MongoObjectId,
)
import motor.motor_asyncio

ObjectIdType = Annotated[ObjectId, MongoObjectId]


class Artist(MongoModel):
    id: ObjectIdType | None = None
    name: str


class Track(MongoModel):
    id: ObjectIdType | None = None
    title: str
    artist_ids: list[ObjectIdType] = []


app = FastAPI()
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017/local")
db = client.local

tracks_router = CRUDRouter(
    model=Track,
    db=db,
    collection_name="tracks",
    prefix="/tracks",
    populates=[
        CRUDPopulate(field="artist_ids", collection="artists", model=Artist),
    ],
)

app.include_router(tracks_router)
```

Now `GET /tracks` and `GET /tracks/{id}` resolve `artist_ids` with documents from `artists`.

## Batch `$in` Resolution

Population is resolved in batch:

- collect all referenced ObjectIds across the response documents
- execute one `$in` query per populate declaration
- map results back to each document

This avoids N+1 query patterns.

## Missing References

If a referenced ObjectId cannot be found, it is omitted from the resolved result.

## Circular Reference Guard

If `populate.collection` equals the parent router collection, the request fails with `422` and a clear message, for example:

```text
Circular reference detected: populate field 'related_ids' points to collection 'tracks', which matches parent collection 'tracks'.
```
