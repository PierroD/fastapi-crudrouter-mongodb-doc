# 4. Filtering, Sorting & Pagination

Query and organize your data efficiently.

## The GET All Endpoint

The `GET /resource` endpoint supports powerful query parameters.

## Pagination: Skip & Limit

Pagination splits large datasets into pages.

### Skip Parameter

Skip a number of documents:

```
GET /users?skip=0&limit=10
```

Returns documents 0-9 (first 10).

```
GET /users?skip=10&limit=10
```

Returns documents 10-19 (second 10).

### Limit Parameter

Limit results to N documents:

```
GET /users?limit=5
```

Returns at most 5 documents.

### Full Example

```
GET /users?skip=20&limit=10
```

Skip first 20, return next 10 (documents 20-29).

## Sorting: Sort By & Order

Order results by a field.

### Sort By

Specify which field to sort by:

```
GET /users?sort_by=name
```

Sorts by user name (ascending by default).

### Order Direction

Control ascending/descending:

```
GET /users?sort_by=name&sort_dir=1
```

`sort_dir=1` = ascending (A → Z)

```
GET /users?sort_by=name&sort_dir=-1
```

`sort_dir=-1` = descending (Z → A)

### Practical Examples

Get users alphabetically:
```
GET /users?sort_by=name&sort_dir=1
```

Get newest users first (if you have a `created_at` field):
```
GET /tracks?sort_by=plays&sort_dir=-1
```

Get the most played tracks first:
```
GET /tracks?sort_by=plays&sort_dir=-1&limit=10
```

## Filtering: Search & Query

Find specific documents using MongoDB filter syntax.

### Simple Filter

Filter by exact value:

```
GET /users?filters={"premium":true}
```

Returns only premium users.

```
GET /tracks?filters={"genre":"rock"}
```

Returns only rock tracks.

### Multiple Conditions

```
GET /users?filters={"premium":true,"email":"alice@example.com"}
```

Returns users that are premium AND have email "alice@example.com".

### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `$gt` | greater than | `{"plays": {"$gt": 1000}}` - more than 1000 plays |
| `$gte` | greater than or equal | `{"plays": {"$gte": 1000}}` - at least 1000 plays |
| `$lt` | less than | `{"plays": {"$lt": 100}}` - fewer than 100 plays |
| `$in` | in array | `{"status": {"$in": ["active", "pending"]}}` - status is active or pending |

### Filter Examples

Tracks with more than 1000 plays:
```
GET /tracks?filters={"plays":{"$gt":1000}}
```

Users created within last month (if using timestamps):
```
GET /users?filters={"created_at":{"$gte":"2024-04-01"}}
```

Active or premium users:
```
GET /users?filters={"$or":[{"status":"active"},{"premium":true}]}
```

## Combining Everything

Use skip + limit + sort + filter together:

```
GET /tracks?skip=0&limit=20&sort_by=plays&sort_dir=-1&filters={"genre":"rock"}
```

This returns:
1. Rock tracks only
2. Sorted by plays (most played first)
3. First 20 results

Perfect for a "Top 20 Rock Tracks" page!

## Real-World API Usage Examples

### Popular Playlists (Music App)

Get most-followed playlists:
```
GET /playlists?sort_by=followers&sort_dir=-1&limit=10
```

### User Search

Find users whose name starts with "al" (case-insensitive filter):
```
GET /users?filters={"name":{"$regex":"^al","$options":"i"}}
```

### Premium Subscribers Report

Get first 50 premium users by sign-up date:
```
GET /users?filters={"premium":true}&sort_by=created_at&sort_dir=-1&limit=50&skip=0
```

### Top 100 Artists by Plays

```
GET /artists?sort_by=total_plays&sort_dir=-1&limit=100
```

## Error Handling

### Invalid Filters

If filter JSON is malformed:

```
GET /users?filters={"invalid json}
```

**Response:** `422 Unprocessable Entity`
```json
{
  "detail": "Invalid JSON in filters parameter"
}
```

Solution: Make sure filters are valid JSON. Use URL encoding if needed:

```
GET /users?filters={"status":"active"}
```

## Best Practices

1. **Always paginate large datasets** - don't fetch millions at once
2. **Use reasonable limits** - 20-100 items per page is standard
3. **Test filters locally first** - MongoDB filter syntax is powerful but complex
4. **Cache popular queries** - pagination + filters can be slow for large datasets
5. **Combine thoughtfully** - too many filters can slow queries

## Recommended Defaults

For your API, consider defaults like:

```python
# Default pagination
limit=20  # 20 items per page
skip=0    # Start from first page

# Default sorting
sort_by="created_at"
sort_dir=-1  # Newest first
```

This gives users the most recent items by default.

## Next Steps

Continue learning:

- [**Related Data**](05-related-data.md) - Work with relationships (populates, lookups, embeds)
- [**Advanced Patterns**](06-advanced-patterns.md) - Custom features and optimization
- [**Back to Quickstart**](index.md)
