# Application Error Handling with anyhow

## When to Use anyhow

- Binary/application crates (not libraries)
- Rapid prototyping where error taxonomy isn't needed yet
- Scripts and CLI tools where you just want to print a useful error chain

## Setup

```toml
[dependencies]
anyhow = "1"
```

## Basic Usage

```rust
use anyhow::{bail, ensure, Context, Result};

fn main() -> Result<()> {
    let config = load_config()?;
    run_app(config)
}

fn load_config() -> Result<Config> {
    let text = std::fs::read_to_string("config.toml")
        .context("could not read config.toml")?;
    let config: Config = toml::from_str(&text)
        .context("config.toml has invalid format")?;
    Ok(config)
}
```

## Key APIs

### bail! — Return error immediately

```rust
if users.is_empty() {
    bail!("no users found in database");
}
```

### ensure! — Assert condition or bail

```rust
ensure!(port > 0 && port < 65536, "invalid port: {port}");
```

### context / with_context

```rust
// Static context
file.read_to_end(&mut buf).context("reading input file")?;

// Dynamic context (lazy — only evaluated on error)
process(item).with_context(|| format!("processing item {}", item.id))?;
```

### Downcasting

Recover the original typed error:

```rust
match result {
    Err(e) => {
        if let Some(io_err) = e.downcast_ref::<std::io::Error>() {
            if io_err.kind() == std::io::ErrorKind::NotFound {
                return create_default();
            }
        }
        return Err(e);
    }
    Ok(v) => v,
}
```

### Error Chain

```rust
fn print_error_chain(err: &anyhow::Error) {
    eprintln!("Error: {err}");
    for cause in err.chain().skip(1) {
        eprintln!("  Caused by: {cause}");
    }
}
```

## anyhow vs thiserror

| | anyhow | thiserror |
|--|--------|-----------|
| Use case | Applications | Libraries |
| Error type | Type-erased `anyhow::Error` | Concrete enum |
| Matching | Downcast | Pattern match |
| Context | `.context()` | Manual wrapping |
| Public API | No (opaque) | Yes (typed) |

## Combining Both

A library exposes typed errors with thiserror; the application wraps them with anyhow:

```rust
// In library crate
#[derive(Debug, thiserror::Error)]
pub enum DbError { ... }

// In application
use anyhow::Context;
let user = db::find_user(id)
    .context("looking up user for auth check")?;
```

## main() Returning Result

```rust
fn main() -> anyhow::Result<()> {
    // On error, prints: "Error: {msg}\n\nCaused by:\n  ..."
    Ok(())
}
```

For custom formatting, catch in main and format yourself:

```rust
fn main() {
    if let Err(e) = try_main() {
        eprintln!("fatal: {e:#}");  // '#' = alternate, shows full chain inline
        std::process::exit(1);
    }
}
```
