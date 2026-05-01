# CRUDRouter

`CRUDRouter` generates CRUD routes for a MongoDB collection from a single model.

## Constructor

```python
CRUDRouter(
    model,
    db,
    collection_name,
    identifier_field="_id",
    model_out=None,
    lookups=None,
    embeds=None,
    populates=None,
    disable_get_all=False,
    disable_get_one=False,
    disable_create_one=False,
    disable_replace_one=False,
    disable_update_one=False,
    disable_delete_one=False,
    dependencies_get_all=None,
    dependencies_get_one=None,
    dependencies_create_one=None,
    dependencies_replace_one=None,
    dependencies_update_one=None,
    dependencies_delete_one=None,
    filter_dependency=None,
    *args,
    **kwargs,
)
```

## Query Parameters for `GET /`

When `disable_get_all=False`, list endpoints support:

- `skip` (`int`, default `0`)
- `limit` (`int`, default `100`)
- `sort_by` (`str | None`, default `None`)
- `sort_dir` (`int`, default `1`)
- `filters` (`str | None`, JSON string)

Example:

```http
GET /users?skip=0&limit=20&sort_by=name&sort_dir=-1&filters={"status":"active"}
```

Invalid JSON in `filters` returns:

```json
{
  "detail": "Invalid JSON in filters parameter"
}
```

with status `422`.

## `filter_dependency`

Use `filter_dependency` to supply a filter document from a FastAPI dependency:

```python

def active_filter():
    return {"status": "active"}


router = CRUDRouter(
    model=User,
    db=db,
    collection_name="users",
    prefix="/users",
    filter_dependency=active_filter,
)
```

In this mode, `filters` is not read from query string. The dependency output is used instead.

## `identifier_field`

`identifier_field` controls the route parameter used for item routes:

- default: `"_id"` -> route path uses `{id}`
- custom: `"email"` -> route path uses `{email}`

This also updates route summaries/descriptions in OpenAPI.

## `populates`

Use `populates` to resolve reference arrays on `GET /` and `GET /{id}`:

```python
from fastapi_crudrouter_mongodb import CRUDPopulate

router = CRUDRouter(
    model=Track,
    db=db,
    collection_name="tracks",
    prefix="/tracks",
    populates=[
        CRUDPopulate(field="artist_ids", collection="artists", model=Artist),
    ],
)
```

For full details and caveats, see `CRUDPopulate.md`.
