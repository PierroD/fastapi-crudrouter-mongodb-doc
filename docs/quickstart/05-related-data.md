# 5. Working with Related Data

Connect models together using populates, lookups, and embeds.

## The Problem: Related Data

When building real apps, data is related:

- Tracks belong to **Artists**
- Playlists contain **Tracks**
- Users have **Addresses**

Instead of just storing IDs, you usually want the full related data in responses.

## Three Approaches

| Approach | Use Case | Complexity |
|----------|----------|-----------|
| **Populate** | References to other collections | Easy |
| **Lookup** | Nested routes (like `/users/{id}/posts`) | Medium |
| **Embed** | Nested data inside document | Simple |

## 1. Populate: Resolve References

Use **CRUDPopulate** when you store ObjectIds and want to fetch full documents.

### Example: Tracks with Artists

Setup:

```python
from fastapi_crudrouter_mongodb import CRUDPopulate

# Models
class ArtistModel(MongoModel):
    id: ObjectIdType | None = None
    name: str


class TrackModel(MongoModel):
    id: ObjectIdType | None = None
    title: str
    artist_ids: list[ObjectIdType] = []  # ← References


# Output schemas
class ArtistModelOut(MongoModel):
    id: str
    name: str


class TrackModelOut(MongoModel):
    id: str
    title: str
    artist_ids: list[ArtistModelOut] = []  # ← Full artists!


# Define populate
artist_populate = CRUDPopulate(
    field="artist_ids",
    collection="artists",
    model=ArtistModel,
)

# Create router
tracks_router = CRUDRouter(
    model=TrackModel,
    model_out=TrackModelOut,
    db=db,
    collection_name="tracks",
    prefix="/tracks",
    populates=[artist_populate],
)
```

### Result

**Without populate**, response is:
```json
{
  "id": "507f",
  "title": "Bohemian Rhapsody",
  "artist_ids": ["507f1", "507f2"]  // Just IDs
}
```

**With populate**, response is:
```json
{
  "id": "507f",
  "title": "Bohemian Rhapsody",
  "artist_ids": [
    {"id": "507f1", "name": "Queen"},
    {"id": "507f2", "name": "Brian May"}
  ]  // Full artists!
}
```

### Usage

Get tracks with artists automatically resolved:

```
GET /tracks
```

All artist_ids are automatically fetched from the artists collection!

## 2. Lookup: Nested Routes

Use **CRUDLookup** for routes like `/users/{id}/posts`.

### Example: Users with Messages

Setup:

```python
from fastapi_crudrouter_mongodb import CRUDLookup, CRUDRouter

# Models
class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str
    messages: list["MessageModel"] | None = None  # ← Nested


class MessageModel(MongoModel):
    id: ObjectIdType | None = None
    text: str
    user_id: ObjectIdType  # ← Reference back to user


# Define lookup
messages_lookup = CRUDLookup(
    model=MessageModel,
    collection_name="messages",
    prefix="messages",
    local_field="_id",
    foreign_field="user_id",
)

# Create routers
users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    lookups=[messages_lookup],
)
```

### Result: New Routes

```
GET /users                    # All users
POST /users                   # Create user
GET /users/{id}               # One user
PUT /users/{id}               # Update user
DELETE /users/{id}            # Delete user

GET /users/{id}/messages      # Messages for user ← New!
POST /users/{id}/messages     # Create message for user ← New!
GET /users/{id}/messages/{msg_id}  # One message ← New!
PATCH /users/{id}/messages/{msg_id} # Update message ← New!
DELETE /users/{id}/messages/{msg_id} # Delete message ← New!
```

### Usage

Get all messages for user "507f":

```
GET /users/507f/messages
```

Response:
```json
[
  {"id": "msg1", "text": "Hello!", "user_id": "507f"},
  {"id": "msg2", "text": "How are you?", "user_id": "507f"}
]
```

Create a message for user:

```
POST /users/507f/messages
```

Request:
```json
{
  "text": "This is my first message"
}
```

## 3. Embed: Nested Documents

Use **CRUDEmbed** when child data lives inside the parent document.

### Example: Users with Addresses

Setup:

```python
from fastapi_crudrouter_mongodb import CRUDEmbed

# Models
class AddressModel(MongoModel):
    id: ObjectIdType | None = None
    street: str
    city: str
    country: str


class UserModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    email: str
    addresses: list[AddressModel] = []  # ← Embedded


# Define embed
addresses_embed = CRUDEmbed(
    model=AddressModel,
    embed_name="addresses",
)

# Create router
users_router = CRUDRouter(
    model=UserModel,
    db=db,
    collection_name="users",
    prefix="/users",
    embeds=[addresses_embed],
)
```

### MongoDB Storage

Instead of separate collections, addresses are stored inside each user:

```json
{
  "_id": "507f",
  "name": "Alice",
  "email": "alice@example.com",
  "addresses": [
    {
      "_id": "addr1",
      "street": "123 Main St",
      "city": "New York",
      "country": "USA"
    },
    {
      "_id": "addr2",
      "street": "456 Oak Ave",
      "city": "Boston",
      "country": "USA"
    }
  ]
}
```

### Result: New Routes

```
GET /users/{id}/embeds      # All embedded addresses
POST /users/{id}/embeds     # Add address to user
GET /users/{id}/embeds/{addr_id}  # Get one address
PATCH /users/{id}/embeds/{addr_id} # Update address
DELETE /users/{id}/embeds/{addr_id} # Remove address
```

### Usage

Get all addresses for user:

```
GET /users/507f/embeds
```

Add a new address to user:

```
POST /users/507f/embeds
```

Request:
```json
{
  "street": "789 Elm St",
  "city": "Chicago",
  "country": "USA"
}
```

## Choosing: Populate vs Lookup vs Embed

### Use Populate if:
- Data is in a separate collection
- Same reference appears in multiple places
- You want efficient batch queries

Example: Tracks referencing Artists

### Use Lookup if:
- You need nested REST routes
- Data relationship is hierarchical
- Separate collections make sense

Example: Users with Posts (separate collections)

### Use Embed if:
- Child data only belongs to one parent
- No need for separate queries
- Data is small

Example: Users with Addresses

## Complete Example: Music App

```python
# Artists (separate collection)
class ArtistModel(MongoModel):
    id: ObjectIdType | None = None
    name: str


# Genres (embedded)
class GenreModel(MongoModel):
    id: ObjectIdType | None = None
    name: str
    description: str


# Tracks (separate collection)
class TrackModel(MongoModel):
    id: ObjectIdType | None = None
    title: str
    artist_ids: list[ObjectIdType] = []  # ← Populate
    genres: list[GenreModel] = []        # ← Embed


# Reviews (separate collection, linked to tracks)
class ReviewModel(MongoModel):
    id: ObjectIdType | None = None
    text: str
    track_id: ObjectIdType


# Routers
artist_populate = CRUDPopulate(
    field="artist_ids",
    collection="artists",
    model=ArtistModel,
)

genre_embed = CRUDEmbed(
    model=GenreModel,
    embed_name="genres",
)

reviews_lookup = CRUDLookup(
    model=ReviewModel,
    collection_name="reviews",
    prefix="reviews",
    local_field="_id",
    foreign_field="track_id",
)

tracks_router = CRUDRouter(
    model=TrackModel,
    db=db,
    collection_name="tracks",
    prefix="/tracks",
    populates=[artist_populate],
    embeds=[genre_embed],
    lookups=[reviews_lookup],
)

app = FastAPI()
app.include_router(tracks_router)
```

### Result

```
GET /tracks/507f
```

Returns:
```json
{
  "id": "507f",
  "title": "Bohemian Rhapsody",
  "artist_ids": [{"id": "art1", "name": "Queen"}],  // Populated
  "genres": [{"id": "gen1", "name": "Rock"}],       // Embedded
  // Also has reviews routes:
  // GET /tracks/507f/reviews
  // POST /tracks/507f/reviews
  // etc.
}
```

## Best Practices

1. **Populate** - for references across collections
2. **Embed** - for small nested data
3. **Lookup** - for hierarchical relationships

4. **Keep embedded documents small** - MongoDB has a 16MB document limit
5. **Use populates for large reference arrays** - automatic batch optimization
6. **Test all three before deciding** - see which feels right for your data

## Next Steps

Continue learning:

- [**Advanced Patterns**](06-advanced-patterns.md) - Custom features and optimization
- [**Production Ready**](07-production-ready.md) - Deployment and best practices
- [**Back to Quickstart**](index.md)
