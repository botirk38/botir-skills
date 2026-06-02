# Local Development

Distilled from Modal guide on local development.

## modal serve

The primary local development workflow for web endpoints:

```bash
modal serve server_script.py
```

- Creates an ephemeral app with temporary URLs
- Live-updates when source files change
- No manual restart needed
- Hit Ctrl-C to stop

## Running Functions Locally

Use `.local()` to run a Function in your local Python process (skipping Modal containers):

```python
result = my_function.local(arg)
```

Useful for debugging and quick iteration. Does not have access to GPU, Secrets, or Volumes configured for the remote Function.

## modal.is_local()

Check whether code is running locally or in a Modal container:

```python
if modal.is_local():
    # local-only setup
    secret = modal.Secret.from_dict({"KEY": os.environ["LOCAL_KEY"]})
else:
    secret = modal.Secret.from_dict({})
```

## Programmatic Runs

```python
if __name__ == "__main__":
    with modal.enable_output():
        with app.run():
            result = my_function.remote(42)
```

Always wrap `app.run()` in `if __name__ == "__main__"` to avoid execution in containers.

## Tips

- Use `modal serve` for web endpoint iteration (fastest feedback loop)
- Use `modal run` for batch/script iteration
- Use `.local()` for unit testing individual Functions
- Use `--detach` for long-running ephemeral jobs where you may disconnect
- Enable output with `modal.enable_output()` for programmatic runs
