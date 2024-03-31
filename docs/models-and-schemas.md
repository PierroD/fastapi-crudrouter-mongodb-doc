

The CRUDRouter can generate and document your routes based on a `MongoModel` which is a custom pydantic model passed to it, as well as an output schema representing the JSON result you'll receive.

```py linenums="1"

CRUDRouter(
    model=MyMongoModel,
    model_out=MyOutputMongoModel, # which is a schema basically
    ...
)

```

!!! tip "Automatic Output Schema Generation"

    Omitting the output schema argument when initializing your CRUDRouter leads to the router automatically generating and documenting an output schema for your routes. This process involves automatically excluding any fields that match the primary key in the database since they will be generated server-side.

## In Model vs Out Schema

---


The **"in"** model represents the data stored in your database, while the **"out"** schema represents the data you'll receive in response to route calls. This allows you to explicitly exclude fields you wish to keep hidden in the response to your requests.

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