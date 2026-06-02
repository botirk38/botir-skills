# Web Framework Integration

Distilled from Modal guide on Web Functions.

## FastAPI (Recommended)

### Simple endpoint

```python
image = modal.Image.debian_slim().pip_install("fastapi[standard]")

@app.function(image=image)
@modal.fastapi_endpoint()
def hello(name: str = "world"):
    return {"message": f"Hello {name}"}
```

### Full FastAPI app

```python
@app.function(image=image)
@modal.asgi_app()
def web():
    from fastapi import FastAPI
    app = FastAPI()

    @app.get("/")
    def root():
        return {"status": "ok"}

    @app.get("/items/{item_id}")
    def get_item(item_id: int):
        return {"item_id": item_id}

    return app
```

### FastAPI with lifespan

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app):
    # startup
    yield
    # shutdown

@app.function(image=image)
@modal.asgi_app()
def web():
    from fastapi import FastAPI
    return FastAPI(lifespan=lifespan)
```

## Flask

```python
image = modal.Image.debian_slim().pip_install("flask")

@app.function(image=image)
@modal.wsgi_app()
def flask_app():
    from flask import Flask
    app = Flask(__name__)

    @app.route("/")
    def hello():
        return "Hello from Flask!"

    return app
```

## Django

```python
image = modal.Image.debian_slim().pip_install("django")

@app.function(image=image)
@modal.wsgi_app()
def django_app():
    import django
    from django.conf import settings
    settings.configure(...)
    django.setup()
    from django.core.wsgi import get_wsgi_application
    return get_wsgi_application()
```

## Raw HTTP Server

For any server that listens on a port:

```python
@app.function(image=image)
@modal.web_server(port=8080)
def server():
    import subprocess
    subprocess.Popen(["python", "-m", "http.server", "8080"])
```

## Tips

- `fastapi_endpoint` is simplest for single-endpoint APIs
- `asgi_app` for multi-route FastAPI/Starlette applications
- `wsgi_app` for Django/Flask
- `web_server` as escape hatch for any HTTP server
- All options autoscale and scale to zero
