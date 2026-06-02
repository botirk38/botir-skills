---
name: modal-web-serving
description: Expose Modal Functions as HTTP endpoints using FastAPI, ASGI, WSGI, or raw HTTP. Use when building APIs, web apps, streaming endpoints, or serving ML models over HTTP with custom domains and authentication.
---

# Modal Web Serving

Use this skill when exposing Modal Functions as web endpoints, building APIs, or serving web applications.

## When to Use This Skill

- Creating REST/HTTP APIs from Modal Functions
- Serving ML model inference over HTTP
- Building web apps with FastAPI, Django, or Flask on Modal
- Setting up streaming (SSE) endpoints
- Configuring custom domains
- Adding authentication to web endpoints

## Quick Start

```python
image = modal.Image.debian_slim().pip_install("fastapi[standard]")

@app.function(image=image)
@modal.fastapi_endpoint()
def hello():
    return "Hello world!"
```

Deploy with `modal serve` (dev) or `modal deploy` (production).

## Four Ways to Serve HTTP

### 1. fastapi_endpoint (simplest)

Wraps a function in FastAPI automatically:

```python
@app.function(image=image)
@modal.fastapi_endpoint()
def square(x: int):
    return {"square": x**2}
```

Accepts query params, POST bodies (dict or Pydantic models), path params.

### 2. asgi_app (full framework)

Serve a complete ASGI app (FastAPI, Starlette):

```python
@app.function(image=image)
@modal.asgi_app()
def fastapi_app():
    from fastapi import FastAPI
    web_app = FastAPI()

    @web_app.get("/")
    def root():
        return {"message": "hello"}

    return web_app
```

### 3. wsgi_app (Django, Flask)

```python
@app.function(image=image)
@modal.wsgi_app()
def flask_app():
    from flask import Flask
    app = Flask(__name__)

    @app.route("/")
    def root():
        return "hello"

    return app
```

### 4. web_server (raw HTTP)

For any HTTP server listening on a port:

```python
@app.function(image=image)
@modal.web_server(port=8000)
def serve():
    import subprocess
    subprocess.Popen(["python", "-m", "http.server", "8000"])
```

## Streaming Responses

Use FastAPI's `StreamingResponse` for SSE or chunked responses:

```python
@app.function(image=image)
@modal.fastapi_endpoint()
def stream():
    from fastapi.responses import StreamingResponse

    def generate():
        for i in range(10):
            yield f"data: chunk {i}\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

Combine with `.remote_gen()` for GPU→web streaming:

```python
@app.function(gpu="any")
def gpu_work():
    for chunk in process():
        yield chunk

@app.function(image=image)
@modal.fastapi_endpoint()
def endpoint():
    from fastapi.responses import StreamingResponse
    return StreamingResponse(gpu_work.remote_gen(), media_type="text/event-stream")
```

## Symptom Triage

### "Endpoint returns 500"
- Check container logs on the Modal dashboard
- Ensure FastAPI is installed in the image
- Verify Pydantic models match request schema

### "Slow first request"
- Cold start; use `min_containers=1` for warm containers
- Move model loading to `@modal.enter()` lifecycle hook

### "Streaming not real-time"
- Use `media_type="text/event-stream"` for SSE
- Other content types may be buffered by the web server

## Reference Map

- `references/web-endpoints.md` — endpoint decorators, methods, POST/GET, Pydantic
- `references/streaming-responses.md` — SSE, StreamingResponse, async streaming
- `references/custom-domains-auth.md` — custom domains, authentication
- `references/frameworks.md` — FastAPI, Django, Flask integration patterns

## Guardrails

- `@modal.fastapi_endpoint` requires FastAPI installed in the image
- Web Functions autoscale and scale to zero like regular Functions
- Use `modal serve` for development (live-reload), `modal deploy` for production
- `@modal.web_endpoint` is the old name; use `@modal.fastapi_endpoint` (v0.73.82+)
