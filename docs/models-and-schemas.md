
# Models and Schemas

`CRUDRouter` uses two model types to build your API:

- an input model (`model`) that represents data stored in MongoDB
- an output schema (`model_out`) that represents data returned to API clients

```python
CRUDRouter(
    model=MyMongoModel,
    model_out=MyOutputMongoModel,
    ...
)
```

## Why Use Two Models?

In many APIs, the database model contains fields that should not be exposed to clients (for example, `password`, internal flags, or private notes).

Using a separate output schema helps you:

- hide sensitive fields
- keep response payloads clean
- make API contracts explicit and predictable

## Input Model vs Output Schema

- **Input model (`model`)**: full structure stored in the database
- **Output schema (`model_out`)**: safe, public shape returned by routes

```python
from typing import Annotated
from fastapi_crudrouter_mongodb import (
    MongoModel,
    MongoObjectId,
    ObjectId,
)


class UserModel(MongoModel):
    # Database model (includes sensitive fields)
    id: Annotated[ObjectId, MongoObjectId] | None = None
    name: str
    email: str
    password: str


class UserModelOut(MongoModel):
    # Response schema (safe for API clients)
    id: str
    name: str
    email: str
```

In this example, `password` is stored in MongoDB but never returned in API responses.

## Automatic Output Schema Generation

If you omit `model_out`, `CRUDRouter` automatically generates an output schema and documents it in OpenAPI.

```python
CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
)
```

By default, fields matching the primary key behavior are handled server-side and reflected appropriately in generated response models.

## Recommended Practice

For production APIs, define `model_out` explicitly whenever possible. This gives you better control over security, response shape, and long-term API stability.