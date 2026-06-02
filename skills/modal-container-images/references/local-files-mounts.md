# Local Files and Mounts

Distilled from Modal guide on Images.

## add_local_dir

Copy a local directory into the container:

```python
image = modal.Image.debian_slim().add_local_dir(
    "/Users/me/.aws",
    remote_path="/root/.aws"
)
```

By default, files are added at container startup (not baked into the image). This means fast redeploys but you cannot run build steps after.

Force baking into image layer:

```python
image.add_local_dir("./config", remote_path="/config", copy=True)
```

## add_local_file

Copy a single file:

```python
image.add_local_file("model.bin", remote_path="/root/model.bin")
```

Same `copy=True` behavior as `add_local_dir`.

## add_local_python_source

Add importable Python modules by name:

```python
image = modal.Image.debian_slim().add_local_python_source("my_module")

@app.function(image=image)
def f():
    import my_module
    my_module.do_stuff()
```

Difference from `add_local_dir`:
- Takes module names, not file paths
- Looks up the module path automatically
- Handles packages with `__init__.py` correctly

## Default Source Inclusion

By default, Modal automatically includes the source file of your Function in the container (`include_source=True` on App). Disable with:

```python
app = modal.App(include_source=False)
```

## When to Use copy=True

Use `copy=True` when subsequent image build steps depend on the files:

```python
image = (
    modal.Image.debian_slim()
    .add_local_dir("./my_package", remote_path="/pkg", copy=True)
    .run_commands("cd /pkg && pip install -e .")
)
```

Without `copy=True`, the files are mounted at startup and cannot be used in build steps.

## Tips

- Use `add_local_python_source` for Python code (cleaner than `add_local_dir`)
- Use `add_local_dir` for non-Python config/data files
- Skip `copy=True` for files that change frequently (faster redeploy)
- Use `copy=True` when build steps need the files
