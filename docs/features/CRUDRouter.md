The CRUDRouter is able to generate and document your routes based on the mongo model you pass to it.

```python
CRUDRouter(
    model=MyMongoModel,
    db=MyDbInstance,
    collection_name="my_collection_name",
    lookups=[MyFirstLookup, MySecondLookup],
    prefix="/my_prefix",
)
```

As the CRUDRouter herits from the APIRouter, you can pass all the parameters you would pass to the APIRouter, such as `prefix`, `tags`, `dependencies` etc.

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


The CRUDRouter will also generate the OpenAPI schema for you, and document the routes for you.

![CRUDRouter OpenAPI schema](../assets/img/crud-router.png)

You might want to do some relational stuff in your fastapi app, and you might want to use the CRUDRouter to generate your routes. Insane news, the CRUDRouter is able to generate routes for you based on your mongo model, and it is able to generate the OpenAPI schema for you.

You will find more information about the CRUDRouter in the following page.

[https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/features/CRUDLookup/](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/features/CRUDLookup/)

If you aren't looking for the "lookup" stuff, but for a way to use an embed model, you can find more information about the CRUDEmbed in the following page.

[https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/features/CRUDEmbed/](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/features/CRUDEmbed/)
