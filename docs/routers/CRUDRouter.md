The CRUDRouter is able to generate and document your routes based on the mongo model you pass to it.

```python
CRUDRouter(
    model=MyMongoModel,
    db=MyDbInstance,
    collection_name="my_collection_name",
    prefix="/my_prefix",
)
```

### Define an Output Schema

---

The **"in"** model represents the data stored in your database, while the **"out"** schema represents the data you'll receive in response to route calls. This allows you to explicitly exclude fields you wish to keep hidden in the response to your requests.
!!! question "How to use it ?"
    ```py linenums="1"
    from typing import Annotated
    from fastapi_crudrouter_mongodb import (
        ObjectId,
        MongoObjectId,
        MongoModel,
    )


    # In Model
    class UserModel(MongoModel):
        id: Annotated[ObjectId, MongoObjectId] | None = None
        name: str
        email: str
        password: str

    # Out Model -> Schema
    class UserModelOut(MongoModel):
        id: str
        name: str
        email: str

    # Here, we are excluding the user password from the returned data.
    ```



As the CRUDRouter herits from the APIRouter, you can pass all the parameters you would pass to the APIRouter, such as `prefix`, `tags`, `dependencies` etc.

## Generated Routes

---

The CRUDRouter will generate the following routes for you:

| Route             | Method   | Description              |
| ----------------- | -------- | ------------------------ |
| `/my_prefix`      | `GET`    | Get all documents        |
| `/my_prefix`      | `POST`   | Create a new document    |
| `/my_prefix/{id}` | `GET`    | Get a document by id     |
| `/my_prefix/{id}` | `PUT`    | Replace a document by id |
| `/my_prefix/{id}` | `PATCH`  | Update a document by id  |
| `/my_prefix/{id}` | `DELETE` | Delete a document by id  |

!!! note "If you want to respect the RESTfull"
    Make sure to add an `s` to the end of prefix, so that the route will be `/my_prefixes` instead of `/my_prefix`.

You have the possibility to add some custom dependencies to your routes, and you can also disable some router's routes, see all the parameters bellow :

## Disable routes & Dependencies

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

## OpenAPI Result

---

The CRUDRouter will also generate the OpenAPI schema for you, and document the routes for you.

![CRUDRouter OpenAPI schema](../assets/img/crud-router.png)

## Looking for something else ?

---

You might want to do some relational stuff in your fastapi app, and you might want to use the CRUDRouter to generate your routes. Insane news, the CRUDRouter is able to generate routes for you based on your mongo model, and it is able to generate the OpenAPI schema for you.

!!! tip "Lookups"

    You will find more information about the CRUDLookup in the following page.

    [CRUDLookup](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/features/CRUDLookup/)

!!! tip "Embeds"

    If you aren't looking for the "lookup" stuff, but for a way to use an embed model, you can find more information about the CRUDEmbed in the following page.

    [CRUDEmbed](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/features/CRUDEmbed/)
