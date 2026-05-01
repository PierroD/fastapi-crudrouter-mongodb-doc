# 7. Production Ready

Deploy your API confidently with testing, security, and performance best practices.

## Environment Variables

Never hardcode secrets. Use environment variables:

```python
import os
from dotenv import load_dotenv

load_dotenv()

MONGO_URL = os.getenv("MONGO_URL", "mongodb://localhost:27017")
JWT_SECRET = os.getenv("JWT_SECRET", "dev-secret")
API_KEY = os.getenv("API_KEY", "dev-key")
DEBUG = os.getenv("DEBUG", "false").lower() == "true"

client = motor.motor_asyncio.AsyncIOMotorClient(MONGO_URL)
```

Create `.env`:

```
MONGO_URL=mongodb+srv://username:password@cluster.mongodb.net/music_app
JWT_SECRET=your-secret-key-here
API_KEY=api-key-here
DEBUG=false
```

**Important:** Add `.env` to `.gitignore`!

```
.env
.env.local
```

## Testing

### Unit Tests

Test your models:

```python
import pytest
from my_app import UserModel, UserModelOut

@pytest.mark.asyncio
async def test_user_model():
    user = UserModel(name="Alice", email="alice@example.com", password="secret")
    assert user.name == "Alice"
    assert user.id is None  # Not saved yet


@pytest.mark.asyncio
async def test_user_model_out():
    user_out = UserModelOut(id="507f", name="Alice", email="alice@example.com")
    assert "password" not in user_out.__dict__  # Sensitive field not included
```

### Integration Tests

Test actual API endpoints:

```python
import pytest
from fastapi.testclient import TestClient
from my_app import app

client = TestClient(app)


@pytest.mark.asyncio
async def test_create_user():
    response = client.post(
        "/users",
        json={"name": "Alice", "email": "alice@example.com", "password": "secret"}
    )
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Alice"
    assert "password" not in data  # Output schema hides it


@pytest.mark.asyncio
async def test_get_user():
    response = client.get("/users/507f1f77bcf86cd799439011")
    assert response.status_code in [200, 400]  # Either found or not found


@pytest.mark.asyncio
async def test_delete_user():
    response = client.delete("/users/507f1f77bcf86cd799439011")
    assert response.status_code == 204
```

### Run Tests

```bash
pip install pytest pytest-asyncio
pytest tests/
```

## Security Best Practices

### 1. Never Expose Sensitive Fields

Always use output schemas:

```python
# ❌ BAD: Exposes password
class UserBadOut(MongoModel):
    id: str
    name: str
    email: str
    password: str  # ← Exposed!


# ✅ GOOD: Hides password
class UserGoodOut(MongoModel):
    id: str
    name: str
    email: str
    # password not included
```

### 2. Hash Passwords

Store hashed passwords, never plain text:

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


@app.post("/register")
async def register(user: UserModel):
    user.password = pwd_context.hash(user.password)  # Hash before saving
    return await service.create_one(user)


@app.post("/login")
async def login(email: str, password: str):
    user = await service.find_one({"email": email})
    if not pwd_context.verify(password, user.password):  # Verify hash
        raise HTTPException(status_code=400, detail="Invalid credentials")
    return {"access_token": create_token(user.id)}
```

### 3. Use HTTPS

Always use HTTPS in production:

```python
# Redirect HTTP to HTTPS
@app.middleware("http")
async def redirect_https(request, call_next):
    if request.url.scheme == "http" and not DEBUG:
        url = request.url.replace(scheme="https")
        return RedirectResponse(url=url, status_code=301)
    return await call_next(request)
```

Or let your hosting provider handle it (recommended).

### 4. Implement Rate Limiting

Prevent abuse:

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter


@app.get("/users")
@limiter.limit("100/minute")
async def get_users(request: Request):
    return await service.find_all()
```

Allows max 100 requests per minute per IP.

### 5. Add CORS for Frontend

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com", "https://app.example.com"],  # Specific domains
    allow_credentials=True,
    allow_methods=["GET", "POST", "PATCH", "DELETE"],
    allow_headers=["*"],
)
```

### 6. Validate All Inputs

Use Pydantic models (already done!):

```python
class UserModel(MongoModel):
    name: str  # Required
    email: str  # Type-checked
    password: str  # Must be string


# FastAPI automatically rejects invalid data:
# POST /users with missing name → 422 error
# POST /users with email="not-an-email" → 422 error
```

### 7. Add API Keys

For programmatic access:

```python
from fastapi import Header, HTTPException

async def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key != os.getenv("API_KEY"):
        raise HTTPException(status_code=403, detail="Invalid API key")
    return x_api_key


@app.get("/admin/stats")
async def get_stats(api_key: str = Depends(verify_api_key)):
    return {"users": await db["users"].count_documents({})}
```

Client must include header:

```bash
curl -H "X-API-Key: your-api-key" https://api.example.com/admin/stats
```

## Deployment

### Docker

Create `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - MONGO_URL=mongodb://mongo:27017
      - DEBUG=false
    depends_on:
      - mongo

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

Run:

```bash
docker-compose up
```

### Cloud Deployment

Popular options:

- **Railway** (simple, MongoDB included)
- **Render** (free tier available)
- **Fly.io** (fast, affordable)
- **AWS** (most control, most complex)

All support FastAPI + MongoDB.

## Monitoring

### Health Checks

Add a health endpoint:

```python
@app.get("/health")
async def health_check():
    try:
        await db.client.server_info()  # Check MongoDB connection
        return {"status": "healthy", "db": "connected"}
    except Exception as e:
        return {"status": "unhealthy", "error": str(e)}, 500
```

Monitoring services check this regularly:

```bash
curl https://api.example.com/health
```

### Logging

Use Python logging:

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


@app.post("/users")
async def create_user(user: UserModel):
    logger.info(f"Creating user: {user.email}")
    created = await service.create_one(user)
    logger.info(f"User created: {created.id}")
    return created
```

### Metrics

Export metrics for monitoring tools (Prometheus, DataDog, etc.):

```python
from prometheus_client import Counter, Histogram
import time

request_count = Counter("http_requests_total", "Total requests")
request_duration = Histogram("http_request_duration_seconds", "Request duration")


@app.middleware("http")
async def track_metrics(request, call_next):
    request_count.inc()
    start = time.time()
    response = await call_next(request)
    duration = time.time() - start
    request_duration.observe(duration)
    return response
```

## Performance Checklist

- ✅ Add MongoDB indexes for frequently queried fields
- ✅ Use pagination for large datasets
- ✅ Cache popular queries/responses
- ✅ Enable GZIP compression
- ✅ Use connection pooling for database
- ✅ Monitor response times
- ✅ Use CDN for static content
- ✅ Optimize database queries

## Security Checklist

- ✅ Use output schemas (never expose sensitive data)
- ✅ Hash passwords
- ✅ Use HTTPS
- ✅ Validate all inputs
- ✅ Add rate limiting
- ✅ Add API authentication
- ✅ Use environment variables for secrets
- ✅ Add CORS restrictions
- ✅ Add request logging
- ✅ Keep dependencies updated

## Maintenance

### Update Dependencies

Regularly check for updates:

```bash
pip list --outdated
pip install --upgrade fastapi motor pydantic
```

### Database Backups

For MongoDB Atlas (cloud):
- Atlas handles automatic backups
- Download backups from dashboard

For self-hosted MongoDB:

```bash
# Backup
mongodump --uri "mongodb://localhost:27017" --out backup/

# Restore
mongorestore --uri "mongodb://localhost:27017" backup/
```

### Monitoring Errors

Use error tracking (Sentry, Rollbar):

```python
import sentry_sdk

sentry_sdk.init(
    dsn="https://key@sentry.io/project",
    traces_sample_rate=1.0,
)
```

Now all errors are tracked automatically.

## Conclusion

You now know how to:

✅ Protect sensitive data  
✅ Secure your API  
✅ Test thoroughly  
✅ Deploy with confidence  
✅ Monitor in production  

Your API is production-ready!

## Next Steps

Additional learning:

- Read FastAPI [Security docs](https://fastapi.tiangolo.com/tutorial/security/)
- Read MongoDB [Best Practices](https://docs.mongodb.com/manual/administration/)
- Explore [Motor async docs](https://motor.readthedocs.io/)

- [**Back to Quickstart**](index.md)
