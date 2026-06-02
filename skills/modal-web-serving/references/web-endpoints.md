# Web Endpoints

Distilled from Modal guide on Web Functions.

## fastapi_endpoint

The simplest way to expose a Function over HTTP:

```python
@app.function(image=image)
@modal.fastapi_endpoint()
def f():
    return "Hello"
```

### HTTP Methods

Default is GET. Set with `method=`:

```python
@modal.fastapi_endpoint(method="POST")
def create(item: dict):
    return {"created": True}
```

### Query Parameters

```python
@modal.fastapi_endpoint()
def search(q: str, limit: int = 10):
    return {"query": q, "limit": limit}
# GET /search?q=hello&limit=5
```

### Pydantic Models

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    qty: int = 42

@modal.fastapi_endpoint(method="POST")
def create(item: Item):
    return {"name": item.name}
```

### Response Types

```python
from fastapi.responses import HTMLResponse, JSONResponse

@modal.fastapi_endpoint()
def page():
    return HTMLResponse("<h1>Hello</h1>")
```

## asgi_app

Full ASGI framework (FastAPI with middleware, multiple routes):

```python
@app.function(image=image)
@modal.asgi_app()
def web():
    from fastapi import FastAPI
    app = FastAPI()

    @app.get("/health")
    def health():
        return {"status": "ok"}

    @app.post("/predict")
    def predict(data: dict):
        return model.predict(data)

    return app
```

## wsgi_app

For Django or Flask:

```python
@app.function(image=image)
@modal.wsgi_app()
def django_app():
    from django.core.wsgi import get_wsgi_application
    return get_wsgi_application()
```

## web_server

For any HTTP server on a port:

```python
@app.function(image=image)
@modal.web_server(port=8080)
def serve():
    import subprocess
    subprocess.Popen(["uvicorn", "app:app", "--port", "8080"])
```

## Development Flow

1. `modal serve script.py` — creates ephemeral URLs with live-reload
2. Iterate on code → changes auto-deploy
3. `modal deploy script.py` — create persistent production URLs
