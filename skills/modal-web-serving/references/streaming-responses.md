# Streaming Responses

Distilled from Modal guide on streaming endpoints.

## Basic SSE Streaming

```python
import time

def event_stream():
    for i in range(10):
        yield f"data: event {i}\n\n".encode()
        time.sleep(0.5)

@app.function(image=modal.Image.debian_slim().pip_install("fastapi[standard]"))
@modal.fastapi_endpoint()
def stream():
    from fastapi.responses import StreamingResponse
    return StreamingResponse(event_stream(), media_type="text/event-stream")
```

The `text/event-stream` MIME type tells the web server to return responses immediately without buffering.

## GPU → Web Streaming with .remote_gen()

Chain a GPU Function's generator output to a web endpoint:

```python
@app.function(gpu="any")
def gpu_process():
    for chunk in model.generate():
        yield chunk.encode()

@app.function(image=image)
@modal.fastapi_endpoint()
def endpoint():
    from fastapi.responses import StreamingResponse
    return StreamingResponse(gpu_process.remote_gen(), media_type="text/event-stream")
```

## Streaming with .map()

Fan out to multiple containers and stream results back:

```python
@app.function()
def process_segment(i):
    return f"segment {i}\n"

@app.function(image=image)
@modal.fastapi_endpoint()
def parallel_stream():
    from fastapi.responses import StreamingResponse
    return StreamingResponse(process_segment.map(range(10)), media_type="text/plain")
```

Use `order_outputs=False` on `.map()` to return results as they complete (faster, unordered).

## Async Streaming

For async web apps, use `.map.aio()` to avoid blocking the event loop:

```python
@app.function(image=image)
@modal.fastapi_endpoint()
async def async_stream(request):
    from fastapi.responses import StreamingResponse

    async def wrapper():
        async for result in process.map.aio(segments):
            yield f"data: {result}\n\n"

    return StreamingResponse(wrapper(), media_type="text/event-stream")
```

## Tips

- Always use `media_type="text/event-stream"` for real-time SSE
- Other MIME types may be buffered by the web server
- Use `.remote_gen()` to stream from generator Functions
- Use `.map()` for parallel fan-out streaming
