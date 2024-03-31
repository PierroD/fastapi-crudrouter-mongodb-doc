
## Starting from version 0.1.0, Pydantic v2 is now included.

Here's a guide you can follow to migrate your FastAPI-Crudrouter-Mongodb project to ensure compatibility with the latest version of Pydantic.

## Imports

New imports are now available and may be necessary; these are the most important ones.

```py hl_lines="3 11"
import motor.motor_asyncio

from typing import Optional, Union, Annotated
from fastapi import FastAPI
from fastapi_crudrouter_mongodb import (
    ObjectId,
    MongoObjectId,
    MongoModel,
    CRUDRouter,
    CRUDLookup,
    CRUDEmbed,
)


```

## Typing 

A lot of type have changed with this update so let's start with it 

- The ID's :

```py hl_lines="7"
# OLD Version
class OLD_Model(MongoModel):
    id: Optional[MongoObjectId] = Field()

# NEW Version
class NEW_Model(MongoModel):
    id: Annotated[ObjectId, MongoObjectId] | None = None

```

- The Embeds :

This version change the way Embed are working

```py hl_lines="20 22 28"
# OLD Version
class OLD_Embed_Model(MongoModel):
    id: Optional[MongoObjectId] = Field()
    field: str

class OLD_Main_Model(MongoModel):
    id: Optional[MongoObjectId] = Field()
    field: str
    embeds: Optional[List[OLD_Embed_Model]]


# NEW Version
class NEW_Embed_Model(MongoModel):
    id: Annotated[ObjectId, MongoObjectId] | None = None
    field: str

class NEW_Main_Model(MongoModel):
    id: Annotated[ObjectId, MongoObjectId] | None = None
    field: str
    embeds: Optional[list[NEW_Embed_Model]] = []

new_embed_router = CRUDEmbed(model=NEW_Embed_Model, embed_name="embeds")

new_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    embeds=[new_embed_router],
    prefix="/users",
    tags=["users"],
)
```