# Classes and Methods

Distilled from Modal guide on class-based Functions.

## Converting Functions to Classes

Before:

```python
@app.function(gpu="A100")
def predict(prompt):
    model = load_model()  # loaded every call
    return model(prompt)
```

After:

```python
@app.cls(gpu="A100")
class Model:
    @modal.enter()
    def setup(self):
        self.model = load_model()  # loaded once

    @modal.method()
    def predict(self, prompt):
        return self.model(prompt)
```

## @app.cls Decorator

Takes the same arguments as `@app.function()`:

```python
@app.cls(
    gpu="A100",
    image=my_image,
    secrets=[modal.Secret.from_name("api-keys")],
    volumes={"/data": volume},
    max_containers=10,
    scaledown_window=300,
)
class MyService:
    ...
```

## @modal.method Decorator

Exposes a method as a remotely invokable Function:

```python
@modal.method()
def predict(self, x):
    return self.model(x)
```

### Multiple methods

A class can have multiple methods, each invokable independently:

```python
@app.cls(gpu="A100")
class Model:
    @modal.enter()
    def setup(self):
        self.model = load_model()

    @modal.method()
    def predict(self, prompt):
        return self.model(prompt)

    @modal.method()
    def embed(self, text):
        return self.model.embed(text)
```

## Invocation

```python
model = Model()
result = model.predict.remote("hello")
embedding = model.embed.remote("world")

# Parallel
results = model.predict.map(["a", "b", "c"])
```

## Web Endpoints on Classes

```python
@app.cls(image=image)
class WebService:
    @modal.enter()
    def setup(self):
        self.db = connect_db()

    @modal.fastapi_endpoint()
    def api(self, request: dict):
        return self.db.query(request)
```

## from_name (Cross-App Reference)

```python
Model = modal.Cls.from_name("deployed-app", "Model")
result = Model().predict.remote("hello")
```

## Tips

- Use `@app.cls` instead of `@app.function` for stateful workloads
- `@modal.method()` replaces `@app.function()` inside classes
- Each class instance with the same parameters shares a container pool
- Web endpoint decorators work on class methods too
