# Relationships Example

This example shows the three relationship patterns supported by the library in one FastAPI app:

- `CRUDPopulate` for referenced documents in another collection
- `CRUDEmbed` for nested data inside the parent document
- `CRUDLookup` for nested child routes

```py linenums="1"
from typing import Optional, Union

from fastapi import FastAPI
import motor.motor_asyncio
from pydantic import Field
from fastapi_crudrouter_mongodb import (
    CRUDEmbed,
    CRUDLookup,
    CRUDPopulate,
    CRUDRouter,
    MongoModel,
    ObjectIdType,
)


client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017/local")
db = client.local


class ArtistModel(MongoModel):
    id: ObjectIdType | None = None
    name: str


class ArtistModelOut(MongoModel):
    id: str
    name: str


class GenreModel(MongoModel):
    id: ObjectIdType | None = None
    label: str


class ReviewModel(MongoModel):
    id: ObjectIdType | None = None
    track_id: ObjectIdType
    rating: int
    comment: str


class TrackModel(MongoModel):
    id: ObjectIdType | None = None
    title: str
    artist_ids: list[ObjectIdType] = Field(default_factory=list)
    genres: list[GenreModel] = Field(default_factory=list)
    reviews: Optional[Union[list[ReviewModel], ReviewModel]] = None


class TrackModelOut(MongoModel):
    id: str
    title: str
    artist_ids: list[ArtistModelOut] = Field(default_factory=list)
    genres: list[GenreModel] = Field(default_factory=list)


artist_populate = CRUDPopulate(
    field="artist_ids",
    collection="artists",
    model=ArtistModel,
)

genres_embed = CRUDEmbed(
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
    model_out=TrackModelOut,
    db=db,
    collection_name="tracks",
    prefix="/tracks",
    populates=[artist_populate],
    embeds=[genres_embed],
    lookups=[reviews_lookup],
    tags=["tracks"],
)

artists_router = CRUDRouter(
    model=ArtistModel,
    db=db,
    collection_name="artists",
    prefix="/artists",
    tags=["artists"],
)


app = FastAPI(title="Relationships Example")
app.include_router(artists_router)
app.include_router(tracks_router)
```

## What this gives you

- `GET /tracks` returns tracks and can populate `artist_ids`
- `POST /tracks/{id}/reviews` creates a review linked to a track
- `GET /tracks/{id}/reviews` lists reviews for a track
- Embedded `genres` are stored inside each track document

This is a good starting point for music, catalog, and content APIs.
