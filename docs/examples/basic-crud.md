# Basic CRUD Example

This is the smallest practical example in the documentation.

Use it when you want to generate a complete CRUD API from a single model with no relationships.

```py linenums="1"
from fastapi import FastAPI
import motor.motor_asyncio
from fastapi_crudrouter_mongodb import CRUDRouter, MongoModel, ObjectIdType


client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017/local")
db = client.local


class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str


users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    tags=["users"],
)


app = FastAPI(title="Basic CRUD Example")
app.include_router(users_router)
```

## Generated routes

This example gives you:

- `GET /users`
- `POST /users`
- `GET /users/{id}`
- `PUT /users/{id}`
- `PATCH /users/{id}`
- `DELETE /users/{id}`

## Run it

```bash
uvicorn main:app --reload
```

Then open `http://localhost:8000/docs`.
