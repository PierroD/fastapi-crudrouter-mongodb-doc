# Service and Repository Example

Use this pattern when you want the library helpers but need custom endpoints instead of only generated CRUD routes.

```py linenums="1"
from fastapi import FastAPI
import motor.motor_asyncio
from fastapi_crudrouter_mongodb import (
    CRUDRepository,
    CRUDService,
    MongoModel,
    ObjectIdType,
)


client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017/local")
db = client.local


class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str
    role: str = "user"


service = CRUDService(
    model=UserModel,
    db=db,
    collection_name="users",
)

repository = CRUDRepository(
    model=UserModel,
    db=db,
    collection_name="users",
)


app = FastAPI(title="Service and Repository Example")


@app.get("/users")
async def list_users():
    return await service.find_all(limit=50, sort_by="name", order_by="ASC")


@app.get("/users/{id}")
async def get_user(id: str):
    return await service.find_one(id)


@app.get("/reports/admin-users")
async def list_admin_users():
    return await repository.find_all(filters={"role": "admin"}, sort_by="name", order_by="ASC")


@app.post("/seed")
async def seed_user():
    user = UserModel(name="Admin", email="admin@example.com", role="admin")
    return await repository.create_one(user)
```

## Why use this pattern

- `CRUDService` is useful when you want built-in HTTP error handling
- `CRUDRepository` is useful when you want lower-level data access inside custom endpoints
- You can mix these with normal FastAPI routes in the same app
