# botir-skills

Multi-skill repository for `skills.sh`-compatible agent skills.

## Skills

- `rust-performance` - measurement-first Rust optimization guidance distilled from The Rust Performance Book

## Install

List skills in this repo:

```bash
npx skills add <your-user>/botir-skills --list
```

Install a specific skill:

```bash
npx skills add <your-user>/botir-skills --skill rust-performance
```

## Layout

- `skills/` - individual skills, one directory per skill
- repo root licenses - carried attribution and license files for derived work where needed

## Attribution

The `rust-performance` skill is derived from The Rust Performance Book by Nicholas Nethercote and contributors:

- Source: `https://github.com/nnethercote/perf-book`
- License: `MIT OR Apache-2.0`

This repo contains a distilled derivative skill, not a verbatim mirror of the upstream book.
