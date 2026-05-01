
# Migration Guide (v0.1.0+ / Pydantic v2)

Starting with version `0.1.0`, this project uses **Pydantic v2**.

This guide shows the most important changes needed to migrate existing code.

## What Changed

The migration mainly affects:

- imports used for typing and models
- id field typing (`Annotated[ObjectId, MongoObjectId] | None`)
- explicit embed configuration with `CRUDEmbed`

## Updated Imports

Use these imports as a baseline in migrated projects:

```python
import motor.motor_asyncio

from typing import Optional, Union
from fastapi import FastAPI
from fastapi_crudrouter_mongodb import (
    CRUDEmbed,
    CRUDLookup,
    CRUDRouter,
    MongoModel,
    ObjectIdType
)
```

## Typing Changes

### 1) ID Field Type

The id field moved from the old optional style to an annotated ObjectId type.

```python
# Old (pre-v0.1.0)
class OldModel(MongoModel):
    id: Optional[MongoObjectId] = Field()


# New (v0.1.0+)
class NewModel(MongoModel):
    id: ObjectIdType | None = None
```

### 2) Embed Model Declaration

Embed fields still live on the parent model, but now they should be configured explicitly with `CRUDEmbed` in the router.

```python
from typing import Annotated, Optional


# Old (pre-v0.1.0)
class OldEmbedModel(MongoModel):
    id: Optional[MongoObjectId] = Field()
    field: str


class OldMainModel(MongoModel):
    id: Optional[MongoObjectId] = Field()
    field: str
    embeds: Optional[list[OldEmbedModel]]


# New (v0.1.0+)
class NewEmbedModel(MongoModel):
    id: ObjectIdType | None = None
    field: str


class NewMainModel(MongoModel):
    id: ObjectIdType | None = None
    field: str
    embeds: Optional[list[NewEmbedModel]] = []


new_embed_router = CRUDEmbed(
    model=NewEmbedModel,
    embed_name="embeds",
)


new_router = CRUDRouter(
    model=NewMainModel,
    db=db,
    collection_name="users",
    embeds=[new_embed_router],
    prefix="/users",
    tags=["users"],
)
```

## Migration Checklist

Use this checklist when upgrading an existing codebase:

1. Replace old id typing with `ObjectIdType | None = None`.
2. Update imports to include `Annotated` and `ObjectId`.
3. Ensure embed fields are declared on the parent model as lists (or optional lists).
4. Register embeds explicitly with `CRUDEmbed(...)` and pass them via `embeds=[...]`.
5. Run your app and verify OpenAPI docs and CRUD routes behave as expected.