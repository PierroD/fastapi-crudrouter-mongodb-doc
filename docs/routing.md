
The main feature of the CRUDRouter is its automatic route generation. Below, we'll discuss how you can prefix, customize, or disable any of the generated routes.

## Default Routes

---

By default, the CRUDRouter will automatically generate the following six routes for you.

| Route   | Method   | Description              |
| --------| -------- | ------------------------ |
| `/`     | `GET`    | Get all documents        |
| `/`     | `POST`   | Create a new document    |
| `/{id}` | `GET`    | Get a document by id     |
| `/{id}` | `PUT`    | Replace a document by id |
| `/{id}` | `PATCH`  | Update a document by id  |
| `/{id}` | `DELETE` | Delete a document by id  |


!!! note "Route URLs"
    Note that the route url is prefixed by the defined prefix.

    **Example:** If the CRUDRouter's prefix is configured as "users" and I intend to update a particular user, the route I should access is `/users/my_user_id`, where *my_user_id* represents the **ID** of the user.

## Prefixes

---

The route prefix is required and can be configured within the CRUDRouter instance.

```py
CRUDRouter(
    model=MyMongoModel,
    db=MyDbInstance,
    collection_name="my_collection_name",
    prefix="/my_prefix", # <--- right here
)
```

## Disabling Routes

---

In certain situations, you may wish to deactivate a route that has been enabled by default in the CRUDRouter.

| Param Name               | Default Value | Type              | Description                 | Default Behavior                        |
|--------------------------|---------------|-------------------|-----------------------------|-----------------------------------------|
| disable_get_all          | False         | bool              | Disable get all route       | Get all route is enable / visible       |
| disable_get_one          | False         | bool              | Disable get by id route     | Get by id route is enable / visible     |
| disable_create_one       | False         | bool              | Disable create by id route  | Create by id route is enable / visible  |
| disable_replace_one      | False         | bool              | Disable replace by id route | Replace by id route is enable / visible |
| disable_update_one       | False         | bool              | Disable update by id route  | Update by id route is enable / visible  |
| disable_delete_one       | False         | bool              | Disable delete by id route  | Delete by id route is enable / visible  |


!!! note "Disable a route"
    To disable a route just add this parameter to your route to disable it

    ```py
    user = CRUDRouter(
        model=UserModel,
        db=db,
        collection_name="users",
        prefix="/users",
        tags=["users"],
        disable_get_all=True, # <--- Here the 'get_all' route is disable
    )  
    ``` 

## Add Custom Dependencies to a Route

---

Given the ability to disable certain routes, you may also consider adding custom dependencies to one of them.

| Param Name               | Default Value | Type              | Description                 | Default Behavior                        |
|--------------------------|---------------|-------------------|-----------------------------|-----------------------------------------|
| dependencies_get_all     | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_get_one     | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_create_one  | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_replace_one | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_update_one  | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |
| dependencies_delete_one  | None          | Sequence[Depends] | Add custom dependencies     | Default router dependencies             |

!!! note "Add custom dependency to a route"
    To disable a route just add this parameter to your route to disable it

    ```py
    user = CRUDRouter(
        model=UserModel,
        db=db,
        collection_name="users",
        prefix="/users",
        tags=["users"],
        dependencies_get_one=[Depends(verify_admin)], # <--- Here the 'get_one' route will use your dependency
    )  
    ``` 

## Overrind Routes

---

While overriding routes is a default capability in FastAPI, it hasn't been implemented yet in the CRUDRouter. Therefore, you may require some additional configuration to override a default route.

Routes in the CRUDRouter can be overridden by using the standard fastapi route decorators. These include:

- `@router.get(path: str, *args, **kwargs)`
- `@router.post(path: str, *args, **kwargs)`
- `@router.put(path: str, *args, **kwargs)`
- `@router.delete(path: str, *args, **kwargs)`
- `@router.api_route(path: str, methods: List[str] = ['GET'], *args, **kwargs)`

!!! tip "Tip"
    All routes within the CRUDRouter are subclasses of FastAPI's [APIRouter](https://fastapi.tiangolo.com/tutorial/bigger-applications/#apirouter), allowing for extensive customization to suit your needs.

### Overriding Example

Here's an example where we override the routes for /users when using the CRUDRouter.

```py
from typing import Annotated
from fastapi import FastAPI
from fastapi_crudrouter_mongodb import (
    ObjectId,
    MongoObjectId,
    MongoModel,
    CRUDRouter,
)
import motor.motor_asyncio

# Database connection using motor
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017/local")

# store the database in a global variable
db = client.local

# Database Model
class UserModel(MongoModel):
    id: Annotated[ObjectId, MongoObjectId] | None = None
    name: str
    email: str
    password: str


# Instantiating the CRUDRouter, and a lookup for the messages
# a User is a model that contains a list of embedded addresses and related to multiple messages

users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    disable_get_all=True, # <--- disabling the route is needed to override it
    tags=["users"],
)

@users_router.get('/') # you can add a '/' to make it iso to the previous one's
def overrides_get_all():
    return "My overrided route that returns all the users"

# Instantiating the FastAPI app
app = FastAPI()
app.include_router(users_router)
```