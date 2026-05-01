# 1. Installation & Setup

Get fastapi-crudrouter-mongodb installed and running your first CRUD API in minutes.

## Prerequisites

Before starting, make sure you have:

- Python 3.8+
- MongoDB running locally or remotely
- pip (Python package manager)

## Installation

Install the package via pip:

```console
pip install fastapi-crudrouter-mongodb
```

This installs:

- `fastapi-crudrouter-mongodb` - the main library
- `motor` - async MongoDB driver
- `pydantic` v2 - for data validation
- `fastapi` - web framework

## Verify Installation

Create a test file to verify everything works:

```python
# test_install.py
from fastapi_crudrouter_mongodb import MongoModel, CRUDRouter
print("✅ Installation successful!")
```

Run it:

```console
python test_install.py
```

You should see `✅ Installation successful!`

## Your First CRUD API

Let's build a simple API to manage users. Create a file called `main.py`:

```python
import motor.motor_asyncio
from fastapi import FastAPI
from fastapi_crudrouter_mongodb import CRUDRouter, MongoModel, ObjectIdType

# 1. Define your model
class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str


# 2. Connect to MongoDB
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.myapp


# 3. Create router
users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    tags=["Users"],
)


# 4. Create FastAPI app
app = FastAPI(title="My First CRUD API")
app.include_router(users_router)


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Run Your API

```console
python main.py
```

You'll see:

```
INFO:     Uvicorn running on http://127.0.0.1:8000
INFO:     Application startup complete
```

## Test Your API

### Interactive Docs

Open your browser to:

- **Swagger UI** (interactive): http://localhost:8000/docs
- **ReDoc** (read-only): http://localhost:8000/redoc

### Create a User

Click **POST /users** in Swagger UI and enter:

```json
{
  "name": "Alice",
  "email": "alice@example.com"
}
```

Click **Execute**. You'll get a response:

```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Alice",
  "email": "alice@example.com"
}
```

### List Users

Click **GET /users** and click **Execute**:

```json
[
  {
    "id": "507f1f77bcf86cd799439011",
    "name": "Alice",
    "email": "alice@example.com"
  }
]
```

### Get One User

Click **GET /users/{id}** and enter the id from above, click **Execute**.

### Update User

Click **PATCH /users/{id}** and change the email, click **Execute**.

### Delete User

Click **DELETE /users/{id}**, click **Execute**. You'll get a 204 No Content response.

## What You Just Built

✅ Full CRUD API with 6 endpoints:

- `GET /users` - list all users
- `POST /users` - create new user
- `GET /users/{id}` - get one user
- `PUT /users/{id}` - replace entire user
- `PATCH /users/{id}` - update user fields
- `DELETE /users/{id}` - delete user

✅ Automatic OpenAPI documentation

✅ Data validation via Pydantic

✅ MongoDB integration via Motor

✅ No manual routing or schema wiring needed

## Next Steps

Ready to learn more? Continue with:

- [**Models & Schemas**](02-models-and-schemas.md) - Hide sensitive data and secure your API
- [**CRUD Operations**](03-crud-operations.md) - Master Create, Read, Update, Delete with error handling
- [**Back to Quickstart**](index.md)
