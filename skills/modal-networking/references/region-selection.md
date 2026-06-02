# Region Selection

Distilled from Modal guide on region selection.

## Container Region

Specify where container runs:

```python
@app.function(region=["us-west"])
def f(): ...
```

### Pricing

| Region Type | Multiplier |
|------------|-----------|
| Broad (e.g., `us`) | 1.5x |
| Narrow (e.g., `us-west`) | 1.75x |

Multiple regions in the list? The smaller multiplier applies.

### Available Regions

**Broad**: `us`, `eu`, `ap`, `uk`, `ca`, `me`, `sa`, `af`, `mx`

**Narrow** (selected):
- US: `us-east`, `us-central`, `us-south`, `us-west`
- EU: `eu-west`, `eu-north`, `eu-south`
- AP: `ap-northeast`, `ap-southeast`, `ap-south`, `ap-melbourne`, `jp`, `au`

Use broader regions for better availability and lower multipliers.

## Routing Region

Control where inputs are routed through (reduces network overhead):

```python
@app.function(routing_region="eu-west")
def f(): ...
```

Options: `us-east` (default), `us-west`, `eu-west`, `ap-south`.

### Restrictions

- `routing_region` set at initial deployment; cannot change on redeployment
- Only works with `.remote()`, `.map()`, and HTTP invocations
- Large inputs/outputs (>2 MiB) still route through `us-east`

## For Sandboxes

```python
sb = modal.Sandbox.create(app=app, region=["eu-west"])
```

## Optimizing Latency

Combine container region + routing region for lowest latency:

```python
@app.function(region=["eu-west"], routing_region="eu-west")
def eu_api():
    # Container in EU, traffic routed through EU
    ...
```

## Checking Container Location

```python
import os
location = os.environ.get("MODAL_REGION")
```

## Tips

- Use broader regions for better availability
- Set `routing_region` near your users for lower latency
- Combine container + routing region for best results
- Default (no region) routes through `us-east` with global container placement
