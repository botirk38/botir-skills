# Custom Domains and Authentication

Distilled from Modal guides on custom domains and authentication.

## Custom Domains

Attach your own domain to a Modal web endpoint:

1. Go to Modal dashboard → Custom Domains
2. Add your domain
3. Set DNS records (CNAME) as instructed
4. Reference in code:

```python
@app.function(image=image)
@modal.fastapi_endpoint(custom_domains=["api.example.com"])
def my_api():
    return {"status": "ok"}
```

Multiple custom domains can point to the same endpoint.

## Authentication

### Built-in Authentication

Modal provides token-based authentication for web endpoints:

```python
@app.function(image=image)
@modal.fastapi_endpoint(requires_auth=True)
def protected():
    return {"data": "secret"}
```

Callers must include a Modal token in the Authorization header.

### Custom Authentication

Implement your own auth using FastAPI dependencies:

```python
from fastapi import Depends, HTTPException, Header

async def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key != os.environ["API_KEY"]:
        raise HTTPException(status_code=403, detail="Invalid API key")

@app.function(
    image=image,
    secrets=[modal.Secret.from_name("api-keys")]
)
@modal.fastapi_endpoint()
def protected(user=Depends(verify_api_key)):
    return {"data": "secret"}
```

### Bearer Token Pattern

```python
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

@app.function(image=image, secrets=[modal.Secret.from_name("auth")])
@modal.fastapi_endpoint()
def api(credentials: HTTPAuthorizationCredentials = Depends(security)):
    if credentials.credentials != os.environ["TOKEN"]:
        raise HTTPException(status_code=401)
    return {"authenticated": True}
```

## CORS

Configure CORS for browser-based clients using FastAPI middleware:

```python
@modal.asgi_app()
def web():
    from fastapi import FastAPI
    from fastapi.middleware.cors import CORSMiddleware

    app = FastAPI()
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["https://mysite.com"],
        allow_methods=["*"],
        allow_headers=["*"],
    )
    return app
```
