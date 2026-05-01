# MongoModel

**MongoModel** is the base model used throughout the library for MongoDB documents.

It adds helper methods for converting MongoDB data to and from Pydantic-style models, especially around MongoDB's `_id` field.

Use MongoModel whenever you define a model that will be stored in MongoDB and used with `CRUDRouter`.

## Quick Import

```python
from fastapi_crudrouter_mongodb import MongoModel


class UserModel(MongoModel):
    # Your fields here
    pass
```

## Common ID Pattern

Most models in this library define an `id` field typed with `ObjectIdType` or `Annotated[ObjectId, MongoObjectId]`.

That is the recommended pattern when the model represents a MongoDB document id:

```python
from fastapi_crudrouter_mongodb import MongoModel, ObjectIdType


class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str
```

### How the ID Field Works

- `None` when creating new documents (MongoDB auto-generates the id)
- stored as `_id` in MongoDB (the standard MongoDB primary key)
- returned as `id` in API responses (for REST convention)
- automatically handled by `CRUDRouter`

`MongoModel` itself does not enforce the presence of an `id` field, but most router-compatible document models should define one.

## Inheritance

`MongoModel` extends the library's `CamelModel`, which in turn behaves like a normal Pydantic model.

That means you still get standard Pydantic features such as:

- field validation
- type hints
- Field() for additional field options
- model_validate, model_dump, etc.

```python
from fastapi_crudrouter_mongodb import MongoModel, ObjectIdType
from pydantic import Field


class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str = Field(..., min_length=1)
    email: str
    age: int = Field(default=0, ge=0, le=150)
```

## Conversion Methods

MongoModel provides helpers to convert between Python objects, dicts, and MongoDB documents:

| Method      | Description                                          | Status     |
|-------------|------------------------------------------------------|------------|
| `from_mongo` | Convert MongoDB BSON to a MongoModel instance        | ✅ Active  |
| `to_mongo`   | Convert a MongoModel to a MongoDB-ready dict         | ✅ Active  |
| `mongo`      | Convert to dict (alias for to_mongo)                 | ⚠️ Deprecated |
| `convert_to` | Convert a MongoModel to another Pydantic model       | ✅ Active  |

### Method Signatures

```python
@classmethod
def from_mongo(cls, data: dict)

def mongo(self, add_id: bool = False, **kwargs)

def to_mongo(self, add_id: bool = False, by_alias: bool = False, **kwargs)

def convert_to(self, model)
```

### Example: from_mongo

```python
from bson import ObjectId
from fastapi_crudrouter_mongodb import MongoModel, ObjectIdType


class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str


# Simulate data from MongoDB
bson_data = {"_id": ObjectId(), "name": "John"}
user = UserModel.from_mongo(bson_data)
print(user.id)  # ✓ ObjectId, converted from _id
```

`from_mongo` returns the input unchanged when `data` is falsy.

### Example: to_mongo

```python
user = UserModel(name="Jane")
mongo_dict = user.to_mongo()
# Result: {"name": "Jane"}
```

`to_mongo`:

- calls `model_dump(exclude_none=True, by_alias=True, ...)`
- renames `id` to `_id` when needed
- optionally generates `_id` when `add_id=True`

```python
user = UserModel(name="Jane")
mongo_dict = user.to_mongo(add_id=True)
# Result includes a generated "_id"
```

### Example: mongo

```python
user = UserModel(name="Jane")
mongo_dict = user.mongo()
```

`mongo()` is deprecated and the implementation explicitly recommends using `to_mongo()` instead.

### Example: convert_to

```python
class UserModelOut(MongoModel):
    id: str
    name: str


user = UserModel(id=ObjectId(), name="Alice")
user_out = user.convert_to(UserModelOut)
# Converts to output schema and turns ObjectId values into strings
```

`convert_to` recursively converts:

- `ObjectId` values to strings
- nested dict values
- nested lists
- nested Pydantic models

## `_id` Handling

MongoModel includes special handling for MongoDB's `_id` field in two places:

- `from_mongo()` moves `_id` into `id`
- `to_mongo()` and `mongo()` move `id` back to `_id`

The initializer also checks for `_id` in the incoming constructor data and assigns it to `self.id`.

## Further Reading

For detailed information on Pydantic models and MongoDB ObjectId typing:

- [Pydantic v2 docs](https://docs.pydantic.dev/latest/)
- [MongoDB ObjectId with Pydantic v2 (StackOverflow)](https://stackoverflow.com/questions/76686267/what-is-the-new-way-to-declare-mongo-objectid-with-pydantic-v2-0)
- [FastAPI Pydantic integration](https://fastapi.tiangolo.com/tutorial/body/)
