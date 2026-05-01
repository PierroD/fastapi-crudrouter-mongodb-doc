

# Releases

This page tracks notable changes across released versions of `fastapi-crudrouter-mongodb`.

The format is intentionally changelog-style so you can quickly scan for breaking changes, new features, fixes, and migration impact.

## v1.0.0

**Date:** 2025-05-02

### Highlights

- Added `CRUDPopulate` for automatic population of referenced documents
- Continued internal refactoring
- Expanded test coverage

### Added

`CRUDPopulate` lets you resolve referenced ObjectIds into full documents from another collection. This is especially useful when your MongoDB models store relationships as ids but your API responses need the related data expanded.

## v0.1.4

**Date:** Silent release

### Added

- Added `CRUDRepository`
- Added support for custom identifier fields instead of only `_id`

### Changed

- Refactored the internals to use a repository pattern

## v0.1.3

**Date:** 2024-12-17

### Highlights

- Reuse the service attached to an existing `CRUDRouter`
- Create your own service layer for custom endpoints

### Example

```python
from fastapi_crudrouter_mongodb import CRUDRouterService


@app.get("/get_one/{id}")
async def get_one(id: str):
    # Assuming you already have a users_router CRUDRouter instance
    return await users_router.service.get_one(id)


class TestModel(MongoModel):
    id: Annotated[ObjectId, MongoObjectId] | None = None
    name: str


service = CRUDRouterService(TestModel, db, "test")


@app.get("/test")
async def test():
    return await service.get_all()
```

## v0.1.2

**Date:** 2024-08-21

### Breaking Change

- `MongoModel` now uses camelCase as the default input and output style

You do not need to manually transform field names. The library converts snake_case fields to camelCase for FastAPI input and output.

### Output Example

Previous output:

```json
{
  "user_id": "..."
}
```

New output:

```json
{
  "userId": "..."
}
```

## v0.1.1

**Date:** 2024-04-13

### Fixed

- `convert_to` can now replace every `ObjectId` inside dicts and lists

## v0.1.0

**Date:** 2024-03-31

### Breaking Changes

- Typing changed significantly due to the Pydantic v2 migration
- Embeds must now be declared explicitly

### Added

- Migrated the library to Pydantic v2
- Added output schema support to `CRUDLookup`
- Added the new `CRUDEmbed` router

### Fixed

- DELETE routes now return the expected schema
- Lookup fields are removed automatically from the parent schema when needed

### Documentation

- Added new documentation pages
- Added migration guides: [Migration Guide](migrate.md)

## v0.0.8

**Date:** 2023-09-13

### Added

- Added `model_out` to `CRUDRouter` so you can hide sensitive fields in responses
- Added `black` for formatting
- Added `ruff` for linting

### Fixed

- Fixed `convert_to` so it converts `id` to `str` when needed for `MongoObjectId` values

### Example

```python
class UserModel(MongoModel):
    id: Optional[MongoObjectId] = Field()
    name: str
    email: str
    password: str
    addresses: Optional[List[AddressModel]]
    messages: Optional[Union[List[MessageModel], MessageModel]] = None


class UserModelOut(MongoModel):
    id: str
    name: str
    email: str


users_controller = CRUDRouter(
    model=UserModel,
    model_out=UserModelOut,
    db=db,
    collection_name="users",
    lookups=[messages_lookup],
    prefix="/users",
    tags=["users"],
)
```

### Result

![CRUDRouter output schema example](https://github.com/PierroD/fastapi-crudrouter-mongodb/assets/40763427/b30f73dc-b545-4e67-93cc-33338902b170)

## v0.0.7

**Date:** 2023-08-13

### Changed

- Forced Pydantic v1.x dependency to prevent crashes at the time
- Refactored `mongo` and renamed it to `to_mongo`

### Added

- Added the `convert_to` helper for converting one model into another

### Example

```python
@app.get("/{id}", response_model=UserModelDAO)
async def root(id: str):
    response = await db["users"].find_one({"_id": MongoObjectId(id)})
    user_dao = UserModelDAO.from_mongo(response)
    user_dto = user_dao.convert_to(UserModelDTO)
    return user_dto
```

## v0.0.5

**Date:** 2023-05-10

### Added

- Added support for disabling individual `CRUDRouter` routes
- Added support for route-specific dependencies

### Example

```python
user = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    tags=["users"],
    disable_get_all=True,
    dependencies_get_one=[Depends(verify_admin)],
)
```

### New `CRUDRouter` Parameters

| Parameter                  | Default | Type              | Description                  |
|---------------------------|---------|-------------------|------------------------------|
| `disable_get_all`         | `False` | `bool`            | Disable the get-all route    |
| `disable_get_one`         | `False` | `bool`            | Disable the get-one route    |
| `disable_create_one`      | `False` | `bool`            | Disable the create route     |
| `disable_replace_one`     | `False` | `bool`            | Disable the replace route    |
| `disable_update_one`      | `False` | `bool`            | Disable the update route     |
| `disable_delete_one`      | `False` | `bool`            | Disable the delete route     |
| `dependencies_get_all`    | `None`  | `Sequence[Depends]` | Add custom dependencies    |
| `dependencies_get_one`    | `None`  | `Sequence[Depends]` | Add custom dependencies    |
| `dependencies_create_one` | `None`  | `Sequence[Depends]` | Add custom dependencies    |
| `dependencies_replace_one`| `None`  | `Sequence[Depends]` | Add custom dependencies    |
| `dependencies_update_one` | `None`  | `Sequence[Depends]` | Add custom dependencies    |
| `dependencies_delete_one` | `None`  | `Sequence[Depends]` | Add custom dependencies    |

## v0.0.4

**Date:** 2023-04-10

### Initial Release

The first public release introduced automatic CRUD route generation for MongoDB-backed FastAPI projects.

### Project Links

- Documentation: [fastapi-crudrouter-mongodb-doc](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/)
- Source code: [pierrod/fastapi-crudrouter-mongodb](https://github.com/pierrod/fastapi-crudrouter-mongodb)

### Credits

- Base project and original idea: [awtkns/fastapi-crudrouter](https://github.com/awtkns/fastapi-crudrouter)
- `_id` to `id` conversion reference: [FastAPI issue discussion](https://github.com/tiangolo/fastapi/issues/1515)

### Installation

```bash
pip install fastapi-crudrouter-mongodb
```

### Basic Example

```python
from datetime import datetime
from typing import List, Optional, Union

from fastapi import FastAPI
from pydantic import Field
import motor.motor_asyncio
from fastapi_crudrouter_mongodb import (
    CRUDLookup,
    CRUDRouter,
    MongoModel,
    MongoObjectId,
)


client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017/local")
db = client.local


class MessageModel(MongoModel):
    id: Optional[MongoObjectId] = Field()
    message: str
    user_id: MongoObjectId
    created_at: Optional[str] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    updated_at: Optional[str] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")


class AddressModel(MongoModel):
    id: Optional[MongoObjectId] = Field()
    street: str
    city: str
    state: str
    zip: str


class UserModel(MongoModel):
    id: Optional[MongoObjectId] = Field()
    name: str
    email: str
    addresses: Optional[List[AddressModel]]
    messages: Optional[Union[List[MessageModel], MessageModel]] = None


messages_lookup = CRUDLookup(
    model=MessageModel,
    collection_name="messages",
    prefix="messages",
    local_field="_id",
    foreign_field="user_id",
)

users_controller = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    lookups=[messages_lookup],
    prefix="/users",
    tags=["users"],
)

app = FastAPI()
app.include_router(users_controller)
```

### OpenAPI Support

All generated routes are automatically documented in the OpenAPI schema.

![CRUDRouter full OpenAPI image](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/assets/img/crud-router-full.png)
