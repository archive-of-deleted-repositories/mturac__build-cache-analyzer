# /cache-analyze

Analyze the current build cache state in this project.

## Usage

```
/cache-analyze
/cache-analyze --system turbo
/cache-analyze --system nx
/cache-analyze --days 14
```

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--system` | auto-detect | Force a specific build system: `turbo`, `nx`, `webpack`, `vite`, `gradle` |
| `--days` | `30` | Number of days of history to analyze |
| `--top` | `10` | Number of cache-busting files to show |

## What it does

Invokes `skills/cache-analyze` to:
1. Auto-detect build system from project config files
2. Parse cache logs and compute hit/miss rates per task
3. Identify the top files causing cache invalidation
4. Print a summary table with overall hit rate and per-task breakdown

## Example output

```
Overall Cache Hit Rate: 73.2%

Tasks with worst hit rates:
  build:app   45.0%  (9/20)
  test:unit   62.5% (10/16)

Top cache busters:
  package-lock.json   — 18 changes, 15 misses
  src/utils/shared.ts — 12 changes, 12 misses

Run /cache-optimize to fix these issues.
```
