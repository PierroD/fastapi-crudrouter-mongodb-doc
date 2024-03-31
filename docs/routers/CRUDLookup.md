The CRUDLookup wasn't easy to implement, but it is now working like a charm :sparkles: (or kinda actually, I'm not really satisfied on how it's working).

You can't define your 'lookup' relation directly inside your model, and I'm really sorry about that, but it's the only way I found to make it work for the moment.

I'm still working on it, and I hope to find a better way to do it, but for now, it's working :sweat_smile:.

!!! question "How to use it ?"
    The parent model should have a field with an `Union` type, which means that it should look like below :

    ```py hl_lines="4"

    # Parent
    class ParentModel(MongoModel):
        id: Annotated[ObjectId, MongoObjectId] | None = None
        childs: Optional[Union[list[ChildModel], ChildModel]] = None

    # Child
    class ChildModel(MongoModel):
        id: Annotated[ObjectId, MongoObjectId] | None = None
        parent_id: Annotated[ObjectId, MongoObjectId]

    ```

    Then, you can define your lookup like this :

    ```py hl_lines="9 10 11 12 13 14 15"

    from fastapi_crudrouter_mongodb import (
        ObjectId,
        MongoObjectId,
        MongoModel,
        CRUDRouter,
        CRUDLookup)

    # Define your lookup
    child_lookup = CRUDLookup(
        model=ChildModel,
        collection_name="childrens",
        prefix="childrens",
        local_field="_id",
        foreign_field="parent_id"
    )

    # Then, you can use it in your CRUDRouter
    parent_controller = CRUDRouter(
        model=ParentModel,
        db=db,
        collection_name="parents",
        lookups=[child_lookup],
        prefix="/parents",
        tags=["parents"],
    )

    ```

The type should look like above because if you want to use the following route : `/parents/{id}/childrens/{id}` , it will return the child item as an object an not a list.

## Output Schemas

---

If you've perused the CRUDRouter documentation, you may have observed the option to utilize an output schema in response to your API requests. I've extended this capability to CRUDLookups as well in version 0.1.0. ✨

!!! question "How to use output schema ?"

    In this example, both the parent and the child have an output schema. As illustrated, the child utilizes the parent's output schema as a foundation.

    ```py hl_lines="29 30 31 32 33 34 35 36 43"
        # Database Model
        class MessageModel(MongoModel):
            id: Annotated[ObjectId, MongoObjectId] | None = None
            message: str
            user_id: Annotated[ObjectId, MongoObjectId]
            created_at: str | None = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            updated_at: str | None = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        class UserModel(MongoModel):
            id: Annotated[ObjectId, MongoObjectId] | None = None
            name: str
            email: str
            password: str
            addresses: Optional[list[AddressModel]] = []
            messages: Optional[Union[list[MessageModel], MessageModel]] = None

        # Model Out -> Schema
        class UserModelOut(MongoModel):
            id: str
            name: str
            email: str
            addresses: list[AddressModel] = []


        class LookupModelOut(UserModelOut):
            messages: list[MessageModel] | MessageModel


        messages_lookup = CRUDLookup(
        model=MessageModel,
        model_out=LookupModelOut,
        collection_name="messages",
        prefix="messages",
        local_field="_id",
        foreign_field="user_id",
        )

        users_router = CRUDRouter(
            model=UserModel,
            model_out=UserModelOut,
            db=db,
            collection_name="users",
            lookups=[messages_lookup],
            prefix="/users",
            tags=["users"],
        )

    ```


## Routes

---

The CRUDRouter will automatically build the following routes :

| Route                                 | Method   | Description                                  |
| ------------------------------------- | -------- | -------------------------------------------- |
| `/parents`                            | `GET`    | Get all documents                            |
| `/parents`                            | `POST`   | Create a new document                        |
| `/parents/{id}`                       | `GET`    | Get a document by id                         |
| `/parents/{id}`                       | `PUT`    | Update a document by id                      |
| `/parents/{id}`                       | `DELETE` | Delete a document by id                      |
| `/parents/{id}/childrens`             | `GET`    | Get all childrens of a parent                |
| `/parents/{id}/childrens/{lookup_id}` | `GET`    | Get a child of a parent by id                |
| `/parents/{id}/childrens`             | `POST`   | Create a new child document                  |
| `/parents/{id}/childrens/{lookup_id}` | `PUT`    | Replace a child document by id and lookup_id |
| `/parents/{id}/childrens/{lookup_id}` | `PATCH`  | Update a child document by id and lookup_id  |
| `/parents/{id}/childrens/{lookup_id}` | `DELETE` | Delete a document by id                      |

!!! note "If you want to respect the RESTfull"
Make sure to add an `s` to the end of prefix, so that the route will be `/childrens` instead of `/children` (which is not correct, I know, but you see the point here :wink:).

## OpenAPI Result

---

The CRUDRouter will also generate the OpenAPI schema for you, and document the routes for you.

![CRUDRouter OpenAPI schema](../assets/img/crud-router-lookup.png)

If you are looking for a way to make the CRUDRouter work with an `embed` model, don't worry I got your back :muscle:.

[https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/features/CRUDEmbed/](https://pierrod.github.io/fastapi-crudrouter-mongodb-doc/features/CRUDEmbed/)
