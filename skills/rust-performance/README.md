# rust-performance

Distilled Rust optimization guidance for agent workflows, based on The Rust Performance Book.

## What This Skill Does

This skill helps an agent diagnose and improve Rust performance with a measurement-first workflow. It focuses on:

- runtime speed
- memory use and allocation churn
- binary size
- compile times
- Rust-specific hotspots such as hashing, iterators, I/O, logging, and synchronization

The skill is deliberately distilled. It does not mirror the book chapter-by-chapter. Instead, it groups the book into task-oriented references that are faster for an agent to use during optimization work.

## Install

From the multi-skill repo:

```bash
npx skills add <your-user>/botir-skills --skill rust-performance
```

To inspect all skills in the repo:

```bash
npx skills add <your-user>/botir-skills --list
```

## Reference Layout

- `SKILL.md` - activation criteria, triage workflow, guardrails
- `references/measurement.md` - benchmarking and profiling
- `references/build-configuration.md` - build and profile tuning
- `references/allocations-layout.md` - allocations, type size, layout, wrappers
- `references/collections-iterators.md` - std collection and iterator costs
- `references/io-debugging.md` - I/O, logging, and debugging overhead
- `references/parallelism.md` - concurrency and synchronization trade-offs
- `references/compile-times.md` - build speed diagnosis and fixes
- `references/general-principles.md` - optimization mindset and rules of thumb

## Source and Attribution

This skill is derived from:

- The Rust Performance Book
- Author: Nicholas Nethercote and contributors
- Source: `https://github.com/nnethercote/perf-book`

The upstream book is dual-licensed under `MIT OR Apache-2.0`. This repository carries those licenses at the repo root. This skill is a distilled derivative, not a verbatim mirror.

## Distillation Policy

- The complete published book corpus was scraped from upstream `src/SUMMARY.md` and every Markdown chapter it references.
- The raw scraped sources were used to produce this skill, then removed from the working repo after distillation.
- Content here is reorganized for agent usability and token efficiency.

## Maintenance

When updating this skill from upstream:

1. Fetch `src/SUMMARY.md`
2. Fetch every chapter linked from it
3. Re-distill into the existing reference bundles
4. Preserve attribution and license files

## Good Fit

Use this skill when a user asks for help with:

- profiling slow Rust code
- reducing allocations
- tuning Cargo profile settings
- diagnosing slow builds
- evaluating iterator, hashing, or I/O overhead
- improving performance without cargo-culting changes
