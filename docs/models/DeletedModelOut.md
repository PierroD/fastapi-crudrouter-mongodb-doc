This model has been created to return the id of the deleted document in the database

```py
from .mongo_model import MongoModel

class DeletedModelOut(MongoModel):
    id: str
```

!!! tips "The result of a DELETE request"
    ```json
    {
        "id": "66094bbb5d477c1e95410be6"
    }
    ```