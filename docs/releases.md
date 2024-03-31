# v0.1.0 - Migrate to Pydantic v2 & Refacto 

**2024-03-31**

### ⚠️ Breaking changes 
- A lot of the typing has changed due to Pydantic v2
- The Embed needs to be written explicitly


### Feat
- Migrate to Pydantic v2
- Add output schema to the CRUDLookup
- New CRUDEmbed router 

### Fix 
- DELETE Routes will returned the same Schema
- Remove lookups fields from the parent Schema automatically

### Doc
- New documentation pages
- Migration guides available here [Migration Guide]()

--- 

# v0.0.8 - Add output schema to the CRUDRouter 
**2023-09-13**

### Feat 
- Add `model_out` to the CRUDRouter, gives you the opportunity not to reveal sensitive data
- Add black to format the code
- Add ruff to lint the code

### Fix 

- fix the `convert_to` method so it will convert the `id` to str to prevent issue while converting MongoObjectId to str

#### Example

---

```py
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

#### Result

---

![image](https://github.com/PierroD/fastapi-crudrouter-mongodb/assets/40763427/b30f73dc-b545-4e67-93cc-33338902b170)

--- 

# v0.0.7 - Fix Pydantic to v1.x 
**2023-08-13**

### Fix Pydantic crashes and add methods

- Fix : force pydantic v 1.X dependency to prevent the lib from crashes
- Refactor : refacto `mongo` and rename it to `to_mongo`
- Feat : add a new method called `convert_to` (it helps you to convert a model to another)

#### Example
 
---

> Using the convert_to option 
```py
@app.get("/{id}", response_model=UserModelDAO)
async def root(id: str):
    response = await db['users'].find_one({'_id': MongoObjectId(id)})
    userDAO = UserModelDAO.from_mongo(response)
    userDTO = userDAO.convert_to(UserModelDTO)
    return userDTO
```

--- 

# v0.0.5 - Remove routes and add dependencies
**2023-05-10**


### Remove route and add dependencies to a specific route :rocket: 

- Feat : allowed to disable any route of the CRUDRouter
- Feat : allowed to add custom dependencies to a specific route 

#### Example 

---

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

#### New CRUDRouter params

---

| Param Name               | Default Value | Type              | Description                 | Default Behavior                        |
|--------------------------|---------------|-------------------|-----------------------------|-----------------------------------------|
| disable_get_all          | False         | bool              | Disable get all route       | Get all route is enable / visible       |
| disable_get_one          | False         | bool              | Disable get by id route     | Get by id route is enable / visible     |
| disable_create_one       | False         | bool              | Disable create by id route  | Create by id route is enable / visible  |
| disable_replace_one      | False         | bool              | Disable replace by id route | Replace by id route is enable / visible |
| disable_update_one       | False         | bool              | Disable update by id route  | Update by id route is enable / visible  |
| disable_delete_one       | False         | bool              | Disable delete by id route  | Delete by id route is enable / visible  |
| dependencies_get_all     | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_get_one     | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_create_one  | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_replace_one | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_update_one  | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_delete_one  | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |

--- 

# v0.0.4 - Initial Release
**2023-04-10**

![](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/assets/img/logo-long-color.png)

<h1 align="center">
 :partying_face:  Initial Release :partying_face:  
</h1>

Tired of rewriting the same generic CRUD routes? Need to rapidly prototype a feature for a presentation or a hackathon? Thankfully, fastapi-crudrouter-mongodb has your back. As an extension to the APIRouter included with FastAPI, the FastAPI CRUDRouter will automatically generate and document your CRUD routes for you.


**Documentation**: [https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/)

**Source Code**: [https://github.com/pierrod/fastapi-crudrouter-mongodb](https://github.com/pierrod/fastapi-crudrouter-mongodb)

**Credits** :

- Base projet and idea : [awtkns](https://github.com/awtkns/fastapi-crudrouter)

- Convert \_id to id : [mclate github guide](https://github.com/tiangolo/fastapi/issues/1515)


## Installation

```bash
pip install fastapi-crudrouter-mongodb
```

## Basic Usage

I will provide more examples in the future, but for now, here is a basic example of how to use the FastAPI CRUDRouter for Mongodb.

```python
from datetime import datetime
from typing import List, Optional, Union
from fastapi import FastAPI
from pydantic import Field
from fastapi_crudrouter_mongodb import CRUDRouter, MongoModel, MongoObjectId, CRUDLookup
import motor.motor_asyncio


# Database connection using motor
client = motor.motor_asyncio.AsyncIOMotorClient(
    "mongodb://localhost:27017/local")

db = client.local

# Models
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


# Instantiating the CRUDRouter, and a lookup for the messages
# a User is a model that contains a list of embedded addresses and related to multiple messages
messages_lookup = CRUDLookup(
        model=MessageModel,
        collection_name="messages",
        prefix="messages",
        local_field="_id",
        foreign_field="user_id"
    )

users_controller = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    lookups=[messages_lookup],
    prefix="/users",
    tags=["users"],
)

# Instantiating the FastAPI app
app = FastAPI()
app.include_router(users_controller)
```

## OpenAPI Support

By default, all routes generated by the CRUDRouter will be documented according to OpenAPI spec.

Below are the default routes created by the CRUDRouter shown in the generated OpenAPI documentation.

![CRUDRouter full OpenAPI image](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/assets/img/crud-router-full.png)
