# CLI Commands

Distilled from Modal CLI documentation and guides.

## Core Commands

### modal run

Run an ephemeral app:

```bash
modal run script.py                     # auto-detect entrypoint
modal run script.py::app.func_name      # specific function
modal run script.py --foo 42 --bar baz  # pass arguments
modal run --detach script.py            # keep running after client disconnects
```

### modal serve

Dev server with live-reload for web endpoints:

```bash
modal serve server_script.py
```

Creates temporary URLs, live-updates on file changes. Hit Ctrl-C to stop.

### modal deploy

Persist an app indefinitely:

```bash
modal deploy script.py
modal deploy --name my-app script.py
modal deploy --env production script.py
```

Deployment strategies set in code: `app.deploy(strategy="rolling")` or `"recreate"`.

### modal app

Manage deployed apps:

```bash
modal app list                # list deployed apps
modal app stop my-app         # stop a deployed app
```

## Secret Management

```bash
modal secret list
modal secret create db-secret PGHOST=uri PGPORT=5432 PGUSER=admin PGPASSWORD=hunter2
modal secret delete db-secret
```

## Volume Management

```bash
modal volume list
modal volume create my-vol
modal volume put my-vol local/path remote/path
modal volume get my-vol remote/path local/path
modal volume ls my-vol /path
```

## Environment Management

```bash
modal environment list
modal environment create staging
modal environment delete staging
modal config set-environment staging   # set default
```

## Other Useful Commands

```bash
modal setup               # authenticate (first time)
python -m modal setup     # alternative if modal setup fails
modal token new            # generate a new token
modal profile list         # list authentication profiles
```
